---
layout: post
title:  "SF Geospatial Sentiment Analysis"
date:   2015-01-17
theme: cosmo
---

# [ this post is in progress]

<p class="intro"><span class="dropcap">T</span>his post his post uses scraped twitter data in combination with R and several geospatial techniques to analyze whether or not people in San Francisco are happier near public parks</p>

# Outline

* intro  √
* data/scraping techniques (1-2 sentences)  √
* eda (?)
* sentiment analysis
* interpolation
* public parks
* permutation test
* conclusion



## Introduction

In 2006, Twitter was created as a microblogging site. Today it is used by over 500 million people . As a dataset, Twitter has proved invaluable to researchers and has been utilized for a number of tasks, such as predicting financial markets, political affiliation, and analyzing the after-effects of natural disasters. It is also used for sentiment analysis, with results yielding similar resuls to traditional metrics, such as polling. In what follows, we'll combine standard sentiment analysis techniques with geography in an analysis to determine that public sentiment is different near public parks.

## Data

As stated, the data for the analysis comes from Twitter. It comes in two forms. First, a dataset was obtained [http://www.followthehashtag.com/datasets/free-twitter-dataset-usa-200000-free-usa-tweets/] with 200,000 geo-tagged tweets from the years 2014 and 2015 in the United States. This dataset provides utility in that it prevents our data from being too confounded with recent events.

Second, approximately 1 million tweets were scraped via the Twitter API through R into a mongodb database. These scraped tweets were scraped over the period of a month and limited to the greater San Francisco area. The two datasets were then merged. The final dataset was restricted to only those tweets that were geotagged within San Francisco and free of *"noise"* hashtags, such as *#werehiring* or *#weatherupdate*, as these provide no information and distort the average sentiment in the data. 

* Initial view of data overlayed on shapefile

* Sentiment Analysis
  - word-embeddings
  - should include eda?
      - histograms of scores
      - distribution plots of sentiment
      
