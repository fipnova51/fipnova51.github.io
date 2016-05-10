---
published: true
title: How do I create a dedicated cache for the transaction logs in Sybase?
layout: post
tags: [sybase, cache, transaction_log]
categories: [database]
---
*By default the transaction logs is put in the default data cache, it can be interesting for performance purpose to have a dedicated cache for it, this note explains how to setup a dedicated cache for the transaction log*

<!--excerpt-->

## Create a new cache of type `logonly`

<pre style='color:#000000;background:#ffffff;'>sp_cacheconfig <span style='color:#0000e6; '>'log cache'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'14000M'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'logonly'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'cache partition=4'</span>
</pre>

By default ASE will flush ULC in chunks of 2* ase page size so you have to create a dedicated pool

<pre style='color:#000000;background:#ffffff;'>sp_poolconfig <span style='color:#0000e6; '>'log cache'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'12000M'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'16K'</span>
</pre>

let's review the cache available

<pre style='color:#000000;background:#ffffff;'><span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> sp_cacheconfig
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#800000; font-weight:bold; '>Cache</span> Name Status <span style='color:#800000; font-weight:bold; '>Type</span> Config <span style='color:#800000; font-weight:bold; '>Value</span> <span style='color:#800000; font-weight:bold; '>Run</span> <span style='color:#800000; font-weight:bold; '>Value</span>
<span style='color:#696969; '>------------------ -------- -------- -------------- ------------</span>
<span style='color:#800000; font-weight:bold; '>default</span> <span style='color:#800000; font-weight:bold; '>data</span> <span style='color:#800000; font-weight:bold; '>cache</span> Active <span style='color:#800000; font-weight:bold; '>Default</span> <span style='color:#008000; '>74000.00</span> Mb <span style='color:#008000; '>92160.00</span> Mb
<span style='color:#bb7977; font-weight:bold; '>log</span> <span style='color:#800000; font-weight:bold; '>cache</span> Active <span style='color:#bb7977; font-weight:bold; '>Log</span> <span style='color:#800000; font-weight:bold; '>Only</span> <span style='color:#008000; '>14000.00</span> Mb <span style='color:#008000; '>14000.00</span> Mb
<span style='color:#696969; '>------------ ------------</span>
Total <span style='color:#008000; '>88000.0</span> Mb <span style='color:#008c00; '>106160</span> Mb
<span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span><span style='color:#808030; '>=</span>
<span style='color:#e34adc; '>Cache:</span> <span style='color:#bb7977; font-weight:bold; '>log</span> <span style='color:#800000; font-weight:bold; '>cache</span><span style='color:#808030; '>,</span> <span style='color:#e34adc; '>Status:</span> Active<span style='color:#808030; '>,</span> <span style='color:#e34adc; '>Type:</span> <span style='color:#bb7977; font-weight:bold; '>Log</span> <span style='color:#800000; font-weight:bold; '>Only</span>
Config <span style='color:#800000; font-weight:bold; '>Size</span><span style='color:#808030; '>:</span> <span style='color:#797997; '>14000</span><span style='color:#808030; '>.</span><span style='color:#797997; '>00</span> Mb<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>Run</span> <span style='color:#800000; font-weight:bold; '>Size</span><span style='color:#808030; '>:</span> <span style='color:#797997; '>14000</span><span style='color:#808030; '>.</span><span style='color:#797997; '>00</span> Mb
Config <span style='color:#e34adc; '>Replacement:</span> strict LRU<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>Run</span> <span style='color:#e34adc; '>Replacement:</span> strict LRU
Config <span style='color:#e34adc; '>Partition:</span> <span style='color:#008c00; '>4</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>Run</span> <span style='color:#e34adc; '>Partition:</span> <span style='color:#008c00; '>4</span>
IO <span style='color:#800000; font-weight:bold; '>Size</span> Wash <span style='color:#800000; font-weight:bold; '>Size</span> Config <span style='color:#800000; font-weight:bold; '>Size</span> <span style='color:#800000; font-weight:bold; '>Run</span> <span style='color:#800000; font-weight:bold; '>Size</span> APF <span style='color:#800000; font-weight:bold; '>Percent</span>
<span style='color:#696969; '>-------- ------------- ------------ ------------ -----------</span>
<span style='color:#008c00; '>4</span> Kb <span style='color:#008c00; '>245760</span> Kb <span style='color:#008000; '>0.00</span> Mb <span style='color:#008000; '>2000.00</span> Mb <span style='color:#008c00; '>10</span>
<span style='color:#008c00; '>16</span> Kb <span style='color:#008c00; '>245760</span> Kb <span style='color:#008000; '>12000.00</span> Mb <span style='color:#008000; '>12000.00</span> Mb <span style='color:#008c00; '>10</span>
</pre>
<br/>

