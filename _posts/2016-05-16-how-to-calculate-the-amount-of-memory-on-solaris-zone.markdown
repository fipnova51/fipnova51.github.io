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

     kstat -n system_pages -p -s physmem | nawk -v pagesize=$(pagesize) '{print $2*pagesize/1024/1024 "MB"}'

Example:

~~~
mx599zn autoengine /tmp/
bash$ kstat -n system_pages -p -s physmem | nawk -v pagesize=$(pagesize) '{print $2*pagesize/1024/1024 "MB"}'
514296MB
~~~