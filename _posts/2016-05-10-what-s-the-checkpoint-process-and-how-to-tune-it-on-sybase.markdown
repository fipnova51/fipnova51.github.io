---
published: true
title: What's the checkpoint process and how to tune it on Sybase
layout: post
---
*This note explains what the checkpoint process is and how we can analyze and tune its behaviour*

<!--excerpt-->

The checkpoint process is in charge of flushing on disk  the dirty pages in cache to reduce their number and the recovery time in case ASE is restarted.

~~~
recovery interval in minutes sets the maximum number of minutes per database that Adaptive Server uses to complete its recovery procedures in case of a system failure. The recovery procedure rolls transactions backward or forward, starting from the transaction that the checkpoint process indicates as the oldest active transaction. The recovery process has more or less work to do, depending on the value of recovery interval in minutes.
~~~

So we know the checkpoint purpose, now how it works?

1. checkpoint process is awake every 60 seconds,
1. checkpoint process knows that it can basically recover 6000 rows per minute, so depending on parameter `recovery interval in minutes`, if number of rows > 6000 * `recovery interval in minutes` it starts flushing pages,
1. checkpoint flush pages in `i/o batch size` groups of pages. Once a batch is issued it goes to sleep and wait until the flush is done and execute the next batch of writes,
1. if there are no dirty pages (thanks to the housekeeper) the checkpoint is considered as `free checkpint`,
1. when finished, it writes a checkpoint record in the log as well as writes a list of pages flushed to disk using Page Flush Time Stamp (PFTS) records

So now we know how it works, let's see the point to consider to find the right balance tuning:

1. we can reduce its work by increasing `recovery time in minutes` but on the other hand it leaves more dirty pages in cache and can lead to cache stalls (process is waiting for a dirty page to be written to disk in order to get a free page in the cache)
1. increase `i/o batch size` but it will stress the I/O subsystem and potentially generate some waits on the MASS bit
1. Increase `number of checkpoint processes` if there are a lot of databases in the system but as several checkpoint could be executed in parallel make sure to not exceed the number of `disk I/O structures`

OK. So now how can know if the checkpoint is behaving correctly?

To do so, we need to look at sp_sysmon output.

1. get the sample interval (x)
2. get the number of DB on the dataserver (y)
3. knowing that checkpoint is triggered every minute, during the sample we can have a maximum of **x time y** checkpoints. This must be compared with the `# of normal checkpoints` in sp_sysmon output and make sure we're below
4. then look at `Avg time per normal chkpt` and mulitply it by `y` and make sure the value is less than a minute.