---
layout: post
title:  "Scrape Zillow - Hundreds of Thousands of Properties Pt. 2 w/o Tax Data"
date:   2019-02-10 19:31:43 +0700
categories: [python, web scraping, Zillow]
---
This tutortial will walk you through how to scrape hundreds of thousands of Zillow property data. This article assumes you have Zillow property URLs and ZPIDs already or have already read the Scrape Zillow - Hundreds of Thousands of Properties Pt. 1 article.

Prerequites

1. Python 2
2. Libraries: itertools, lxml, gevent, codecs, sys, json, time

{% highlight ruby %}
>>> from itertools import permutations
>>> perms = [''.join(p)+"@gmail.com" for p in permutations('abc', 3)]
>>> for x in range(0, len(perms)):
...     print (perms[x])
... 
abc@gmail.com
acb@gmail.com
bac@gmail.com
bca@gmail.com
cab@gmail.com
cba@gmail.com
>>> 
{% endhighlight %}
