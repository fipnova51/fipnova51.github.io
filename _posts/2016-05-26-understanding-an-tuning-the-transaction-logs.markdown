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

as we saw the picture above, the `ULC` is flushed in the `log cache` then written to disk but when does the flush really take place?
To make it simple, the flush is done:

* when the number of log pages equal to `log io size` is reached. Any `SPID` that filled the log cache are put to sleep until the write is initiated

![commit sleep scenario 1]({{ site.url}}/assets/pictures/sybase_group_commit_sleep_scenario1.png)

* the number of log pages in the cache does not reach the `log io size` but the first `SPID` put to sleep reach the top of the run queue and no other process forced the log page to be written to disk

![commit sleep scenario 2]({{ site.url}}/assets/pictures/sybase_group_commit_sleep_scenario2.png)

### A few words about log semaphore contention

Log records are writthen in serial order. This is enforced by the usage of a `log semaphore` and can be a bottleneck in the performance of the  dataserve but ** the log semaphoe can appear as the bottleneck but it's important to find the root cause leading to this **.

First of all, how could we measure contention on the log semaphore?

One day to do it is to regularly compute ratio AppendLogWaits/AppendLogRequests metrics taken from monOpenDatabases and plot the output.

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> DBID<span style='color:#808030; '>,</span> DBName<span style='color:#808030; '>,</span> AppendLogRequests<span style='color:#808030; '>,</span> AppendLogWaits<span style='color:#808030; '>,</span> <span style='color:#808030; '>(</span><span style='color:#bb7977; font-weight:bold; '>convert</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>numeric</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>10</span><span style='color:#808030; '>,</span><span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span>AppendLogWaits<span style='color:#808030; '>)</span><span style='color:#808030; '>/</span><span style='color:#bb7977; font-weight:bold; '>convert</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>numeric</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>10</span><span style='color:#808030; '>,</span><span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span>AppendLogRequests<span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>*</span><span style='color:#008c00; '>100</span> <span style='color:#800000; font-weight:bold; '>as</span> <span style='color:#0000e6; '>'LogContention%'</span>  
<span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monOpenDatabases
<span style='color:#800000; font-weight:bold; '>go</span>
</pre>
<br/>

Let's look at some possible root-cause of the `log semaphore contention`

#### Transaction log phusical devices

Ideally the log device is written to but never read, therefore it should be one of the fastest device on the system. How can we check the number of Reads/Writes and their timing? in table monDeviceIO

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> LogicalName<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>Reads</span><span style='color:#808030; '>,</span> APFReads<span style='color:#808030; '>,</span> ReadTime<span style='color:#808030; '>,</span> <span style='color:#bb7977; font-weight:bold; '>convert</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>numeric</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>10</span><span style='color:#808030; '>,</span><span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span>ReadTime<span style='color:#808030; '>)</span><span style='color:#808030; '>/</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>Reads</span> <span style='color:#808030; '>+</span> APFReads<span style='color:#808030; '>)</span> <span style='color:#800000; '>"Read_ms"</span><span style='color:#808030; '>,</span> 
       Writes<span style='color:#808030; '>,</span> WriteTime<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>case</span> Writes <span style='color:#800000; font-weight:bold; '>when</span> <span style='color:#008c00; '>0</span> <span style='color:#800000; font-weight:bold; '>then</span> <span style='color:#008c00; '>0</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#bb7977; font-weight:bold; '>convert</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>numeric</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>10</span><span style='color:#808030; '>,</span><span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span>WriteTime<span style='color:#808030; '>)</span><span style='color:#808030; '>/</span>Writes <span style='color:#800000; font-weight:bold; '>end</span> <span style='color:#800000; '>"Writes_ms"</span><span style='color:#808030; '>,</span> DevSemaphoreRequests<span style='color:#808030; '>,</span> DevSemaphoreWaits  
         <span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monDeviceIO monIO <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>sysdevices dev <span style='color:#800000; font-weight:bold; '>on</span> monIO<span style='color:#808030; '>.</span>LogicalName <span style='color:#808030; '>=</span> dev<span style='color:#808030; '>.</span>name 
         <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#800000; font-weight:bold; '>distinct</span> u<span style='color:#808030; '>.</span>vdevno <span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>sysusages u <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>sysdatabases d <span style='color:#800000; font-weight:bold; '>on</span> d<span style='color:#808030; '>.</span>dbid <span style='color:#808030; '>=</span> u<span style='color:#808030; '>.</span>dbid <span style='color:#800000; font-weight:bold; '>where</span> d<span style='color:#808030; '>.</span>name <span style='color:#808030; '>=</span> <span style='color:#0000e6; '>'BBVA_CONVERSION_DEBUG'</span><span style='color:#808030; '>)</span> dev_used  <span style='color:#800000; font-weight:bold; '>on</span> dev<span style='color:#808030; '>.</span>vdevno <span style='color:#808030; '>=</span> dev_used<span style='color:#808030; '>.</span>vdevno<span style='color:#808030; '>;</span> <span style='color:#696969; '>--IO speed information</span>
</pre>
<br/>

