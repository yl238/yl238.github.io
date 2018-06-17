---
layout: default
title: "Stack Overflow question recommender"
excerpt: "First steps in building a question recommender using SO data"
date: 2018-06-17 11:00:00
---
# Stack Overflow Question Recommender

<img src="/figures/stackoverflow.png" width="10%" />
As a computer scientist, when I'm stuck on some technical problem Stack Overflow is the first place I look for answers. Just a minute ago I searched for "how to save Github credentials on the commandline", and probably not for the first time either.  How to remember previously searched for answers is a serious problem, but let's leave that for another day.

I don't always get the question right the first time. Sometimes I type in a vague question and SO leads me to a page that's not quite what I want. But if I look at the right hand side there is a section called 'Related' where SO lists of other questions that might be related to the one I just asked. Sometimes these are useful, sometimes not. Occasionally I browse through them just for fun. But how are they generated? 

Natually searching for this question lead me to a page on Meta Stack Overflow (where else?). It's a fairly old question with equally old answers that involve ElasticSearch and tf-idf vectors. With the whole of StackExchange data now available to download, can I build a recommender system that when given a SO question can identify similar SO questions? 

## Data Extraction
First I need to get hold of the SO data. [Google BigQuery](https://cloud.google.com/bigquery/public-data/stackoverflow) will allow you to access the dataset using SQL queries, but since I'm not looking for answers to specific questions, it seemed better to download the raw data. Fortunately all Stack Exchange content are dumped on a quarterly basis as `.7z` files and can be downloaded through the [Internet Archive](https://archive.org/download/stackexchange). Some of the files are massive: the SO posts file is over 12Gb, which when uncompressed gives a 61Gb XML file. Downloading took a very long time (4 hours for a 12Gb file), which I suspect is due to the fact that there are no mirrors in Europe.

I also needed to download the [readme.txt](https://ia800107.us.archive.org/27/items/stackexchange/readme.txt) file, which describes the dataset schema so I can actually interpret the XML file. A typical line in the `Posts.xml` file looks like this in plain text:
```
<row Id="13" PostTypeId="1" CreationDate="2008-08-01T00:42:38.903" Score="529" ViewCount="153785" Body="&lt;p&gt;Is there any standard way for a Web Server to be able to determine a user's timezone within a web page? &lt;/p&gt;&#xA;&#xA;&lt;p&gt;Perhaps from an HTTP header or part of the user-agent string?&lt;/p&gt;&#xA;" OwnerUserId="9" LastEditorUserId="5321363" LastEditorDisplayName="Rich B" LastEditDate="2018-05-30T15:55:48.913" LastActivityDate="2018-05-30T15:56:46.080" Title="Determine a User's Timezone" Tags="&lt;javascript&gt;&lt;html&gt;&lt;browser&gt;&lt;timezone&gt;&lt;timezoneoffset&gt;" AnswerCount="25" CommentCount="6" FavoriteCount="135" />
```
The schema tells us that `PostTypeId` is 1 for 'question' and 2 for 'answer'. In this first iteration let's ignore the answers and simply look at the questions. Useful fields are `Body`, `Title` and `Tags`. Fortunately the Python module [`xml.etree.elementTree`](https://docs.python.org/3/library/xml.etree.elementtree.html) implements an efficient API for parsing XML so we can extract these fields for each question. We'll also do some data cleaning (e.g. removing brackets around tag names) and save it as a CSV file.


