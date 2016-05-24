---
published: true
title: How to tune the housekeeper wasg process
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

## Housekeeper Wash

First of all, the `housekeeper wash` doesn't process `logonly` or `relaxed` caches.

To reduce the checkpoint process duration and increase the number of free checkpoints, one possibility is to increase the number of checkpoint processes or the recovery interval.

The other alternative is to tune the `housekeeper wash`. The `housekeeper wash` is set with parameter `housekeeper free write percent`. By default the value is set to 1 which is quite low. You can increase the value but it might add some contention on the MASS bits and it's hard to find out whether the contention is due to the checkpoint or housekeeper process, one possible way is to monitor waitEvent 31 and 52 for both processes

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> T2<span style='color:#808030; '>.</span>Command<span style='color:#808030; '>,</span>T2<span style='color:#808030; '>.</span>Priority<span style='color:#808030; '>,</span>T2<span style='color:#808030; '>.</span>SPID<span style='color:#808030; '>,</span> T3<span style='color:#808030; '>.</span>WaitEventID<span style='color:#808030; '>,</span>T3<span style='color:#808030; '>.</span>Description<span style='color:#808030; '>,</span>T1<span style='color:#808030; '>.</span>Waits<span style='color:#808030; '>,</span> T1<span style='color:#808030; '>.</span>WaitTime<span style='color:#808030; '>,</span>T4<span style='color:#808030; '>.</span>PhysicalWrites 
<span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monProcessWaits T1 <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monProcess T2 <span style='color:#800000; font-weight:bold; '>on</span> T1<span style='color:#808030; '>.</span>SPID<span style='color:#808030; '>=</span>T2<span style='color:#808030; '>.</span>SPID <span style='color:#800000; font-weight:bold; '>and</span> T1<span style='color:#808030; '>.</span>InstanceID <span style='color:#808030; '>=</span> T2<span style='color:#808030; '>.</span>InstanceID <span style='color:#800000; font-weight:bold; '>and</span> T1<span style='color:#808030; '>.</span>KPID <span style='color:#808030; '>=</span> T2<span style='color:#808030; '>.</span>KPID
                                <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monWaitEventInfo T3 <span style='color:#800000; font-weight:bold; '>on</span> T1<span style='color:#808030; '>.</span>WaitEventID <span style='color:#808030; '>=</span> T3<span style='color:#808030; '>.</span>WaitEventID 
                                <span style='color:#800000; font-weight:bold; '>join</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monProcessActivity T4 <span style='color:#800000; font-weight:bold; '>on</span> T1<span style='color:#808030; '>.</span>SPID <span style='color:#808030; '>=</span> T4<span style='color:#808030; '>.</span>SPID <span style='color:#800000; font-weight:bold; '>and</span> T1<span style='color:#808030; '>.</span>InstanceID <span style='color:#808030; '>=</span> T4<span style='color:#808030; '>.</span>InstanceID <span style='color:#800000; font-weight:bold; '>and</span> T1<span style='color:#808030; '>.</span>KPID <span style='color:#808030; '>=</span> T4<span style='color:#808030; '>.</span>KPID
<span style='color:#800000; font-weight:bold; '>where</span> T2<span style='color:#808030; '>.</span>Command <span style='color:#800000; font-weight:bold; '>in</span> <span style='color:#808030; '>(</span><span style='color:#0000e6; '>'CHECKPOINT SLEEP'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'HK WASH'</span><span style='color:#808030; '>)</span><span style='color:#808030; '>;</span>
</pre>
<br/>

Moreover the housekeeper has an internal limit of only submiting 3 I/O to any devices. If the number of I/Os to submit is higher then it'll cancel the request, therefore it's recommended to increase this value with `dbcc tune(deviochar,<vdevno>,"<number of IO>")`. Setting vdevno to -1 will do the changes for all devices, you can increase the value to 20 even more. This has to be checked. Last, this setting is not **persistent** meaning you'll have to set it again if the server reboots

Last point, it's possible to disable the HKW on a specific cache by adding the following in Sybase configuration file

~~~
cache status = HK ignore cache
~~~
Keep in mind that this setup must not be set for `logonly`, `relaxed` caches type nor on the `default data cache`.
I good test would be to create a dedicated cache to the tempdb and set this parameter on it.