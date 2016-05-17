---
published: true
title: How to calculate the total  memory on Solaris physical server?
layout: post
tags: [solaris, zone, memory]
categories: [system]
---
*prtconf will return the allocated memory for a zone, what if you want to know the total amount of memory on the physical server?*

<!--excerpt-->

Use the command below

     kstat -n system_pages -p -s physmem | nawk -v pagesize=$(pagesize) '{print $2*pagesize/1024/1024 "MB"}'

Example:

~~~
mx599zn autoengine /tmp/
bash$ kstat -n system_pages -p -s physmem | nawk -v pagesize=$(pagesize) '{print $2*pagesize/1024/1024 "MB"}'
514296MB
~~~