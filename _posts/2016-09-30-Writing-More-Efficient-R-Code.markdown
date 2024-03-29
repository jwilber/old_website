---
layout: post
title:  "Writing Efficient R Code"
date:   2016-12-08
---


<p class="intro"><span class="dropcap">T</span></p>he goal of this post is to (hopefully) introduce you to some methods that will aid you in writing more efficient R code. While I am by no means an R expert, I've picked up some techniques in my few years of using R that will hopefully prove beneficial to you!


### Why?
R is a programming language primarily used for data science and statistical analysis. 
While R is complimented by many when it comes to data analysis, it's definitely not praised for its efficiency (specifically, it's speed or memory management). That's not to say that R isn't very efficient; for the vast majority of cases, it will get the job done and do it well. However, it is true that R is not as efficient as it can be. This stems from two primary reasons

1. R is very, very dynamic. Although this dynamism gives us more flexibility, when coupled with lexical scoping it results in slower code.
2. The current most popular implementation of R (gnu-R, this is what you have if you don't know which implementatin of R you have) is far from peak-optimization with regards to the current implementation.

That said, the point of this post isn't discussing R's current implementation, it's to help you write better R code. If you would like to learn more about R, I suggest reading either [BOOK OR BOOK LINK]

Here is an outline of what I will cover:

* Time your code
  1. `system.time()`
  2. `microbenchmark()`
  3. `proc.time()`

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

For example, we can analyze the time it takes to loop through 1000 calculations of calling the mean function on a random sample of points:

```R
# system.time() Example
system.time(for(i in 1:1000) mean(sample(1:1000, 100)))
```

| user  | system | elapsed |
|-------|---------|---------|
| 0.020 | 0.016   | 0.059   |

The above expression analyzes the input expression and returns its time in seconds. The important column to pay attention to is the _elapsed_ column, as this reveals the total elapsed time required to evaluate our provided expression.

#### `microbenchmark()`
The `microbenchmark()` function is easily the most useful timing function currently in R. If you're going to remember one method to time your code, this is the one. `microbenchmark()` facilitates comparing runtime between multiple function calls. It takes in multiple functions as arguments and outputs summary runtime statistics. By default, it runs each function 100 times and averages the results. You can change this option, as well as the unit of time to be measured via the `times` and `unit` arguments, respectively.

To give an example, we'll compare the runtimes for several different implementations of the standard deviation function for a group of numbers.

Recall the formula for standard deviation is as follows: [insert latex]

$$ s = \sqrt{\frac{1}{N-1} \sum_{i=1}^N (x_i - \overline{x})^2} $$



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


| expr | min | lq | mean    | median | uq  | max  | neval |
|------|-----|----|---------|--------|-----|------|-------|
| sd1  | 33  | 45 | 52.8414 | 45     | 48 | 11362 | 10000 |
| sd2  | 34  | 48 | 71.0916 | 47     | 50 | 63875 | 10000 |
| sd3  | 34  | 46 | 55.5363 | 45     | 48 | 10846 | 10000 |
| sd4  | 39  | 51 | 61.8710 | 48     | 51 | 14515 | 10000 |
| sd5  | 39  | 49 | 54.1965 | 46     | 48 | 11086 | 10000 |

When comparing the output from microbenchmark, look to the median output. You can also compare the lower quartile,`lq`, and the upper quartile, `uq`, to get a sense of the variance of the timings. From the above example, `sd3 <- var(x) ^ (1/2)` was our fastest implementation, but only slightly.

#### `proc.time()`
If we want to measure the speed of arbitrary chunks of our code (including the whole script itself), we use the `proc.time()` function. 

`proc.time()` works as follows:

* Preface the chunk of code you wish to time with `time_start <- proc.time()`. This initializes the timer.
* Add `proc.time() - time_start` to the line just after the end of the chunk of code you'd like to time.
In this manner, we can think of `proc.time()` as a sort of stop watch: we initialize a timer at the beginning of the code, time the code's duration, then initialize another timer at the end of the code and take the difference between our new timer and our old timer. The total duration of code is output under the `elapsed` column.

```R
# proc.time() example
time_start <- proc.time()
x <- runif(100)
y <- rnorm(200)
z <- rpois(400, lambda=2)
r <- sum(x,y,z)
proc.time() - time_start
```

| user  | system | elapsed |
|-------|---------|---------|
| 0.004 | 0.014   | 0.045   |

A quick note - the above three functions are all related in the following ways:

* `system.time` calls `proc.time()` before and after the input expression.
* `microbenchmark` takes and compares the average number of `system.time()` calls. In our case, that's equivalent to calling `system.time(for (i in 1:100) sd(x)) / 100)`.

Timing functions is very important and an essential part of any efficient workflow, and the above functions should be sufficient for any time measurement needs that you'll encounter. 
  
***

#### Memory Pre-allocation
One of the most important aspects of memory management in R is pre-allocating your memory. An important application of this is when dealing with vectors or any multi-dimensional objects. It's always much more efficient to initialize an object of the desired size than to grow it iteratively. Let's see this for ourselves:

```R
# Memory Pre-Allocation Example
cum_prod_grow <- function(x) {
  obj      <- c(x[1])
  for( i in 2:length(x)) {
    obj[i] <- x[i] + obj[i-1]
  }
  obj
}

cum_prod_init <- function(x) {
  obj      <- vector(mode="numeric", length=length(x))
  obj[1]   <- x[1]
  for( i in 2:length(x)) {
    obj[i] <- x[i] + obj[i-1]
  }
  obj
}

microbenchmark(cum_prod_init(c(1:10000)), cum_prod_grow(c(1:10000)), times=100)
```


| expr                      | min       | lq        | mean      | median     | uq        | max       | neval |
|---------------------------|-----------|-----------|-----------|------------|-----------|-----------|-------|
| `cum_prod_init(c(1:10000))` | 8.99947   | 9.19716   | 11.46031  | 9.581028   | 11.51252  | 26.54054  | 100   |
| `cum_prod_grow(c(1:10000))` | 96.828066 | 110.18554 | 128.99623 | 114.860681 | 130.11597 | 221.84280 | 100   |

Thus, initializing an object of the correct size is eleven times faster. This disparity only grows as the number of dimensions of our object increases.

***

#### Memoisation and Caching Variables

Another important aspect in memory management is caching variables. Caching variables refers to storing the value of a variable for future use. This storage shaves off time incrementally, but can have a big effect when used in conjunction with apply statements or loops.

Memoizaton is a popular optimization technique in caching and refers to the technique of storing the results of function calls and returning the cached result when the same inputs occur again. This is very easy to implement in R thanks to the `memoise` package.


Below I compare three functions: one with no variable caching, one where we cache our variables, and one that is fully memoised.

```R
# Cache Variable Example
no_cache  <- function(x) {
  vec      <- vector(length = length(x))
  for(i in seq_along(x)) {
    vec[i] <- sum(max(x), log(x), min(x), mean(x), i)
    vec[i] <- vec[i] / length(x)
  }
  vec
}

si_cache <- function(x) {
  mx       <- max(x)
  mn       <- min(x)
  lx       <- log(x)
  ux       <- mean(x)
  n        <- length(x)
  vec      <- vector(length = n)
  for(i in seq_along(x)) {
    vec[i] <- sum(mx, mn, lx, ux, 1)
    vec[i] <- vec[i] / n
  }
  vec
}

library(memoise)
mem_cache  <- memoise(no_cache)
microbenchmark(no_cache(c(1:1000)), si_cache(c(1:1000)), mem_cache(c(1:1000)))
```

| expr                 | min       | lq        | mean       | median   | uq         | max        | neval |
|----------------------|-----------|-----------|------------|----------|------------|------------|-------|
| no_cache(c(1:1000))  | 17361.978 | 20183.470 | 22221.3387 | 21419.49 | 22253.2410 | 110095.342 | 100   |
| si_cache(c(1:1000))  | 2631.881  | 2683.565  | 2785.6216  | 2738.47  | 2806.6720  | 3510.548   | 100   |
| mem_cache(c(1:1000)) | 67.733    | 80.592    | 116.8227   | 130.80   | 138.5335   | 177.456    | 100   |

We can see the difference each method makes: caching variables is almost ten times faster, while memoising is almost 200 times faster. Thus, when writing time-consuming code, it's important to consider whether or not the opportunity for caching or memoisation exists. If you can find any examples of the same value being called over-and-over again, you probably have a case for either option.

***

#### Compile Code

In R, compiling our code is a quick, easy way to have it run more efficiently. To achieve this, we'll use the `compiler` package, which compiles an expression into a byte code object What does this mean, and why is it effective? Byte code is simply machine code for a virtual machine. In general, lower level languages (e.g. C), compile their source code to machine code. Other languages, such as Python, compile their code to byte code.

In standard R, expressions are parsed into a parse tree, which is then interpreted upon evaluation. With the `compiler` package, we can convert R's expressions into byte code, to be evalutaed via a stack-based virtual machine architecture. In the vast majority of cases this greatly improves runtime.

[ insert example ]

```R
# compiler example
library(compiler)
```

***

#### Memory In Use

To get pertinent information regarding memory used in session, you can use the `pryr` package. The `pryr` package contains multiple useful functions that yield memory usage information. Some examples are below:

* `mem_change`: Returns change in memory (megabytes) before and after running the code.
* `mem_used`: Returns total megabytes of ram used by R. 
* `object_size`: Returns an estimate of the size of the object (bytes)


#### Useful Memory Facts

* R counts the number of names that point to each object in our workspace, and deleted objects that no longer have names pointing to them. In this way, R automatically relases memory from objects that are no longer being used. This is known as _garbage colection_, and while in some languages it's required to call this on your own, R does it automatically.

* Every argument in a function slows down the function call by about 20 nanoseconds. Why is this the case? Every argument creates a promise object that creates an expression required to carry out the result as well as the environment in which to carray out the expression. For this reason, it's good practice to limit the number of function arguments when applicable.

***

### Conclusion

While the above list certainly wasn't exhaustive, I hope that you learned something new. Each of the above topics can be used to your benefit when incorporated into your R workflow. Thanks for reading, and if you have an questions or comments please let me know!
