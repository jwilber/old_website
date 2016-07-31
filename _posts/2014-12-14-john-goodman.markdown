---
layout: post
title:  "John Goodman: Hit Status"
date:   2014-12-14
image: touring.jpg
---

<p class="intro"><span class="dropcap">D</span> oes John Goodman have a significant effect on a movie becoming a hit or not? Find out via a permutation test application. [CURRENTLY IN PROGRESS] </p>


# J G

This is currently in Progress.

{% highlight R %}
diffMeans <- function(data, hasTrt){
  # computes our test statistics: the difference of means
  # hasTrt: boolean vector, TRUE if has treatment
  test_stat <- mean(data[hasTrt]) - mean(data[!hasTrt])
  return(test_stat)
}
{% endhighlight %}
