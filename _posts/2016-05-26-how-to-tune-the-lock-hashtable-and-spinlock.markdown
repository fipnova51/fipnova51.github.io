---
published: true
title: How to tune the lock hashtable and spinlock
layout: post
tags: [sybase, lock, hashtable, spinlock]
categories: [database]
---
* In this post, we'll review how the locking mechanism works and how to tune it *

<!--excerpt-->

Briefly, when a memory area is shared we need a way to lock it before doing the modifucation with mutexes for example. in ASE, the mechanism is called spinlock.

Then the locking mechanism is managed with a lock hashtable and simple serial chain. For example, we have 10 hash buckets and the hash function is mod(). Then the `spinlock ratio` is 3 meaning each spinlock will manage 3 buckets so we end up with 4 spinlocks.

If we want an excluive lock on page 25, then 25%10 = 5 so we need to scan hash bucket 5 to see if a lock is already on this page. whether the page is lock or not, we need to add our lock to the lock chain so we grab the spinlock that is managing hash bucket 5, modify the lock chain and release the spinlock. See the picture below for a better understanding.

![spinlock]({{ site.url }}/assets/pictures/sybase_spinlocks.png)

