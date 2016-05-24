---
published: true
title: How to tune the housekeeper process
layout: post
tags: [sybase, housekeeper]
categories: [database]
---
*This note explains the housekkeper and how to tune it*

<!--excerpt-->

First of all, there are three types of housekeeper tasks

* `Housekeeper wash` that will flush dirty pages to disks as I/O limits allow. This task only runs at idle and is controlled by the `housekeeper free write percent`
* `Housekeeper garbage collection` that will remove deallocated pages
* `Housekeeper chores` that will flush statistics information at idle times
