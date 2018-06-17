---
layout: default
title: "Stack Overflow question recommender"
excerpt: "First steps in building a question recommender using SO data"
date: 2018-06-17 11:00:00
---
# Stack Overflow Question Recommender

<img src="/figures/stackoverflow.png" width="10%" />
As a data scientist, when I'm stuck on some technical problem Stack Overflow is the first place I look for answers. Just a minute ago I searched for "how to save Github credentials on the commandline", and probably not for the first time either.  

I don't always get the question right the first time. Sometimes I type in a vague question and SO leads me to a page that's not quite what I want. But if I look at the right hand side of the page there is a section called 'Related', which is a list of other questions that SO thinks are related to the one I asked. Sometimes they are useful, and other times they seem quite arbitrary. But how are they generated? 

Searching for this question led me to a page on Meta Stack Overflow, which mentioned that SO uses ElasticSearch, and the 'Related' questions are obtained using the `find_most_like` query. Since I have not delved into the depth of the ElasticSearch[ codebase](https://github.com/elastic/elasticsearch), I can only presume that they are using a similar item finder of some sort.  

For our work we are unlikely to employ a full-scale real-time search engine like ElasticSearch, but we would still like to find similar items from text. For example, we might want to find all the tasks in previous projects that are similar (in some predefined way) to the current one, so we can use their attributes to guide present planning. 

So let's build an offline model that can find similar questions asked in Stack Overflow. Despite it being an unsupervised problem, there is even a way of assessing the quality of the model. SO users manually mark and close questions considered to be 'too similar' to a previous question. If our algorithm works well, it should be able to identify these repeated questions. 

Finding similar items is a well-known problem in information retrieval and there are a number of algorithms out there. The most often used one is Locality Sensitive Hashing (LSH) but neural networks based models such as autoencoders have been developed. In this project we shall try to explore them all. 

Jupyter notebooks for the project can be found at [this repository](https://github.com/yl238/stackoverflow).

## Data Extractio
[Google BigQuery](https://cloud.google.com/bigquery/public-data/stackoverflow) will allow you to access the SO dataset using SQL queries, but since I'm not looking for answers to specific questions, it seemed better to just download the raw data. Fortunately all Stack Exchange content are dumped on a quarterly basis as `.7z` files and can be downloaded through the [Internet Archive](https://archive.org/download/stackexchange). Some of the files are massive: the SO posts file is over 12Gb, which when uncompressed gives a 61Gb XML file. Downloading took a very long time (4 hours for a 12Gb file), which I suspect is due to the fact that there are no mirrors in Europe.

I also needed to download the [readme.txt](https://ia800107.us.archive.org/27/items/stackexchange/readme.txt) file, which describes the dataset schema so I can actually interpret the XML file. A typical line in the `Posts.xml` file looks like this in plain text:
```
<row Id="13" PostTypeId="1" CreationDate="2008-08-01T00:42:38.903" Score="529" ViewCount="153785" Body="&lt;p&gt;Is there any standard way for a Web Server to be able to determine a user's timezone within a web page? &lt;/p&gt;&#xA;&#xA;&lt;p&gt;Perhaps from an HTTP header or part of the user-agent string?&lt;/p&gt;&#xA;" OwnerUserId="9" LastEditorUserId="5321363" LastEditorDisplayName="Rich B" LastEditDate="2018-05-30T15:55:48.913" LastActivityDate="2018-05-30T15:56:46.080" Title="Determine a User's Timezone" Tags="&lt;javascript&gt;&lt;html&gt;&lt;browser&gt;&lt;timezone&gt;&lt;timezoneoffset&gt;" AnswerCount="25" CommentCount="6" FavoriteCount="135" />
```
The schema tells us that `PostTypeId` is 1 for 'question' and 2 for 'answer'. In this first iteration let's ignore the answers and simply look at the questions. Useful fields are `Body`, `Title` and `Tags`. The Python module [`xml.etree.elementTree`](https://docs.python.org/3/library/xml.etree.elementtree.html) implements an efficient API for parsing XML so we can extract these fields for each question. We'll also do some data cleaning (e.g. removing brackets around tag names) and save it as a CSV file.

