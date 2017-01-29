---
layout: post
title:  "SF Geospatial Sentiment Analysis"
date:   2015-01-17
theme: cosmo
---

<img src="/images/idw_interpolation_plot.png" />

<p class="intro"><span class="dropcap">T</span>his post his post uses scraped twitter data in combination with R and several geospatial techniques to analyze whether or not people in San Francisco are happier near public parks</p>

# Outline

* intro
* image
* discussion of image
* question: public parks? (why parks, not bars)
* choose represenative
* permutation test
* conclusion
* appendix
*   - data
*   - sentiment
*   - geospatial stuff

* intro  √
* data/scraping techniques (1-2 sentences)  √
* eda (?)
* sentiment analysis
* interpolation
* public parks
* permutation test
* conclusion



## Introduction

In 2006, Twitter was created as a microblogging site. Today it is used by over 500 million people . As a dataset, Twitter has proved invaluable to researchers and has been utilized for a number of tasks, such as predicting financial markets, political affiliation, and analyzing the after-effects of natural disasters. It is also used for sentiment analysis, with results yielding similar resuls to traditional metrics, such as polling. In what follows, we'll combine sentiment analysis and geospatial techniques  in an analysis to determine that public sentiment is different near public parks.

 In this case, sentiment is defined as being either negative or positive, on a scale from 0 to 1. If a tweet has a score near 0, it is negative, near 1, positive. Tweets with sentiment scores near 0.5 are considered neutral.

The above map depicts sentiment distributed spatially about San Francisco, with bright areas corresponding to happier areas, and dark areas corresponding to negative areas. Viewing the data in this way allows us to gain insight we otherwise would have missed from the raw texts of the individual tweets.

I can't expect every reader to be familiar with San Francisco, so below is an uncolored map for reference.

[INSERT REGULAR MAP OF SAN FRANCISCO]

We can observe that the greener (positive) colored areas of the sentiment map appear to be near the greener areas of the regular map. Is this just a cool coincidence, or did I pick that color on purpose? (Spoiler: I just chose the colors because they contrast). As a matter of fact, this result is to be expected, as least for surface-level expectations: Green-colored areas, such a public parks, probably are happier than other areas. It's one thing to infer this from an image, it's another to prove so mathematically. Concluding this from a single image is one thing, showing it mathematically is another.
In what follows, we'll see with statistically significant resuls that people are happier 

This article provides 



## Data

As stated, the data for the analysis comes from Twitter. It comes in two forms. First, a dataset was obtained [http://www.followthehashtag.com/datasets/free-twitter-dataset-usa-200000-free-usa-tweets/] with 200,000 geo-tagged tweets from the years 2014 and 2015 in the United States. This dataset provides utility in that it prevents our data from being too confounded with recent events.

Second, approximately 1 million tweets were scraped via the Twitter API through R into a mongodb database. These scraped tweets were scraped over the period of a month and limited to the greater San Francisco area. The two datasets were then merged. The final dataset was restricted to only those tweets that were geotagged within San Francisco and free of *"noise"* hashtags, such as *#werehiring* or *#weatherupdate*, as these provide no information and distort the average sentiment in the data. 

* Initial view of data overlayed on shapefile

* Sentiment Analysis
  - word-embeddings
  - should include eda?
      - histograms of scores
      - distribution plots of sentiment
      