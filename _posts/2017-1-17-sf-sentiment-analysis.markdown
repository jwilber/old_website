---
layout: post
title:  "SF Geospatial Sentiment Analysis"
date:   2015-01-17
theme: cosmo
---

<span class="dropcap">T</span>his post his post uses scraped twitter data in combination with R and several geospatial techniques to analyze whether or not people in San Francisco are happier near public parks

## Introduction

In 2006, Twitter was created as a microblogging site. Today it is used by over 500 million people . As a dataset, Twitter has proved invaluable to researchers and has been utilized for a number of tasks, such as predicting financial markets, political affiliation, and analyzing the after-effects of natural disasters.  In what follows, we'll combine sentiment analysis and geospatial techniques to determine that public sentiment is different near public parks.

<img src="/images/idw_interpolation_plot.png" width="650" height="500" />

The above map depicts sentiment distributed spatially about San Francisco, with brighter areas corresponding to happier areas, and darker areas corresponding to negative areas. Viewing the data in this way allows us to gain insight we otherwise would have missed from the raw texts of the individual tweets. Namely, we can see which locations are happier, which are madder.

I can't expect every reader to be familiar with San Francisco, so below I show a colored  map of San Francisco for all you non-locals.

<img src="/images/plainsf.png" width="650" height="500" />


We can observe that the greener (positive) colored areas of the sentiment map appear to be near the greener areas of the regular map. Is this just a cool coincidence, or did I pick that color on purpose? (Spoiler: I chose the colors because they contrast). As a matter of fact, this result is to be expected, as least from a high-level standpoint: Green-colored areas, such a public parks, probably are happier than other areas. 

That said, concluding this from a single image is one thing, showing it mathematically is another.
In what follows, we'll see with statistically significant resuls that people**** (who tweet) are indeed happier near public parks.

Let's get to it.

To test our hypothesis, we'll measure the average sentiment of people who tweet near public parks and compare it to the average sentiment of those tweets not near public parks.

San Francisco has plenty of public parks, and we don't want to measure multiple parks in the same general vicinity. Thus, we'll choose parks that are distant from each other but together compose most of San Francisco. This way our sample is as representative as possible.

|             Park            | Longitude | Latitude |
|:---------------------------:|-----------|----------|
| Dolores Park                | -122.4271 | 37.7598  |
| Golden Gate Park            | -122.4862 | 37.7694  |
| Lafayette Park              | -122.4276 | 37.7916  |
| Park Merced                 | -122.4810 | 37.7183  |
| John Mclaren Park           | -122.4194 | 37.7193  |
| Victoria Manalo Draves Park | -122.4061 | 37.7771  |
| Mountain Lake Park          | -122.4697 | 37.7873  |

Plotted graphically, these public parks appear to meet our critera.


<img src="/images/pubparks.png" width="400" height="300" />

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
      
  In this case, sentiment is defined as being either negative or positive, on a scale from 0 to 1. If a tweet has a score near 0, it is negative, near 1, positive. Tweets with sentiment scores near 0.5 are considered neutral.



** Addressing interpolation**
Because our data didn't fully exhaust the geospatial space, we used interpolation techniques to predict the sentiment for those locations we didn't have. We used a method called **inverse-distance weighting**.

Inverse distance weighting works as follows (insert LaTeX).

To use this, we first split our data up into as follows. 
[show plots]
Then we applied the formula to our those locations without tweets.
Combining these techniques with some standard shapefile, crs, etc. clean up resulted in the map you saw above.

** Addressing sentiment**
discuss how tweet sentiment was created

**Address Permutation Tests**

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

* eda (?)
* sentiment analysis
* interpolation
* public parks
* permutation test
* conclusion

