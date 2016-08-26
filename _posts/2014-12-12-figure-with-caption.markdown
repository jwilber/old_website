---
layout: post
title:  "State of the Union Address Text Analysis"
date:   2014-12-15
theme: cosmo
---

- to do list
  - clean up grammer, diction, etc.
  - create proper name for each (4) cluster
  - Clean up plots
  - 

<p class="intro"><span class="dropcap">T</span>his post his post uses R and several text mining techniques to analyze the presidential State of the Union Address Speeches</p>


# Introduction

The State of the Union address is an (typipcally) annual speech given by the President of the United States to a joint session of Congress that outlines the President's legislative agenda, national priorities, and the current state of the country.

With the upcoming elections pending near, I thought it'd be interesting to analyze some political content. In the following post, we'll use text mining techniques to analyze the State of the Union addresses from the years 1790 to 2012. Hopefully the following analysis will showcase some of the power of text mining techniques in R.

The data used for this analysis is a very long text file containing all of the State of the Union Adress speeches from the years 1790 to 2012. It is freely available to view/download at [http://www.gutenberg.org/ebooks/5050]. 

Using the data as-is obviously won't work. However, extensive use of data cleaning (particularly regular expressions) yields a workable dataset. My code used to clean the dataset (as well as for the rest of the analysis) is available on my github at github.com/jwilber.



EXPLAIN TFIDF MATRIX STUFF

- above, possible discuss data more (num lines, etc.)


In analyzing the State of the Union speeches, we can take two primary strategies: 
  -analyze the speeches individually on a president-by-president basis
  -analyze the speeches as an aggregate and search for general trends.

In the individual case we can zoom in on each president and analyze them piecewise. For example, we can create a word cloud for each president. Below I'll juxtapose the most-common words used by Barack Obama with the most-common words used by Geroge W. Bush:

<img src="/images/bushobama_clouds.png" />


COMMENT ON FINDINGS



Whiles individual analyses are undoubtedly interesting, zooming in on each  president is large both in time and space. 
Thus, for this analysis, I chose to analyze the aggregate and view speeches together.

In order to discuss the relationship between the speeches, a distance metric must be selected. For example, perhaps the simplest distance metric for comparing speeches is the "hamming distance", which EXPLAIN HAMMING DISTANCE
In this analysis, I utilize a much more complicated measure of distance, the INSERT DISTANCE METRIC AND EXPLAIN.
This distance metric _ and allows us to __
DISCUSS TEXT DISTANCE METHOD
To perform the following analysis, a distance metric must be defined. For the following post, I se the


An obvious point of interest is to view which presidents share the most in common with each other. We'll use a clustering algorithm called "Hierarchical Clustering" to create a dendrogram for easy visualization of this relationship.
In agglomerative hierarchical clustering each observation is initialized as its own cluster, and pairs of clusters are merged going up the hierarchy until all observations fall into one class. Note that the earlier in the dendrogram that observations are clustered, the more similar they are to each other.

<img src="/images/sotus_hclust.png" />

Our dendrogram reveals, then, that Andrew Jackson and Martin Van Buren are our most similar presidents according to SOTUS content. [WHAT DO THEY HAVE IN COMMON?], while John F. Kennedy appears to be the most separate from any president.  [SPECULATE AS TO WHY]. Many more observations can be made, but perhaps the most SALIENT is that the presidents appear to be clustered in a chronological manner. For example, on the far left we can see George W. Bush alongside Bill Clinton and Obama. Notice that within this cluster, Obama and Clinton are clustered closer; perhaps a clustering exists at a party level as well as a chronological one?

To investigate this, we'll use two techniques in tandem: T-SNE and K-Means Clustering.

T-SNE is a relatively new dimensionality reduction technique developed by Laurens van der Maaten. In this context, t-SNE should reveal any clusters in our data. Below is a plot of our t-sne embedding.

<img src="/images/uncolored_tsne.png" />


Our t-SNE output appears to have some clusters in the data. In fact, the clusters are even more profound than they were for our dendrogram. If Our earlier presuppositions appear to be correct; the speeches are clearly divided into distinct blocks, based on chronological events. 

On top of our t-SNE plot, we'll use a basic clustering algorithm called k-means.K means is a clustering algorithm that iteratively assigns elements to clusters based on a given distance metric such that each cluster is as similar as possible (i.e. the variance within each cluster is as small as possible).  We'll use k-means to identify and explicitly highlight our clusters. 

<img src="/images/colored_tsne.png" />

In this specific case, k-means performed most optimally with four clusters.


Based on the cluster assignments, I've determined the following class assignments:

Cluster 1:
Presidents:

Cluster 2:
Presidents:

Cluster 3:
Presidents:

Cluster 4:
Presidents:


I should also briefly mention that these clusters reappeared using other dimensionality reduction techniques as well. For example, clustering on Multi-Dimensional Scaling (MDS), a technique used to preserve distance, is shown below. Although the clusters appear differently than they did in our previous t-SNE embedding, the class assignments are exactly the same.

<img src="/images/mds.png" />

So clearly our data is divided into four main blocks. But why?

Recall that the primary aim of the SOTUS is to discuss the present state of the nation. Thus it makes sense to observe that presidents closer together in time have more similar speeches, because they're discussing similar (or related) events! Intuitively, we'd expect words like "iraq" and " terror" to be common in speeches post-2001, while absent from speeches during the 1800s. This also explains why we don't observe a large disparity among political parties; the focal point of the speeches are the nation's issues, it's the solutions to those issues that (usually) differ by party.

Below we'll view the relevant issues unique for each time frame. That is, we'll view the distribution of those words unique to each time frame, as these should reveal the salient issues of each time frame.

- discuss early hist/cloud
<img src="/images/early_hist.png" />

- archaic vocabulary (prussia, barbarian, hayti)
- suffrage
- confederacy, cherokee, barbarian



<img src="/images/postviet_hist.png" />
- war and pessismism seems t odominate
- war/middle east: al-quada, bosnia, hussein, iraq, saddam, taliban, terror
- tech growth: high-tech, entrepreneur, internet, math (though math is probably related with teen, )
- job emphasis: americorps, teach,
- brady-bill




- discuss ww2-vietname hist/cloud
<img src="/images/preww2_hist.png" />
WW2 dominates: nazi, kremlin, tank
- bilateral, 
- polaris

<img src="/images/ww2viet_hist.png" />
- battleship, porto-rico, filipino, isthmus, 
- monroe
- hague
- exposition, cable




The above results aren't very surprising; after all, after discovering the blocks for our data, it's no surprise that they have separate vocabulary., the eras differ in the vocabulary utilized, as obvoiusly each speech is a function of the contemporary events.. 
But do the groups differ in other manners as well?
We'll pursue this end by investigating the implicit, "meta" features of the text as opposed to the explicit vocabulary used.

To make any apparent trends obvious, I fit a regression line to each of the plots.

Counting from left-to-right, top-to-bottom, we can see that the first three plots yield little information about general trends in the data. However the fourth plot, Average Sentence Length per Year, clearly reveals a downward slope. 

<img src="/images/avgsent.png" />

Note the stark difference in the slope of plot four; the decreasing line indicates that average sentence lengths decrease over time. Is this downward slope just a coincidence, or is this trend legitimately occuring? We can determine that this occurence is statistically significant: we'll split our data up into the four groups, calculate the average sentence length/year for each president in said group, and test if the mean length for each group is statistically significant using ANOVA. 

Our results indicate that it is indeed the case that sentences are getting less complicated over time. So why is this the case? Perhaps this is an effect of our decreasing attention spans. With the increasing importance of technology over the years, informatin has been compressed and, as a result, many attention spans have been lowered. Or perhaps networks put pressure on sentence length; after all, although a SOTUS could technically go on for hours, networks airing the speech may put pressure so that they can keep to their schedule after certain time. Whatever the reason, the phenomena is indeed occuring.

However, the above hypotheses are weak as they don't adequately explain why tsuch trends exist for our earliest groups.doesn't explain why this trend exists for our earliest groups. Thus, it's hard to say

Unfortunately, I am by no means a historian, so my comments as to why we observed what we did are pretty unsubstantiated. Still, I think it's very interesting what we can learn with some simple application of statistical methods.

Thus, our simple data analysis has yielded several interesting facts:
- SOTUS are blocked into 4 primary timeframes
- political party does not effect time frame

- popular terms for each era
- JFK is a bit of an outlier, at least in regards to his SOTUS rhetoric
- sentence complexity has decreased over time

