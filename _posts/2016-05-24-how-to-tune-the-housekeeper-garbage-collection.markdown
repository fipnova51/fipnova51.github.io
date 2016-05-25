---
published: true
title: How to tune the housekeeper garbage collection?
layout: post
tags: [sybase, housekeeper]
categories: [database]
---
*This note explains the housekeeper Garbage Collection and how to tune it*

<!--excerpt-->

First of all, there are three types of housekeeper tasks

* `Housekeeper wash` that will flush dirty pages to disks as I/O limits allow. This task only runs at idle and is controlled by the `housekeeper free write percent`
* `Housekeeper garbage collection` that will remove deallocated pages
* `Housekeeper chores` that will flush statistics information at idle times

## Housekeeper Garbage Collection

The `HK GC` task is responsible for reclaiming space from deleted rows and deallocating empty pages from DOL tables - both datarows and datapage locking.

One interesting point, when a change is made on *APL locked table*, the command will automatically reclaims the space on the page by compacting data or completely reclaiming the space is the page is now empty. Therefore as you can imagine the operation can be long as you *delete* and *reclaim* space at the same time.

On DOL table, it's another story. Basically when a row is deleted, it's marked *for deleteion* and the space reclamation is done by `HK GC` asynchronously.

`HK GC` behaviour is controlled by parameter `enable housekeeper GC` and SYBASE recommends a value of 4 or 5. 

Then the `HK GC` maintains a queueof pending request such as deletes, if HK GC is not running enough this queue might get an overflow and the lost requests will be lost. Moreover this queue is clean after reboot. This is why it's important to **monitor those queues** 

Example of SQL:

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> EngineNumber<span style='color:#808030; '>,</span> HkgcMaxQSize<span style='color:#808030; '>,</span> HkgcPendingItems<span style='color:#808030; '>,</span> HkgcHWMItems<span style='color:#808030; '>,</span> HkgcOverflows <span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monEngine<span style='color:#808030; '>;</span>
<span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#800000; font-weight:bold; '>top</span> <span style='color:#008c00; '>25</span> DBName<span style='color:#808030; '>,</span> ObjectName<span style='color:#808030; '>,</span> IndexID<span style='color:#808030; '>,</span> LogicalReads<span style='color:#808030; '>,</span> PhysicalWrites<span style='color:#808030; '>,</span> PagesWritten<span style='color:#808030; '>,</span> RowsInserted<span style='color:#808030; '>,</span> RowsDeleted<span style='color:#808030; '>,</span> RowsUpdated  HkgcRequests<span style='color:#808030; '>,</span> HkgcPending<span style='color:#808030; '>,</span> HkgcOverflows 
<span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monOpenObjectActivity <span style='color:#800000; font-weight:bold; '>order</span> <span style='color:#800000; font-weight:bold; '>by</span> HkgcPending <span style='color:#800000; font-weight:bold; '>desc</span><span style='color:#808030; '>;</span>
</pre>
<br/>

**Below is a list of configuration to consider for tuning**

![checkpoint and housekeeper]({{ site.url }}/assets/pictures/checkpoint_housekeeper_tuning_starting_point.png)

