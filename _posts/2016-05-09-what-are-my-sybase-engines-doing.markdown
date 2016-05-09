---
published: true
title: What are my sybase engines doing?
layout: post
tags: [sybase, performance, monSysLoad, engine, MDA]
categories: [database]
---
*I need to have a quick overview of what my Sybase engines are doing, how can I do that?*

<!--excerpt-->

You can *sp_sysmon* and look for the adequate report section or you can query monSysLoad.

*monSysLoad* provides trended statistics on a per-engine basis with an avg in the last minutes, last 5 minutes and last 15 minutes

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> StatisticID<span style='color:#808030; '>,</span> Statistic<span style='color:#808030; '>,</span> InstanceID<span style='color:#808030; '>,</span> EngineNumber<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>Sample</span><span style='color:#808030; '>,</span> SteadyState<span style='color:#808030; '>,</span> Avg_1min<span style='color:#808030; '>,</span> Avg_5min<span style='color:#808030; '>,</span> Avg_15min<span style='color:#808030; '>,</span> Peak<span style='color:#808030; '>,</span> Max_1min<span style='color:#808030; '>,</span> Max_5min<span style='color:#808030; '>,</span> Max_15min <span style='color:#800000; font-weight:bold; '>from</span> <span style='color:#800000; font-weight:bold; '>master</span><span style='color:#808030; '>.</span><span style='color:#808030; '>.</span>monSysLoad<span style='color:#808030; '>;</span> <span style='color:#696969; '>-- check especially Statistic 'run queue length'</span>
</pre>