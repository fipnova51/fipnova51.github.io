---
published: true
title: How disk I/O structures are managed in Sybase
layout: post
tags: [sybase, sp_sysmon, disk_io_structure]
categories: [database]
---
*Disk I/O structures controls the number of IO requests done by ASE, if this value is too low ASE will wait for an available token to be freed before issuing its I/O request, this note explains how to monitor it*

<!--excerpt-->

To avoid contention on allocating `disk I/O structures` during runtime in SMP environments, half of the allocated `I/O structures` are distributed evenly at start-up between engines to create a `per-engine` local pool of `disk I/O structures`.

The other half are retianed in the `disk I/O structure` global pool

`local-pool disk I/O structure size = ('number of I/O structures'/2) / 'number of engines' `

This being said, how can you check the configured value is correct?

You can use `Disk I/O Management` section of `sp_sysmon` and check that the max value for a specific engine is lower than the value computed previously

for example

~~~
Disk I/O Management
-------------------
 
  Max Outstanding I/Os            per sec      per xact       count  % of total
  -------------------------  ------------  ------------  ----------  ---------- 
    Server                            n/a           n/a         721       n/a   
    Engine 0                          n/a           n/a         517       n/a   
    Engine 1                          n/a           n/a         408       n/a   
    Engine 2                          n/a           n/a         410       n/a   
    Engine 3                          n/a           n/a         505       n/a   
    Engine 4                          n/a           n/a         427       n/a   
    Engine 5                          n/a           n/a         441       n/a   
~~~
<br/>
Here, the max value is `505` for a dataserver with 5 engines, therefore the configured value should be at least
`505 * 5 * 2 = 5050`
