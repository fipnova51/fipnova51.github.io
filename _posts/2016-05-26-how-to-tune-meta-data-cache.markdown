---
published: true
title: How to tune meta data cache?
layout: post
tags: [sybase, meta, datacache]
categories: [database]
---
*Meta data cache is data about data. Every time you access a table information about the database, the table, the index, the partition must be accessed. Therefore it's very important that the parameters managing those objects are not out-tuned. This notes explains how to check the values*

<!--excerpt-->

Again make sure meta data cache objects aren't scavenged and the parameters are correctly set to prevent any **reuse**.

How can we check the parameters are set correctly?
* look section `Metadata Cache Management` of sp_sysmon. Make user `Reuse Request` are empty as `Object Manager Spinlock Contention`,
* look at the output of `sp_monitorconfig 'all'` and make sure `Reuse_cnt` is empty.

### Object Manager Spinlock Contention

Each medata cache object (DES / IDES / PDES) has some values that can be higly to low volatile. Every time you modify a value you need to lock it through a spinlock and in multi-concurrent envs, it can lead to contention.

Moreover every time you access a table, you need to access the object DES, index DES and partition DES so the access chain is DES->IDES->PDES

Whenever a descriptor is active it stays in a KEPT list and when not used it stays in a SCAVENGE list. Hot objects tend to move from one list to the other leading to some contention. The idea here is to reduce this contention by keeping hot objects in KEP list.

So do you identify `hot object`? with `sp_object_stats 'HH:MM:SS', top N, db_name`

The you bind the table to the KEPT list with `dbcc tune (des_bind,<dbid>,<objname>)`.

To unbind an object, use `dbcc tune (des_unbind,<dbid>,<objname>)`.

To find the bind state, use `dbcc tune (hot_des,<dbid>)`

**dbcc tune commands are not persisted after a reboot**

Below are some recommendations

![DES recommendations 1]({{ site.url }}/assets/pictures/sybase_des_recommendations_1.png)

![DES recommendations 2]({{ site.url }}/assets/pictures/sybase_des_recommendations_2.png)