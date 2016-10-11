---
layout: post
title:  "Permutation Test: Explanation and Implementation in R"
date:   2016-09-12
theme: cosmo
---

<p class="intro"><span class="dropcap">T</span>his post explains what a permutation test is, how it is performed, and why you might want to utilize it. It also shows you how to implement your own permutation test from scratch in R, as well as how to use pre-defined permutation tests in existing packages</p>


# Introduction

The permutation test, also known as the randomization test, is a nonparametric method of statistical inference that tests a specific null hypothesis that the treatment levels we are comparing are completely equivalent and serve only as labels; i.e. that the responses we observed for our units would be the same no matter which treatments had been applied.

Put more simply, when considering a permutation/randomization based analysis, our null hypothesis dictates that our response values are equally likely of ending up in any treatment group. Then, under that hypothesis, permuting our data will yield no effect on the outcome. 

Our alternative hypothesis, then, dictates that the data do come from different distributions; that there does exist some difference between treatments for one or more subjects.

To construct a permutation test, we select a test statistic for
the data and then get the distribution of that test statistic under the permutation null hypothesis. This is akin to permuting our data and calculcating the test statistic for each permutation. 

The permutation test p-value is the probability (under the null
randomization distribution) of getting a test statistic as extreme or
more extreme than the one we observed.

An expression for a two-tailed permutation test consisting of B permutations is shown below:

<img src="/images/perm.png" />


Unlike the more traditional statistical tests (e.g. t-tests or F-tests), the permutation test relies only on the performed randomization and, as such, we can ignore many annoying assumptions often required for linear models (such as normality, heteroscedastic residuals, etc.). That said, our data does need to meet the assumption of exchangeability (i.e. can we permute the data without affecting the distribution?) You can visit the following link for more information on exchangeability: https://en.wikipedia.org/wiki/Exchangeable_random_variables

Carrying out a permutation test consists of 3 steps:

  - 1 Come up with test statistic
  - 2 Permute Data and Obtain Sampling Distribution
  - 3 Conduct Permutation Test and Obtain P-Value
  
In what follows, I'll go into each step in detail and, along the way, reveal how one could implement each step in R.

# R Implementation

### Our Data and Brief EDA

For ease of reproducibility, we'll create our own dataset. 

We'll create a data set with two columns:
  - SURVIVAL: Survival in months after some operation was performed.
  - Treatment: A boolean; A value of TRUE indicates that the unit received post-operation treatment. FALSE indicates that they did not (Control group).
  
For example, the response values in our dataset may refer to survival times after some breast cancer operation, where our "TREATMENT" refers to whether or not the individuals received some post-operation treatment or not.

The data is defined and showcased below:

{% highlight R %}
ourData <- data.frame(SURVIVAL = c(2,6,8,10,10,12,12,14,14,16,16,16,18,20,26,30,34,38,40,48,2,4,4,6,8,8,10,10,12,14,18,18,20,22,32,36,46,46,48,58,58,66,72,82),
TREATMENT = c(rep(FALSE,22),rep(TRUE,22)))
ourData
{% endhighlight %}

<img src="/images/dataset1.png" />

We'll use the following hypothesis scheme:

  - Null Hypothesis: The means for the Control (FALSE) and Treatment (TRUE) groups are the same; i.e. the treatment has no effect on survival outcome.
  - Alternative Hypothesis: The means for the Control (FALSE) and Treatment (TRUE) groups are different; i.e. the treatment does have some effect on survival times.

When performing any analysis, it's always a good idea to carry out some exploratory data analysis (EDA). For this example, we'll be brief and create a plot to visualize what we expect the outcome of our test will be:

