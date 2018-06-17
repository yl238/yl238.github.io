---
layout: default
title: "Stack Overflow question recommender"
excerpt: "First steps in building a question recommender using SO data"
date: 2018-06-17 11:00:00
---
<img src="/figures/stackoverflow.png" width="10%" />
As a computer scientist, when I'm stuck on some technical problem Stack Overflow is the first place I look for answers. Just a minute ago I searched for "how to save Github credentials on the commandline", and probably not for the first time either.  How to remember previously searched for answers is a serious problem, but let's leave that for another day.

I don't always get the question right the first time. Sometimes I type in a vague question and SO leads me to a page that's not quite what I want. But if I look at the right hand side there is a section called 'Related' where SO lists of other questions that might be related to the one I just asked. Sometimes these are useful, sometimes not. Occasionally I browse through them just for fun. But how are they generated? 

Natually searching for this question lead me to a page on Meta Stack Overflow (where else?). It's a fairly old question with equally old answers that involve ElasticSearch and tf-idf vectors. With the whole of StackExchange data now available to download, can I build a recommender system that can identify similar SO questions? 

