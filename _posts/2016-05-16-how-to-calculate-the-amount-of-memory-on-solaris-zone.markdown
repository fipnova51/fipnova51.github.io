---
published: true
title: How to calculate the amount of memory on Solaris zone
layout: post
tags: [solaris, zone, memory]
categories: [system]
---
*prtconf is not working on a non-global zone. How to find out the allocated memory?*

<!--excerpt-->

Use the command below

    echo $(( $( kstat :::physmem | tail -2 | head -1 | awk '{print $NF}') * $(pagesize) / (1024  * 1024) ))

Example:

~~~
mx599zn autoengine /tmp/
bash$ echo $(( $( kstat :::physmem | tail -2 | head -1 | awk '{print $NF}') * $(pagesize) / (1024  * 1024) ))
514296
~~~