---
published: true
title: Understanding an tuning the transaction logs
layout: post
tags: [sybase, transaction, logs, syslogs]
categories: [database]
---
*The transaction logs is a vital component and source of several bottlenecks. Understanding how it works can help you tune it for better performance*

<!--excerpt-->

### Who's using the transaction logs

The transaction logs is used by:

* The checkpoint process to know which pages to flush on disk
* Post commit processing, it's scanned to know what data to change during a deferred inserts, updates or deletes
* Sybase replication
* Dump transaction
* Rollback of large transactions if the data to be reverted are not in the ULC
* Recovery from shutdown/crash

### transaction space allocation

Transaction logs is represented by `syslogs` table hosted on `segmap` of value 4. It's used in a circular way meaning that if your transactions logs is split on several segmap, once the current segmap is used, it'll continue to the next one and like this till then end of the last segmap before continuing to he beginning of the first segmap.

### Log writes, the ULC and Log Cache

Below is a description of the whole chain when modifying data

1.  `SPID` grabs a spinlock on the spinlock protecting its `ULC` (`sp_configure 'user log cache spinlock ratio`)
1. Modifed rows are written in the `ULC`
1. When the `ULC` is full, a commit is reach or any reason, the `SPID` tries to get a lock on the `last log page` via the `log semaphone`last
1. The log semaphore is granted, then the SPID needs to get spinlock on the cache the log is using
1. Once both objects are taken, ULC is flushed to the last log page in the cache (with all flushing mechanism as a normal cache if no free pages)
1. Once all pages are in cache, the process flushes the cache by issuing physical writes on each log page in sequence. The `last log page` may not be written to disk if not full tue to the `group commit sleep` implementation. Once the log pages are modified in the cache, the spinlock is released
1. Once all pages are written to disk including the `last log page`, the transaction is considered committed, the `SPID` releases the `log semaphore` and continue with the next statement

*Keep in mind that what is modified and written to disk are `log` information`. The actual data/index information modified are still in the cache and will be written to disk by the `checkpoint` or `housekeeper` process*

![transaction logs process]({{ site.url }}/assets/pictures/sybase_transaction_logs_process.png)

#### Remarks about the log semaphore.

Again, the flush to syslogs is done by getting the `log semaphore` on the `last log page`. If the `SPID` can't get it, it goes to sleep. The reason it can't get it is because another SPID has it and will write some log information in `syslogs` leading to a change of the `last log page`. The sleeping `SPID` is awake when the `log semaphore` is available.

The example below shows the update of the last log page:

~~~
use demo_db
go
dbcc traceon(3604)
go
dbcc dbtable('demo_db')
go
.
..
...
dbt_logsema=0000000003BAEE40
dbt_nextseq=12   dbt_oldseq=12

-- after deleting some rows

dbcc dbtable('demo_db')
go
.
..
...
dbt_logsema=0000000003BCEE40
dbt_nextseq=5653   dbt_oldseq=5638
~~~

Here are some comments about the WaitEvent MDA tables

![wait event for log semaphore]({{ site.url }}/assets/pictures/sybase_waitevent_logsemaphore.png)

`WaitEventID 54` is clearly the event showing the SPID couldn't get the `log semaphore` because another is having it.

### When does the write from log cache to syslogs happen?

as we saw the picture above, the ULC is flushed in the `log cache` then written to disk but when does the flush really take place?
To make it simple, the flush is done:

* when the number of log pages equal to `log io size` is reached. Any SPID that filled the log cache are put to sleep until the write is initiated

![commit sleep scenario 1]({{ site.url}}/assets/pictures/sybase_group_commit_sleep_scenario1.png)

* the number of log pages in the cache does not reach the `log io size` but the first `SPID` put to sleep reach the top of the run queue and no other process forced the log page to be written to disk

![commit sleep scenario 2]({{ site.url}}/assets/pictures/sybase_group_commit_sleep_scenario2.png)