Another to check whether or not the contention is there is to look at monSysWaits and monProcessWaits, keep in mind that
* WaitEventID 54 is the wait for the log semaphore so the contention
* WaitEventID 55 is the wait for the write to syslogs to finish

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#800000; font-weight:bold; '>top</span> <span style='color:#008c00; '>20</span> T2<span style='color:#808030; '>.</span>Description Event_Desc<span style='color:#808030; '>,</span> T3<span style='color:#808030; '>.</span>Description Class_Desc<span style='color:#808030; '>,</span> T1<span style='color:#808030; '>.</span>WaitEventID<span style='color:#808030; '>,</span> T1<span style='color:#808030; '>.</span>WaitTime<span style='color:#808030; '>,</span> T1<span style='color:#808030; '>.</span>Waits<span style='color:#808030; '>,</span> <span style='color:#bb7977; font-weight:bold; '>convert</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>numeric</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>10</span><span style='color:#808030; '>,</span><span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span>WaitTime<span style='color:#808030; '>)</span><span style='color:#808030; '>/</span>Waits <span style='color:#800000; '>"ms/Wait"</span>
<span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monSysWaits T1 <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monWaitEventInfo T2 <span style='color:#800000; font-weight:bold; '>on</span> T1<span style='color:#808030; '>.</span>WaitEventID <span style='color:#808030; '>=</span> T2<span style='color:#808030; '>.</span>WaitEventID 
                            <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monWaitClassInfo T3 <span style='color:#800000; font-weight:bold; '>on</span> T2<span style='color:#808030; '>.</span>WaitClassID <span style='color:#808030; '>=</span> T3<span style='color:#808030; '>.</span>WaitClassID 
<span style='color:#800000; font-weight:bold; '>order</span> <span style='color:#800000; font-weight:bold; '>by</span> WaitTime <span style='color:#800000; font-weight:bold; '>desc</span><span style='color:#808030; '>;</span> <span style='color:#696969; '>-- get System wide Waits events</span>
<span style='color:#800000; font-weight:bold; '>select</span> T1<span style='color:#808030; '>.</span>SPID<span style='color:#808030; '>,</span> T1<span style='color:#808030; '>.</span>InstanceID <span style='color:#808030; '>,</span>T1<span style='color:#808030; '>.</span>KPID<span style='color:#808030; '>,</span> T2<span style='color:#808030; '>.</span>Waits<span style='color:#808030; '>,</span> T2<span style='color:#808030; '>.</span>WaitTime<span style='color:#808030; '>,</span> <span style='color:#bb7977; font-weight:bold; '>convert</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>numeric</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>10</span><span style='color:#808030; '>,</span><span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span>T2<span style='color:#808030; '>.</span>WaitTime<span style='color:#808030; '>)</span><span style='color:#808030; '>/</span>T2<span style='color:#808030; '>.</span>Waits <span style='color:#800000; '>"ms/Wait"</span><span style='color:#808030; '>,</span> T3<span style='color:#808030; '>.</span>Description Event_Desc<span style='color:#808030; '>,</span> T4<span style='color:#808030; '>.</span>Description Class_Desc
<span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monProcess T1 <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monProcessWaits T2 <span style='color:#800000; font-weight:bold; '>on</span> T1<span style='color:#808030; '>.</span>SPID <span style='color:#808030; '>=</span> T2<span style='color:#808030; '>.</span>SPID <span style='color:#800000; font-weight:bold; '>and</span> T1<span style='color:#808030; '>.</span>InstanceID <span style='color:#808030; '>=</span> T2<span style='color:#808030; '>.</span>InstanceID <span style='color:#800000; font-weight:bold; '>and</span> T1<span style='color:#808030; '>.</span>KPID <span style='color:#808030; '>=</span> T2<span style='color:#808030; '>.</span>KPID 
                           <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monWaitEventInfo T3 <span style='color:#800000; font-weight:bold; '>on</span> T2<span style='color:#808030; '>.</span>WaitEventID <span style='color:#808030; '>=</span> T3<span style='color:#808030; '>.</span>WaitEventID
                           <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monWaitClassInfo T4 <span style='color:#800000; font-weight:bold; '>on</span> T3<span style='color:#808030; '>.</span>WaitClassID <span style='color:#808030; '>=</span> T4<span style='color:#808030; '>.</span>WaitClassID
<span style='color:#800000; font-weight:bold; '>where</span> T1<span style='color:#808030; '>.</span>SPID <span style='color:#800000; font-weight:bold; '>in</span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>select</span> spid <span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>sysprocesses <span style='color:#800000; font-weight:bold; '>where</span> dbid <span style='color:#800000; font-weight:bold; '>in</span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>select</span> dbid <span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>sysdatabases <span style='color:#800000; font-weight:bold; '>where</span> name <span style='color:#808030; '>=</span> <span style='color:#0000e6; '>'ENV0000121_20864313'</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>;</span> <span style='color:#696969; '>-- get process Waits events</span>
</pre>
<br/>

### User Log cache and Log IO Size

First of all, few facts about the `ULC` and `log io size`.

* ULC size should be at least the same size of `log io size`
* Default `log io size` is 2 time the `page size` but  a buffer pool must be available otherwise the log will use the `page size` pool,
* unless the transaction logs is binded to a dedicated cache, `default data cache` will be used

#### sp_logiosize

Defining the `log io size` is difficult. One way to measure it is to check the value for `Avg # Writes per Log Page` in `sp_sysmon` and trying to have the lowest value.