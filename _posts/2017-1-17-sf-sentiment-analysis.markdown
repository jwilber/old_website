---
layout: post
title:  "SF Geospatial Sentiment Analysis"
date:   2017-01-17
theme: cosmo
---

<p class="intro"><span class="dropcap">T</span>his post his post uses scraped twitter data in combination with R and several geospatial techniques to analyze whether or not people in San Francisco are happier near public parks</p>

# Outline

* intro  √
* data/scraping techniques (1-2 sentences)
* eda (?)
* sentiment analysis
* interpolation
* public parks
* permutation test
* conclusion



## Introduction

In 2006, Twitter was created as a microblogging site. Today it is used by over 500 million people . As a dataset, Twitter has proved invaluable to researchers and has been utilized for a number of tasks, such as predicting financial markets, political affiliation, and analyzing the after-effects of natural disasters. It is also used for sentiment analysis, with results yielding similar resuls to traditional metrics, such as polling. In what follows, we'll combine standard sentiment analysis techniques with geography in an analysis to determine that public sentiment is different near public parks.