## Bind your database syslogs to this dedicated cache

<pre style='color:#000000;background:#ffffff;'><span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> sp_dboption TPK0002287_17285240<span style='color:#808030; '>,</span><span style='color:#0000e6; '>'single'</span><span style='color:#808030; '>,</span><span style='color:#800000; font-weight:bold; '>true</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#800000; font-weight:bold; '>Database</span> <span style='color:#800000; font-weight:bold; '>option</span> <span style='color:#0000e6; '>'single user'</span> turned <span style='color:#800000; font-weight:bold; '>ON</span> <span style='color:#800000; font-weight:bold; '>for</span> <span style='color:#800000; font-weight:bold; '>database</span> <span style='color:#0000e6; '>'TPK0002287_17285240'</span><span style='color:#808030; '>.</span>
Running <span style='color:#800000; font-weight:bold; '>CHECKPOINT</span> <span style='color:#800000; font-weight:bold; '>on</span> <span style='color:#800000; font-weight:bold; '>database</span> <span style='color:#0000e6; '>'TPK0002287_17285240'</span> <span style='color:#800000; font-weight:bold; '>for</span> <span style='color:#800000; font-weight:bold; '>option</span> <span style='color:#0000e6; '>'single user'</span> <span style='color:#800000; font-weight:bold; '>to</span> take effect<span style='color:#808030; '>.</span>
<span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>return</span> status <span style='color:#808030; '>=</span> <span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>checkpoint</span> TPK0002287_17285240
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>use</span> TPK0002287_17285240
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> sp_bindcache <span style='color:#0000e6; '>'log cache'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'TPK0002287_17285240'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'syslogs'</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
</pre>
<br/>

## Define the logiosize for your database

You must be in your database before executing command `sp_logiosize`

<pre style='color:#000000;background:#ffffff;'><span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> sp_logiosize <span style='color:#0000e6; '>'16'</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#bb7977; font-weight:bold; '>Log</span> <span style='color:#800000; font-weight:bold; '>I</span><span style='color:#808030; '>/</span>O <span style='color:#800000; font-weight:bold; '>size</span> <span style='color:#800000; font-weight:bold; '>is</span> <span style='color:#800000; font-weight:bold; '>set</span> <span style='color:#800000; font-weight:bold; '>to</span> <span style='color:#008c00; '>16</span> Kbytes<span style='color:#808030; '>.</span>
<span style='color:#800000; font-weight:bold; '>The</span> <span style='color:#800000; font-weight:bold; '>transaction</span> <span style='color:#bb7977; font-weight:bold; '>log</span> <span style='color:#800000; font-weight:bold; '>for</span> <span style='color:#800000; font-weight:bold; '>database</span> <span style='color:#0000e6; '>'TPK0002287_17285240'</span> will <span style='color:#800000; font-weight:bold; '>use</span> <span style='color:#800000; font-weight:bold; '>I</span><span style='color:#808030; '>/</span>O <span style='color:#800000; font-weight:bold; '>size</span> <span style='color:#800000; font-weight:bold; '>of</span> <span style='color:#008c00; '>16</span> Kbytes<span style='color:#808030; '>.</span>
<span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>return</span> status <span style='color:#808030; '>=</span> <span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span>
</pre>



