---
layout: post
title:  "Geospatial Plot Storage"
date:   2014-12-14
image: touring.jpg
---

Just a bunch of geospatial related plots I've done. Dumping them here.


<img src="/images/geo1.png" />
<img src="/images/geo2.png" />
<img src="/images/geo3.png" />
<img src="/images/geo4.png" />
<img src="/images/geomath.png" />

<img src="/images/gp.png" />
<img src="/images/dt1.png" />
<img src="/images/dt2.png" />
<img src="/images/dt2.png" />
<img src="/images/dt4.png" />
<img src="/images/dtlast.png" />


<img src="/images/tiled.png" />
<img src="/images/D-mat.png" />
<img src="/images/L-mat.png" />
<img src="/images/lab6q3fin2.png" />
<img src="/images/lab7q3fin3.png" />
<img src="/images/lab6q3fin1.png" />
<img src="/images/fin.png" />
<img src="/images/lab7part2plot2.png" />
<img src="/images/lab7part2plot3.png" />
<img src="/images/lab7part2plot4.png" />
<img src="/images/lab7part2plot5.png" />
<img src="/images/lab7part2plot1.png" />
<img src="/images/q2scatter1.png" />
<img src="/images/q2scatter2.png" />
<img src="/images/q2scatter3.png" />
<img src="/images/l7p1.png" />
<img src="/images/l7p2.png" />









<img src="/images/repi2.png" />
<img src="/images/repi3.png" />
<img src="/images/repi4.png" />
<img src="/images/repi5.png" />
<img src="/images/repi6.png" />
<img src="/images/repi7.png" />
<img src="/images/repi8.png" />
<img src="/images/repi9.png" />
<img src="/images/repi10.png" />

<img src="/images/movreg.png" />
<img src="/images/lab6p6.png" />
<img src="/images/proj3.png" />
<img src="/images/proj3p2.png" />
<img src="/images/l6p21.png" />
<img src="/images/l6p22.png" />
<img src="/images/l6p222.png" />
<img src="/images/l6p23.png" />
<img src="/images/l6p8.png" />
<img src="/images/l6p7.png" />
<img src="/images/l6p5.png" />
<img src="/images/l6p51.png" />
<img src="/images/l6p52.png" />
<img src="/images/lab6p4.png" />
<img src="/images/l6p3.png" />
<img src="/images/l6p2.png" />
<img src="/images/l6p1.png" />







<img src="/images/lab5p1.png" />
<img src="/images/lab5p2.png" />
<img src="/images/lab5p3.png" />
<img src="/images/lab5p4.png" />
<img src="/images/lab5p5.png" />
<img src="/images/lab5p6.png" />
<img src="/images/lab5p62.png" />
<img src="/images/lab5p7.png" />
<img src="/images/lab5p8.png" />
<img src="/images/lab5p9.png" />
<img src="/images/lab5p10.png" />
<img src="/images/lab5p11.png" />
<img src="/images/lab5p12.png" />
<img src="/images/lab5p13.png" />









<img src="/images/q1p1.png" />
<img src="/images/q1p2.png" />
<img src="/images/q2p1.png" />
<img src="/images/q2p2.png" />
<img src="/images/q3.png" />
<img src="/images/q4p1.png" />
<img src="/images/q4p2.png" />







<img src="/images/plot1.png" />

<img src="/images/plot2.png" />

<img src="/images/matlabhw2plot.png" />


## Lab 3
<img src="/images/crimes1.png" />
<img src="/images/crimes2.png" />
<img src="/images/crimes3.png" />
<img src="/images/ppp1.png" />
<img src="/images/ppp2.png" />
<img src="/images/ppp3.png" />
<img src="/images/ppp4.png" />
<img src="/images/ppp5.png" />
<img src="/images/cholera1.png" />
<img src="/images/cholera2.png" />
<img src="/images/cholera3.png" />
<img src="/images/cholera4.png" />




{% highlight R %}
diffMeans <- function(data, hasTrt){
  # computes our test statistics: the difference of means
  # hasTrt: boolean vector, TRUE if has treatment
  test_stat <- mean(data[hasTrt]) - mean(data[!hasTrt])
  return(test_stat)
}
{% endhighlight %}