<img src="/images/box.png" width="450px" height="450px" />
`

The difference in centrality and spread certainly give credence to the hypothesis that the two groups come from different distributions.  Of course, we don't know for sure until we perform our test. Let's dive into our analysis and find out for sure.

### 1. Come up with test statistic
The first step of a permutation test is to come up with a relevant test-statistic. Recall that a test statistic is some numerical summary of our data that can be used during a statistical test to distinguish the null hypothesis from the alternative hypothesis. For a permutation test we can use essentially anything for our test statistic. The goal of our analysis is to detect any difference between the treatment and control groups. Therefore, we'll employ the difference of mean values between the two groups as our test statistic. 

{% highlight ruby %}
diffMeans <- function(data, hasTrt){
  # computes our test statistics: the difference of means
  # hasTrt: boolean vector, TRUE if has treatment
  test_stat <- mean(data[hasTrt]) - mean(data[!hasTrt])
  return(test_stat)
}
{% endhighlight %}

``` R
currentStat <- diffMeans(ourData$SURVIVAL, ourData$TREATMENT)
cat("Initial Test Statistic: ", currentStat)
```

We calculate the test statistic for our initial data set, which we'll utilize later in our analysis.

{% highlight R %}
currentStat <- diffMeans(ourData$SURVIVAL, ourData$TREATMENT)
cat("Initial Test Statistic: ", currentStat)
{% endhighlight %}

<img src="/images/teststat1.png" />

### 2. Permute Data and Obtain Sampling Distribution

Once we've decided upon our test statistic, we permute our data. Permuting the data is just a fancy way of saying to rearrange or shuffle the data in all possible manners. As our null hypothesis states that shuffling the data should have no effect, this process creates the backbone of our permutation test: our randomization distribution. Recall the number of permutations for n elements is factorial(n), a function grows faster than even exponential(n). See below plot.

<img src="/images/growth.png" width="450px" height="450px" />



Thus, even for moderately sized n, computing all permutations quickly becomes unfeasible. For example, factorial(15) is an astronomical 1,307,674,368,000. In comparison, exp(15) is a tiny 3,269,017.

To resolve this issue, instead of creating a distribution of all possible permutations, we'll simply build our permutation distribution by drawing some number, say k, of samples from the total permutation set. 

We then calculate our test statistic for each generated sample. In this way, we build a distribution for our test statistic (our randomization distribution under the null).

**Note** - this method will tend to be conservative for small sample sizes. For example, the smallest obtainable p-value (greater than 0) for 20 samples is 0.05. Hence, a sufficient sample size is necessary to attain an accurate p-value. Typically we'll use no less than 5,000 examples. The following implementation uses 10,000 samples.



{% highlight R %}
simPermDsn <- function(data, hasTrt, testStat, k=10000){
  # Simulates the permutation distribution for our data
  # hasTrt: boolean indicating whether group is treatment or control
  # testStat: function defining the test statistic to be computed
  # k: # of samples to build dsn.      
  sim_data   <- replicate(k, sample(data))
  test_stats <- apply(sim_data, 2,
                      function(x) {testStat(x, hasTrt)})
  return( test_stats)
}
{% endhighlight %}


The above code simple creates k permutations of our data.

For our test statistic, we observe the following distribution of values:

<img src="/images/hist1.png" width="450px" height="450px" />


I should also briefly clarify that while the two are often confused,the above sampling scheme is not the same as bootstrapping. Bootstrapping samples with replacement.


### 3. Conduct Permutation Test and Obtain P-Value


Once we've created a sampling distribution for our test statistic, we can proceed to carry out the permutation test with ease. Acquiring our p-value is simple: we simply count the number of times we've obtained test statistics as OR MORE EXTREME than our initial test statistic, then divide that sum by the total number of samples.

Recall that our hypothesis was of the "two-way" form. Thus, in my code, I look for the sum of the *absolute value* of deviations greater than our initial test statistic, divided by the total number of permuted samples, k. For a one-way test, just sum those as-or-more-extreme observations in the relevant direction, then divide by k.

To perform our test, we'll encapsulate everything into one function as follows:

{% highlight R %}
permutationTest <- function(data, hasTrt, testStat, k=1000){
  # Performs permutation test for our data, given some pre-selected
  # test statistic, testStat
  currentStat    <- testStat(data, hasTrt)
  simulatedStats <- simPermDsn(data, hasTrt,testStat, k)
  
  # 2-way P-value
  pValue         <- sum(abs(simulatedStats) >= currentStat)  / k
  
  return(pValue)
}
{% endhighlight %}


The first two lines of code in the above function define our initial test statistic and create our sampling distribution by calling previously defined functions. The third line calculates our p-value via the process stated above.

Congratulations! Now that we have defined our function, we've successfully implemented a permutation test!

So what are our results? Let's run our function on our data to find out:

{% highlight R %}
set.seed(619)
p.val <- permutationTest(ourData$SURVIVAL, ourData$TREATMENT, testStat = diffMeans)
cat("p-value: ", p.val)
{% endhighlight %}

<img src="/images/pvalues1.png" />

So, our p-value is 0.03. 

We can visualize this by observing our original test statistic's location in a sampled test statistic's distribution (symbolized by the vertical blue lines below). Recall that we calculated this earlier, obtaining a value of ~ 13:

<img src="/images/hist2.png" width="450px" height="450px" />

This test statistic looks pretty deep in the tails of our distribution. In fact, with a significance level of 0.05, our p-value is significant. Thus we reject our null hypothesis and conclude that there is a difference provided by the treatment.

For sanity, we can check how our test agrees with the output of a parametric alternative; in this case, a two-sample t-test.
 
### Comparison with T-Test
 We'll implement the test ourselves:


{% highlight R %}
tTest <- function(data, hasTrtment){
  m         <- sum(hasTrtment) # Treatment group size
  n         <- sum(!hasTrtment) # Control group size
  
  # difference of means (assuming unequal variances):
  mean_diff <- diffMeans(data=data, hasTrt=hasTrtment)
  # denominator:
  trt_var   <- var(data[hasTrtment])/m
  ctrl_var  <- var(data[!hasTrtment])/n
  sd_diff   <- sqrt(trt_var + ctrl_var)
  # t-statistic: 
  t_stat    <- mean_diff / sd_diff
  # p-value:
  df        <- min(n-1,m-1)
  pval      <- 1-pt(t_stat, df)
  
  return( pval )
}

tTest(ourData$SURVIVAL, ourData$TREATMENT)
{% endhighlight %}

As expected, both tests yield significant results, thus leading to the same conclusion: we reject the null hypothesis and conclude that the treatment does have a significant effect on Survival time post-operation.

Note that, while it's nice to be able to implement your own permutation test, it's not always necessary. In particular, I know of two R packages: **coin** and **perm**, each of which allows us to conduct a permutation test wit a single line of code: 

{% highlight R %}
# library(coin)
pvalc <- pvalue(independence_test(SURVIVAL~TREATMENT, data=ourData))

# library(perm)
pvalp <- permTS(SURVIVAL ~ TREATMENT, data=ourData, alternative = "two.sided", nmc=10000)$p.value

# compare p-values
cat(" p-value (from coin package) : ", round(pvalc,2), "\n",
    "p-value (from perm package) : ", round(pvalp,2), "\n",
    "p-value (our implementation): ", p.val)

{% endhighlight %}

<img src="/images/pvalues3.png" />

Our implemented permutation test performed just as well as the packages' respective permutation tests. Thus, feel free to use whatever you prefer for your own analysis!

### Conclusion

I hope that the above example revealed the utility of a permutation test.
In my next post, I'll give an applied example of the permutation test where we test whether or not John Goodman has a significant effect on movies being "hits" or not. 
Thanks for reading and I hope you learned something!



*Note,  this post was inspired by an assignment from Stat 158: Experimental Design, a course I took at UC Berkeley under Elizabeth Purdom and Christine Ho.*
