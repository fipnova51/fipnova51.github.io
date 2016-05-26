---
published: true
title: How to tune the lock hashtable and spinlock
layout: post
tags: [sybase, lock, hashtable, spinlock]
categories: [database]
---
*In this post, we'll review how the locking mechanism works and how to tune it*

<!--excerpt-->

Briefly, when a memory area is shared we need a way to lock it before doing the modifucation with mutexes for example. in ASE, the mechanism is called spinlock.

Then the locking mechanism is managed with a lock hashtable and simple serial chain. For example, we have 10 hash buckets and the hash function is mod(). Then the `spinlock ratio` is 3 meaning each spinlock will manage 3 buckets so we end up with 4 spinlocks.

If we want an excluive lock on page 25, then 25%10 = 5 so we need to scan hash bucket 5 to see if a lock is already on this page. whether the page is lock or not, we need to add our lock to the lock chain so we grab the spinlock that is managing hash bucket 5, modify the lock chain and release the spinlock. See the picture below for a better understanding.

![spinlock]({{ site.url }}/assets/pictures/sybase_spinlocks.png)

The idea of tuning is to reduce the chain lenght for each bucket to less than 5. The information is available in sp_sysmon output

![sp_sysmon spinlock]({{ site.url }}/assets/pictures/sybase_sp_sysmon_spinlocks.png)


Sybase maintains two lock hashtable: one for table lock and one for page/row locks.

The `table lock hashtable` has a unique size of 101 hash buckets and a default configuration of `lock table spinlock ratio` of 20. This parameter says that the 101 hash buckets list is managed by 20 spinlock so roughly each spinlock will manage 6 hash buckets. There aren't many possibilities for tuning.

~~~
For Adaptive Servers running with multiple engines, table lock spinlock ratio sets the number of rows in the internal table locks hash table that are protected by one spinlock
~~~

Now for the `page/row lock hashtable`, its size can be changed with `lock hashtable size` (default is 2048) and the spinlock ration is controled by `lock spinlock ratio` (default 85). So with those default config, each spinlock will manage 85 hash buckets so only 26 spinlocks are managing the lock hashtable.

~~~
For Adaptive Servers running with multiple engines, lock spinlock ratio sets a ratio that determines the number of lock hash buckets that are protected by one spinlock
~~~

So based on those 2 parameters, if there are some spinlock contention, reduce parameter `lock spinlock ratio`.
Another way is to increase the hashtable size but by how many? a quick rules of thumb is `number of locks used / 5` (remember the value 5 is coming from PT recommendation). So if we have 250000 locks then we would have a hashtable of 250000/5=50000 buckets and with the default `lock spinlock ratio` of 85 would lead to 600 spinlocks