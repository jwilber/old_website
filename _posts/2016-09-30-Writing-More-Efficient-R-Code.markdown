---
layout: post
title:  "Writing Efficient R Code"
date:   2016-09-30
---
[in progress]

<p class="intro"><span class="dropcap">T</span></p>he goal of this post is to (hopefully) introduce you to some methods that will aid you in writing more efficient R code. While I am by no means an R expert, I've picked up some techniques in my few years of using R that will hopefully prove beneficial to you!


### Why?
R is a programming language primarily used for data science and statistical analysis. 
While R is complimented by many when it comes to data analysis, it's definitely not praised for its efficiency (specifically, it's speed or memory management). That's not to say that R isn't very efficient; for the vast majority of cases, it will get the job done and do it well. However, it is true that R is not as efficient as it can be. This stems from two primary reasons

1. R is very, very dynamic. Although this dynamism gives us more flexibility, when coupled with lexical scoping it results in slower code.
2. The current most popular implementation of R (gnu-R, this is what you have if you don't know which implementatin of R you have) is far from peak-optimization with regards to the current implementation.

That said, the point of this post isn't discussing R's current implementation, it's to help you write better R code. If you would like to learn more about R, I suggest reading either [BOOK OR BOOK LINK]

Here is an outline of what I will cover:

* Time your code
  1. system.time()
  2. microbenchmark
  3. proc.time()

* Memory
  1. Memory Preallocation
  2. Memoisation and Caching Variables
  3. Compiling Code
  4. Analysing Memory in Use

Let's get started.

***



# Timing Your Code
In order to write more efficient code, we need some way by which to facilitate comparison. An obvious choice in measuring speed is timing our code. Below I list three methods we'll use, as well as when to use them.

* `system.time()`: Analyze individual function calls
* `microbenchmark()`: Compare the speed of multiple function calls
* `proc.time()`: Analyze the speed of chunkcs of our code

#### `system.time()`

The `system.time()` function handles a single R expression as its argument. We use `system.time()` when we want to analyze expressions one at a time.

```R
# system.time() Example
system.time(for(i in 1:1000) mean(sample(1:1000, 100)))
```

The above expression analyzes the input expression and returns its time in SECONDS???

#### `microbenchmark()`
The `microbenchmark()` function is easily the most useful timing function currently in R. If you're going to remember one method to time your code, this is the one. `microbenchmark()` facilitates comparing runtime between multiple functions. It takes in multiple functions as arguments and outputs summary runtime statistics. By default, it runs each functino 100 times and averages the results. You can change this option, as well as the unit of time to be measured via the `times` and `unit` arguments, respectively.

To give an example, we'll compare several implementations of a function computing the standard deviation for some function of numbers

```R
# microbenchmark example
library(microbenchmark)
set.seed(100)
x   <- rnorm(10000)
sd1 <- sd(x)
sd2 <- sqrt( var(x) )
sd3 <- var(x) ^ (1/2)
sd4 <- '^'(var(x), .5)
sd5 <- sqrt( sum( (x - mean(x))^2) / (length(x) - 1) )
microbenchmark( sd1, sd2, sd3, sd4, sd5, times=10000)
```
RETURN OUTPUT IN MARKDOWN TABLE 
When comparing the output from microbenchmark, look to the median output. You can also compare the lower quartile,`lq`, and the upper quartile, `uq`, to get a sense of the variance of the timings. From the above example, `sd3 <- var(x) ^ (1/2)` was our fastest implementation, but only slightly.

#### `proc.time()`
If we want to measure the speed of arbitrary chunks of our code (including the whole script itself), we use the `proc.time()` function. 

`proc.time()` works as follows:

* Preface the chunk of code you wish to time with `time_start <- proc.time()`. This initializes the timer.
* Add `proc.time() - time_start` to the line just after the end of the chunk of code you'd like to time.
In this manner, we can think of `proc.time()` as a sort of stop watch; we initialize a timer at the beginning of the code, time the code's duration, then initialize another timer at the end of the code and take the difference between our new timer and our old timer. The total duration of code is output under the `elapsed` column.

```R
# proc.time() example
time_start <- proc.time()
x <- runif(100)
y <- rnorm(200)
z <- rpois(400, lambda=2)
r <- sum(x,y,z)
proc.time() - time_start
```
MARKDOWN TABLE OUTPUT

A quick note - the above three functions are all related in the following ways:

* `system.time` calls `proc.time()` before and after the input expression.
* `microbenchmark` takes and compares the average number of `system.time()` calls. In our case, that's equivalent to calling `system.time(for (i in 1:100) sd(x)) / 100)`.

Timing functions is very important and an essential part of any efficient workflow, and the above functions should be sufficient for any time measurement needs that you'll encounter. 
  
***

#### Memory Pre-allocation
One of the most important aspects of memory management in R is pre-allocating your memory. An important application of this is when dealing with vectors or any multi-dimensional objects. It's always much more efficient to initialize an object of the desired size than to grow it iteratively. 

```R
# Memory Pre-Allocation Example
cum_prod_grow <- function(x) {
  obj <- c(x[1])
  for( i in 2:length(x)) {
    obj[i] <- x[i] + obj[i-1]
  }
  obj
}

cum_prod_init <- function(x) {
  obj <- vector(mode="numeric", length=length(x))
  obj[1] <- x[1]
  for( i in 2:length(x)) {
    obj[i] <- x[i] + obj[i-1]
  }
  obj
}

microbenchmark(cum_prod_init(c(1:10000)), cum_prod_grow(c(1:10000)), times=100)
```
TABLE OUTPUT
Thus, initializing an object of the correct size is much more efficient. This disparity only grows as the number of dimensions of our object increases.

***

#### Memoisation and Caching Variables

Another important aspect in memory management is caching variables. Caching variables refers to storing the value of a variable for future use. This storage shaves off time incrementally, but can have a big effect when used in conjunction with apply statements or loops.

Memoizaton is a popular concept in Computer Science that refers to the technique of storing the results of function calls and returning the cached result when the same inputs occur again. This is very easy to implement in R thanks to the `memoise` package.

Below I compare three functions: one with no variable caching, one with caching, and one that is fully memoised.

```R
# Cache Variable Example
no_cache <- function(x) {
  vec <- vector(length = length(x))
  for(i in seq_along(x)) {
    vec[i] <- sum(max(x), log(x), min(x), mean(x), i)
    vec[i] = vec[i] / length(x)
  }
  vec
}

si_cache <- function(x) {
  mx  <- max(x)
  mn  <- min(x)
  lx  <- log(x)
  ux  <- mean(x)
  n   <- length(x)
  vec <- vector(length = n)
  for(i in seq_along(x)) {
    vec[i] <- sum(mx, mn, lx, ux, 1)
    vec[i] = vec[i] / n
  }
  vec
}

library(memoise)
mem_cache <- memoise(no_cache)
microbenchmark(no_cache(c(1:1000)), si_cache(c(1:1000)), mem_cache(c(1:1000)))
```

<table border=1>
<tr> <th>  </th> <th> expr </th> <th> time </th>  </tr>
  <tr> <td align="right"> 1 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 21664754.00 </td> </tr>
  <tr> <td align="right"> 2 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3111895.00 </td> </tr>
  <tr> <td align="right"> 3 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20475551.00 </td> </tr>
  <tr> <td align="right"> 4 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 94455692.00 </td> </tr>
  <tr> <td align="right"> 5 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 134978.00 </td> </tr>
  <tr> <td align="right"> 6 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 74742.00 </td> </tr>
  <tr> <td align="right"> 7 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19272611.00 </td> </tr>
  <tr> <td align="right"> 8 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19716649.00 </td> </tr>
  <tr> <td align="right"> 9 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 18838515.00 </td> </tr>
  <tr> <td align="right"> 10 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 132388.00 </td> </tr>
  <tr> <td align="right"> 11 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 75462.00 </td> </tr>
  <tr> <td align="right"> 12 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2779735.00 </td> </tr>
  <tr> <td align="right"> 13 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 90134.00 </td> </tr>
  <tr> <td align="right"> 14 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 68493.00 </td> </tr>
  <tr> <td align="right"> 15 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 66825.00 </td> </tr>
  <tr> <td align="right"> 16 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19967859.00 </td> </tr>
  <tr> <td align="right"> 17 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21581413.00 </td> </tr>
  <tr> <td align="right"> 18 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3063169.00 </td> </tr>
  <tr> <td align="right"> 19 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2991705.00 </td> </tr>
  <tr> <td align="right"> 20 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2804301.00 </td> </tr>
  <tr> <td align="right"> 21 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19343773.00 </td> </tr>
  <tr> <td align="right"> 22 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 22181925.00 </td> </tr>
  <tr> <td align="right"> 23 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17541178.00 </td> </tr>
  <tr> <td align="right"> 24 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2636521.00 </td> </tr>
  <tr> <td align="right"> 25 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2638871.00 </td> </tr>
  <tr> <td align="right"> 26 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21360026.00 </td> </tr>
  <tr> <td align="right"> 27 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2748045.00 </td> </tr>
  <tr> <td align="right"> 28 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2741411.00 </td> </tr>
  <tr> <td align="right"> 29 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2779139.00 </td> </tr>
  <tr> <td align="right"> 30 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 140412.00 </td> </tr>
  <tr> <td align="right"> 31 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 73552.00 </td> </tr>
  <tr> <td align="right"> 32 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 68333.00 </td> </tr>
  <tr> <td align="right"> 33 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 66450.00 </td> </tr>
  <tr> <td align="right"> 34 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 66176.00 </td> </tr>
  <tr> <td align="right"> 35 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2698276.00 </td> </tr>
  <tr> <td align="right"> 36 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19246556.00 </td> </tr>
  <tr> <td align="right"> 37 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 127581.00 </td> </tr>
  <tr> <td align="right"> 38 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 72553.00 </td> </tr>
  <tr> <td align="right"> 39 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2697299.00 </td> </tr>
  <tr> <td align="right"> 40 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2637597.00 </td> </tr>
  <tr> <td align="right"> 41 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21898676.00 </td> </tr>
  <tr> <td align="right"> 42 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 144492.00 </td> </tr>
  <tr> <td align="right"> 43 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 83077.00 </td> </tr>
  <tr> <td align="right"> 44 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19891979.00 </td> </tr>
  <tr> <td align="right"> 45 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20523126.00 </td> </tr>
  <tr> <td align="right"> 46 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 130328.00 </td> </tr>
  <tr> <td align="right"> 47 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 18119406.00 </td> </tr>
  <tr> <td align="right"> 48 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19665520.00 </td> </tr>
  <tr> <td align="right"> 49 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 22389591.00 </td> </tr>
  <tr> <td align="right"> 50 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19401711.00 </td> </tr>
  <tr> <td align="right"> 51 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19459603.00 </td> </tr>
  <tr> <td align="right"> 52 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2675520.00 </td> </tr>
  <tr> <td align="right"> 53 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2901206.00 </td> </tr>
  <tr> <td align="right"> 54 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 135769.00 </td> </tr>
  <tr> <td align="right"> 55 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2636436.00 </td> </tr>
  <tr> <td align="right"> 56 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 99059.00 </td> </tr>
  <tr> <td align="right"> 57 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 68732.00 </td> </tr>
  <tr> <td align="right"> 58 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2646526.00 </td> </tr>
  <tr> <td align="right"> 59 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21281484.00 </td> </tr>
  <tr> <td align="right"> 60 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17963762.00 </td> </tr>
  <tr> <td align="right"> 61 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19515156.00 </td> </tr>
  <tr> <td align="right"> 62 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 136183.00 </td> </tr>
  <tr> <td align="right"> 63 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2724958.00 </td> </tr>
  <tr> <td align="right"> 64 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19283781.00 </td> </tr>
  <tr> <td align="right"> 65 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19955326.00 </td> </tr>
  <tr> <td align="right"> 66 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19316374.00 </td> </tr>
  <tr> <td align="right"> 67 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2713144.00 </td> </tr>
  <tr> <td align="right"> 68 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 132580.00 </td> </tr>
  <tr> <td align="right"> 69 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2765091.00 </td> </tr>
  <tr> <td align="right"> 70 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2712961.00 </td> </tr>
  <tr> <td align="right"> 71 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2755762.00 </td> </tr>
  <tr> <td align="right"> 72 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 22091405.00 </td> </tr>
  <tr> <td align="right"> 73 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19910694.00 </td> </tr>
  <tr> <td align="right"> 74 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17237635.00 </td> </tr>
  <tr> <td align="right"> 75 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19180578.00 </td> </tr>
  <tr> <td align="right"> 76 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 135029.00 </td> </tr>
  <tr> <td align="right"> 77 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 75192.00 </td> </tr>
  <tr> <td align="right"> 78 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2702819.00 </td> </tr>
  <tr> <td align="right"> 79 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 86913.00 </td> </tr>
  <tr> <td align="right"> 80 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 66356.00 </td> </tr>
  <tr> <td align="right"> 81 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19214109.00 </td> </tr>
  <tr> <td align="right"> 82 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 119572.00 </td> </tr>
  <tr> <td align="right"> 83 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19493637.00 </td> </tr>
  <tr> <td align="right"> 84 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 134329.00 </td> </tr>
  <tr> <td align="right"> 85 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19531738.00 </td> </tr>
  <tr> <td align="right"> 86 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17972086.00 </td> </tr>
  <tr> <td align="right"> 87 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 22825474.00 </td> </tr>
  <tr> <td align="right"> 88 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2684316.00 </td> </tr>
  <tr> <td align="right"> 89 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 135617.00 </td> </tr>
  <tr> <td align="right"> 90 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19470295.00 </td> </tr>
  <tr> <td align="right"> 91 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2819392.00 </td> </tr>
  <tr> <td align="right"> 92 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 125073.00 </td> </tr>
  <tr> <td align="right"> 93 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 71453.00 </td> </tr>
  <tr> <td align="right"> 94 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19527773.00 </td> </tr>
  <tr> <td align="right"> 95 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19499257.00 </td> </tr>
  <tr> <td align="right"> 96 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19579577.00 </td> </tr>
  <tr> <td align="right"> 97 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2750074.00 </td> </tr>
  <tr> <td align="right"> 98 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 139137.00 </td> </tr>
  <tr> <td align="right"> 99 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2728328.00 </td> </tr>
  <tr> <td align="right"> 100 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20374728.00 </td> </tr>
  <tr> <td align="right"> 101 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2641680.00 </td> </tr>
  <tr> <td align="right"> 102 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19346646.00 </td> </tr>
  <tr> <td align="right"> 103 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 134521.00 </td> </tr>
  <tr> <td align="right"> 104 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20153862.00 </td> </tr>
  <tr> <td align="right"> 105 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21125045.00 </td> </tr>
  <tr> <td align="right"> 106 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21131137.00 </td> </tr>
  <tr> <td align="right"> 107 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2663441.00 </td> </tr>
  <tr> <td align="right"> 108 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2720228.00 </td> </tr>
  <tr> <td align="right"> 109 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2627470.00 </td> </tr>
  <tr> <td align="right"> 110 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 137816.00 </td> </tr>
  <tr> <td align="right"> 111 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 73482.00 </td> </tr>
  <tr> <td align="right"> 112 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 68169.00 </td> </tr>
  <tr> <td align="right"> 113 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19162930.00 </td> </tr>
  <tr> <td align="right"> 114 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 113949.00 </td> </tr>
  <tr> <td align="right"> 115 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20300024.00 </td> </tr>
  <tr> <td align="right"> 116 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19386183.00 </td> </tr>
  <tr> <td align="right"> 117 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2679721.00 </td> </tr>
  <tr> <td align="right"> 118 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2628119.00 </td> </tr>
  <tr> <td align="right"> 119 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2633298.00 </td> </tr>
  <tr> <td align="right"> 120 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2663200.00 </td> </tr>
  <tr> <td align="right"> 121 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 134565.00 </td> </tr>
  <tr> <td align="right"> 122 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 75203.00 </td> </tr>
  <tr> <td align="right"> 123 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2780594.00 </td> </tr>
  <tr> <td align="right"> 124 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19906375.00 </td> </tr>
  <tr> <td align="right"> 125 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2714822.00 </td> </tr>
  <tr> <td align="right"> 126 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19370442.00 </td> </tr>
  <tr> <td align="right"> 127 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2703779.00 </td> </tr>
  <tr> <td align="right"> 128 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 132597.00 </td> </tr>
  <tr> <td align="right"> 129 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19697638.00 </td> </tr>
  <tr> <td align="right"> 130 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 136942.00 </td> </tr>
  <tr> <td align="right"> 131 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2810391.00 </td> </tr>
  <tr> <td align="right"> 132 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2667718.00 </td> </tr>
  <tr> <td align="right"> 133 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 119764.00 </td> </tr>
  <tr> <td align="right"> 134 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20481495.00 </td> </tr>
  <tr> <td align="right"> 135 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2902190.00 </td> </tr>
  <tr> <td align="right"> 136 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 129432.00 </td> </tr>
  <tr> <td align="right"> 137 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 71791.00 </td> </tr>
  <tr> <td align="right"> 138 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2736919.00 </td> </tr>
  <tr> <td align="right"> 139 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 90188.00 </td> </tr>
  <tr> <td align="right"> 140 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 67147.00 </td> </tr>
  <tr> <td align="right"> 141 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2660229.00 </td> </tr>
  <tr> <td align="right"> 142 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19281736.00 </td> </tr>
  <tr> <td align="right"> 143 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2669348.00 </td> </tr>
  <tr> <td align="right"> 144 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2722921.00 </td> </tr>
  <tr> <td align="right"> 145 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 123197.00 </td> </tr>
  <tr> <td align="right"> 146 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2647562.00 </td> </tr>
  <tr> <td align="right"> 147 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2639247.00 </td> </tr>
  <tr> <td align="right"> 148 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17148133.00 </td> </tr>
  <tr> <td align="right"> 149 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19375300.00 </td> </tr>
  <tr> <td align="right"> 150 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 129918.00 </td> </tr>
  <tr> <td align="right"> 151 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19625103.00 </td> </tr>
  <tr> <td align="right"> 152 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 135451.00 </td> </tr>
  <tr> <td align="right"> 153 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20506587.00 </td> </tr>
  <tr> <td align="right"> 154 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3041171.00 </td> </tr>
  <tr> <td align="right"> 155 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2926084.00 </td> </tr>
  <tr> <td align="right"> 156 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2772743.00 </td> </tr>
  <tr> <td align="right"> 157 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 135965.00 </td> </tr>
  <tr> <td align="right"> 158 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2626310.00 </td> </tr>
  <tr> <td align="right"> 159 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2693350.00 </td> </tr>
  <tr> <td align="right"> 160 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19150403.00 </td> </tr>
  <tr> <td align="right"> 161 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2653646.00 </td> </tr>
  <tr> <td align="right"> 162 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 126819.00 </td> </tr>
  <tr> <td align="right"> 163 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3343222.00 </td> </tr>
  <tr> <td align="right"> 164 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3361774.00 </td> </tr>
  <tr> <td align="right"> 165 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2636025.00 </td> </tr>
  <tr> <td align="right"> 166 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2648659.00 </td> </tr>
  <tr> <td align="right"> 167 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 127192.00 </td> </tr>
  <tr> <td align="right"> 168 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 71142.00 </td> </tr>
  <tr> <td align="right"> 169 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2682489.00 </td> </tr>
  <tr> <td align="right"> 170 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 82411.00 </td> </tr>
  <tr> <td align="right"> 171 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 66365.00 </td> </tr>
  <tr> <td align="right"> 172 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2656997.00 </td> </tr>
  <tr> <td align="right"> 173 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 78900.00 </td> </tr>
  <tr> <td align="right"> 174 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2680517.00 </td> </tr>
  <tr> <td align="right"> 175 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2695942.00 </td> </tr>
  <tr> <td align="right"> 176 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19506974.00 </td> </tr>
  <tr> <td align="right"> 177 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2665098.00 </td> </tr>
  <tr> <td align="right"> 178 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 125728.00 </td> </tr>
  <tr> <td align="right"> 179 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17971973.00 </td> </tr>
  <tr> <td align="right"> 180 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2721090.00 </td> </tr>
  <tr> <td align="right"> 181 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 137714.00 </td> </tr>
  <tr> <td align="right"> 182 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 74033.00 </td> </tr>
  <tr> <td align="right"> 183 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 22015470.00 </td> </tr>
  <tr> <td align="right"> 184 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 155354.00 </td> </tr>
  <tr> <td align="right"> 185 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 87409.00 </td> </tr>
  <tr> <td align="right"> 186 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 80591.00 </td> </tr>
  <tr> <td align="right"> 187 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 79395.00 </td> </tr>
  <tr> <td align="right"> 188 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20273326.00 </td> </tr>
  <tr> <td align="right"> 189 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2760774.00 </td> </tr>
  <tr> <td align="right"> 190 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19519381.00 </td> </tr>
  <tr> <td align="right"> 191 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19312592.00 </td> </tr>
  <tr> <td align="right"> 192 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2656242.00 </td> </tr>
  <tr> <td align="right"> 193 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17416927.00 </td> </tr>
  <tr> <td align="right"> 194 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19330580.00 </td> </tr>
  <tr> <td align="right"> 195 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 130805.00 </td> </tr>
  <tr> <td align="right"> 196 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2664298.00 </td> </tr>
  <tr> <td align="right"> 197 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 97606.00 </td> </tr>
  <tr> <td align="right"> 198 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19242671.00 </td> </tr>
  <tr> <td align="right"> 199 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19295901.00 </td> </tr>
  <tr> <td align="right"> 200 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 126340.00 </td> </tr>
  <tr> <td align="right"> 201 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 68925.00 </td> </tr>
  <tr> <td align="right"> 202 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 66719.00 </td> </tr>
  <tr> <td align="right"> 203 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 65314.00 </td> </tr>
  <tr> <td align="right"> 204 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 65633.00 </td> </tr>
  <tr> <td align="right"> 205 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20394018.00 </td> </tr>
  <tr> <td align="right"> 206 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20437812.00 </td> </tr>
  <tr> <td align="right"> 207 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2719383.00 </td> </tr>
  <tr> <td align="right"> 208 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2733765.00 </td> </tr>
  <tr> <td align="right"> 209 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17238180.00 </td> </tr>
  <tr> <td align="right"> 210 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 138090.00 </td> </tr>
  <tr> <td align="right"> 211 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2659473.00 </td> </tr>
  <tr> <td align="right"> 212 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19330680.00 </td> </tr>
  <tr> <td align="right"> 213 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 124069.00 </td> </tr>
  <tr> <td align="right"> 214 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 71511.00 </td> </tr>
  <tr> <td align="right"> 215 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19375667.00 </td> </tr>
  <tr> <td align="right"> 216 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19561135.00 </td> </tr>
  <tr> <td align="right"> 217 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2850810.00 </td> </tr>
  <tr> <td align="right"> 218 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 4654204.00 </td> </tr>
  <tr> <td align="right"> 219 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2971198.00 </td> </tr>
  <tr> <td align="right"> 220 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 146765.00 </td> </tr>
  <tr> <td align="right"> 221 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3112541.00 </td> </tr>
  <tr> <td align="right"> 222 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19391481.00 </td> </tr>
  <tr> <td align="right"> 223 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2704427.00 </td> </tr>
  <tr> <td align="right"> 224 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 123508.00 </td> </tr>
  <tr> <td align="right"> 225 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2697984.00 </td> </tr>
  <tr> <td align="right"> 226 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2658985.00 </td> </tr>
  <tr> <td align="right"> 227 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2681362.00 </td> </tr>
  <tr> <td align="right"> 228 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2698945.00 </td> </tr>
  <tr> <td align="right"> 229 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 120101.00 </td> </tr>
  <tr> <td align="right"> 230 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2677171.00 </td> </tr>
  <tr> <td align="right"> 231 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19350888.00 </td> </tr>
  <tr> <td align="right"> 232 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 22096425.00 </td> </tr>
  <tr> <td align="right"> 233 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3040343.00 </td> </tr>
  <tr> <td align="right"> 234 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3485444.00 </td> </tr>
  <tr> <td align="right"> 235 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 18901840.00 </td> </tr>
  <tr> <td align="right"> 236 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19875070.00 </td> </tr>
  <tr> <td align="right"> 237 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19553241.00 </td> </tr>
  <tr> <td align="right"> 238 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2716291.00 </td> </tr>
  <tr> <td align="right"> 239 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 134859.00 </td> </tr>
  <tr> <td align="right"> 240 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2798811.00 </td> </tr>
  <tr> <td align="right"> 241 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2674498.00 </td> </tr>
  <tr> <td align="right"> 242 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 133393.00 </td> </tr>
  <tr> <td align="right"> 243 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19429545.00 </td> </tr>
  <tr> <td align="right"> 244 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 129076.00 </td> </tr>
  <tr> <td align="right"> 245 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20035010.00 </td> </tr>
  <tr> <td align="right"> 246 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 137005.00 </td> </tr>
  <tr> <td align="right"> 247 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2662857.00 </td> </tr>
  <tr> <td align="right"> 248 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 99295.00 </td> </tr>
  <tr> <td align="right"> 249 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19337297.00 </td> </tr>
  <tr> <td align="right"> 250 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17268090.00 </td> </tr>
  <tr> <td align="right"> 251 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19279762.00 </td> </tr>
  <tr> <td align="right"> 252 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 123551.00 </td> </tr>
  <tr> <td align="right"> 253 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 71550.00 </td> </tr>
  <tr> <td align="right"> 254 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21224965.00 </td> </tr>
  <tr> <td align="right"> 255 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2710790.00 </td> </tr>
  <tr> <td align="right"> 256 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2639719.00 </td> </tr>
  <tr> <td align="right"> 257 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 138430.00 </td> </tr>
  <tr> <td align="right"> 258 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2852254.00 </td> </tr>
  <tr> <td align="right"> 259 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 122944.00 </td> </tr>
  <tr> <td align="right"> 260 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 70403.00 </td> </tr>
  <tr> <td align="right"> 261 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21583854.00 </td> </tr>
  <tr> <td align="right"> 262 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2689058.00 </td> </tr>
  <tr> <td align="right"> 263 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 134470.00 </td> </tr>
  <tr> <td align="right"> 264 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2680158.00 </td> </tr>
  <tr> <td align="right"> 265 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21299586.00 </td> </tr>
  <tr> <td align="right"> 266 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 3753976.00 </td> </tr>
  <tr> <td align="right"> 267 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 225020.00 </td> </tr>
  <tr> <td align="right"> 268 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19125043.00 </td> </tr>
  <tr> <td align="right"> 269 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2624222.00 </td> </tr>
  <tr> <td align="right"> 270 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 124811.00 </td> </tr>
  <tr> <td align="right"> 271 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19473331.00 </td> </tr>
  <tr> <td align="right"> 272 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2653490.00 </td> </tr>
  <tr> <td align="right"> 273 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19378283.00 </td> </tr>
  <tr> <td align="right"> 274 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 137304.00 </td> </tr>
  <tr> <td align="right"> 275 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 75179.00 </td> </tr>
  <tr> <td align="right"> 276 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19549258.00 </td> </tr>
  <tr> <td align="right"> 277 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19285910.00 </td> </tr>
  <tr> <td align="right"> 278 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 136742.00 </td> </tr>
  <tr> <td align="right"> 279 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 72688.00 </td> </tr>
  <tr> <td align="right"> 280 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2688030.00 </td> </tr>
  <tr> <td align="right"> 281 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2702965.00 </td> </tr>
  <tr> <td align="right"> 282 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 112412.00 </td> </tr>
  <tr> <td align="right"> 283 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19716064.00 </td> </tr>
  <tr> <td align="right"> 284 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 129098.00 </td> </tr>
  <tr> <td align="right"> 285 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2658686.00 </td> </tr>
  <tr> <td align="right"> 286 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 99674.00 </td> </tr>
  <tr> <td align="right"> 287 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2706701.00 </td> </tr>
  <tr> <td align="right"> 288 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19407119.00 </td> </tr>
  <tr> <td align="right"> 289 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 17481076.00 </td> </tr>
  <tr> <td align="right"> 290 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 20428677.00 </td> </tr>
  <tr> <td align="right"> 291 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19667938.00 </td> </tr>
  <tr> <td align="right"> 292 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 137394.00 </td> </tr>
  <tr> <td align="right"> 293 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19335036.00 </td> </tr>
  <tr> <td align="right"> 294 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 4334781.00 </td> </tr>
  <tr> <td align="right"> 295 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 21437605.00 </td> </tr>
  <tr> <td align="right"> 296 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 133687.00 </td> </tr>
  <tr> <td align="right"> 297 </td> <td> no_cache(c(1:1000)) </td> <td align="right"> 19363564.00 </td> </tr>
  <tr> <td align="right"> 298 </td> <td> mem_cache(c(1:1000)) </td> <td align="right"> 116848.00 </td> </tr>
  <tr> <td align="right"> 299 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2633042.00 </td> </tr>
  <tr> <td align="right"> 300 </td> <td> si_cache(c(1:1000)) </td> <td align="right"> 2710614.00 </td> </tr>
   </table>

Thus, caching values increases a function speeds more than two-fold, while memoising increases the speed almost four-fold.

***

#### Compile Code

In R, compiling our code is a quick, easy way to have it run more efficiently. To achieve this, we'll use the `compiler` package, which compiles an expression into a byte code object What does this mean, and why is it effective? Byte code is simply machine code for a virtual machine. In general, lower level languages (e.g. C), compile their source code to machine code. Other languages, such as Python, compile their code to byte code.

In standard R, expressions are parsed into a parse tree, which is then interpreted upon evaluation. With the `compiler` package, we can convert R's expressions into byte code, to be evalutaed via a stack-based virtual machine architecture. In the vast majority of cases this greatly improves runtime.

```R
# compiler example
library(compiler)
```


### Index data frames as lists

#### Use seq_len and seq_along()
When uses sequences (espeically in loops), avoid using `1:length(x)`. This is because this can sometimes lead to strange erros. Instead, best practice dictates that you should 
use `seq_along(x)` or `seq_len(length(x))`. The three are essentially identical in speed (test this for yourself).

***

#### Memory In Use

To get pertinent information regarding memory used in session, you can use the `pryr` package. The `pryr` package contains multiple useful functions that yield memory usage information.

* `mem_change`: Returns change in memory (megabytes) before and after running the code.
* `mem_used`: Returns total megabytes of ram used by R. 
* `object_size`: Returns an estimate of the size of the object (bytes)


#### Useful Memory Facts

* R counts the number of names that point to each object in our workspace, and deleted objects that no longer have names pointing to them. In this way, R automatically relases memory from objects that are no longer being used. This is known as _garbage colection_, and while in some languages it's required to call this on your own, R does it automatically.

* Every argument in a function slows down the function call by about 20 nanoseconds. Why is this the case? Every argument creates a promise object that creates an expression required to carry out the result as well as the environment in which to carray out the expression. For this reason, it's good practice to limit the number of function arguments when applicable.

***

### Conclusion

While the above list certainly wasn't exhaustive, I hope that you learned something new. Each of the above topics can be used to your benefit when incorporated into your R workflow. Thanks for reading, and if you have an questions or comments please let me know!
