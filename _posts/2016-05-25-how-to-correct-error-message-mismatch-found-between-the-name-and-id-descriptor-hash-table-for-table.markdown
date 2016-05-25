---
published: true
title: How to correct error message 'Mismatch found between the name and id descriptor hash table for table'?
layout: post
tags: [sybase, mismatch, cache, dbcc]
categories: [database]
---
*We often face the error message `Mismatch found between the name and id descriptor hash table for table xxx` when updating the table owner directly in `sysobjects`. Here's a way to solve it without restarting the dataserver.*

<!--excerpt-->

The command to use is 

~~~
dbcc cacheremove(db_name | db_id, object_name | object_id)
~~~

example:

<pre style='color:#000000;background:#ffffff;'><span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#bb7977; font-weight:bold; '>count</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span> <span style='color:#800000; font-weight:bold; '>from</span> claire
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
Msg <span style='color:#008c00; '>8211</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>Level</span> <span style='color:#008c00; '>26</span><span style='color:#808030; '>,</span> State <span style='color:#008c00; '>1</span>
Server <span style='color:#0000e6; '>'MX509ZN'</span><span style='color:#808030; '>,</span> Line <span style='color:#008c00; '>1</span>
Mismatch <span style='color:#800000; font-weight:bold; '>found</span> <span style='color:#800000; font-weight:bold; '>between</span> <span style='color:#800000; font-weight:bold; '>the</span> name <span style='color:#800000; font-weight:bold; '>and</span> <span style='color:#800000; font-weight:bold; '>id</span> <span style='color:#800000; font-weight:bold; '>descriptor</span> <span style='color:#800000; font-weight:bold; '>hash</span> <span style='color:#800000; font-weight:bold; '>table</span> <span style='color:#800000; font-weight:bold; '>for</span> <span style='color:#800000; font-weight:bold; '>table</span> <span style='color:#0000e6; '>'claire'</span><span style='color:#808030; '>,</span> objid <span style='color:#808030; '>=</span> <span style='color:#008c00; '>1879139275</span><span style='color:#808030; '>.</span> <span style='color:#800000; font-weight:bold; '>Descriptor</span> hashed <span style='color:#800000; font-weight:bold; '>by</span> name <span style='color:#808030; '>=</span> <span style='color:#008000; '>0x0</span> <span style='color:#800000; font-weight:bold; '>and</span> hashed <span style='color:#800000; font-weight:bold; '>by</span> <span style='color:#800000; font-weight:bold; '>id</span> <span style='color:#808030; '>=</span> <span style='color:#008000; '>0x8b864350</span><span style='color:#808030; '>.</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>dbcc</span> cacheremove<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'BBVA_CONFIG_DEBUG_MX'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'claire'</span><span style='color:#808030; '>)</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
Msg <span style='color:#008c00; '>8211</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>Level</span> <span style='color:#008c00; '>26</span><span style='color:#808030; '>,</span> State <span style='color:#008c00; '>1</span>
Server <span style='color:#0000e6; '>'MX509ZN'</span><span style='color:#808030; '>,</span> Line <span style='color:#008c00; '>1</span>
Mismatch <span style='color:#800000; font-weight:bold; '>found</span> <span style='color:#800000; font-weight:bold; '>between</span> <span style='color:#800000; font-weight:bold; '>the</span> name <span style='color:#800000; font-weight:bold; '>and</span> <span style='color:#800000; font-weight:bold; '>id</span> <span style='color:#800000; font-weight:bold; '>descriptor</span> <span style='color:#800000; font-weight:bold; '>hash</span> <span style='color:#800000; font-weight:bold; '>table</span> <span style='color:#800000; font-weight:bold; '>for</span> <span style='color:#800000; font-weight:bold; '>table</span> <span style='color:#0000e6; '>'claire'</span><span style='color:#808030; '>,</span> objid <span style='color:#808030; '>=</span> <span style='color:#008c00; '>1879139275</span><span style='color:#808030; '>.</span> <span style='color:#800000; font-weight:bold; '>Descriptor</span> hashed <span style='color:#800000; font-weight:bold; '>by</span> name <span style='color:#808030; '>=</span> <span style='color:#008000; '>0x0</span> <span style='color:#800000; font-weight:bold; '>and</span> hashed <span style='color:#800000; font-weight:bold; '>by</span> <span style='color:#800000; font-weight:bold; '>id</span> <span style='color:#808030; '>=</span> <span style='color:#008000; '>0x8b864350</span><span style='color:#808030; '>.</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>dbcc</span> cacheremove<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'BBVA_CONFIG_DEBUG_MX'</span><span style='color:#808030; '>,</span><span style='color:#008c00; '>1879139275</span><span style='color:#808030; '>)</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#bb7977; font-weight:bold; '>count</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span> <span style='color:#800000; font-weight:bold; '>from</span> claire
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>

 <span style='color:#696969; '>-----------</span>
           <span style='color:#008c00; '>6</span>
</pre>

