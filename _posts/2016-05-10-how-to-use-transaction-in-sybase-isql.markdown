---
published: true
title: How to use transaction in Sybase isql
layout: post
tags: [sybase, isql, transaction]
categories: [database]
---
*I've been forgeting how to use transaction in `isql`. Once for all I put a sample here*

<!--excerpt-->

<br/>

<pre style='color:#000000;background:#ffffff;'><span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#797997; '>@@trancount</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>

 <span style='color:#696969; '>-----------</span>
           <span style='color:#008c00; '>0</span>

<span style='color:#808030; '>(</span><span style='color:#008c00; '>1</span> <span style='color:#800000; font-weight:bold; '>row</span> affected<span style='color:#808030; '>)</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>begin</span> <span style='color:#800000; font-weight:bold; '>tran</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#bb7977; font-weight:bold; '>count</span><span style='color:#808030; '>(</span><span style='color:#808030; '>*</span><span style='color:#808030; '>)</span> <span style='color:#800000; font-weight:bold; '>from</span> sysobjects
<span style='color:#008c00; '>3</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>

 <span style='color:#696969; '>-----------</span>
       <span style='color:#008c00; '>15000</span>

<span style='color:#808030; '>(</span><span style='color:#008c00; '>1</span> <span style='color:#800000; font-weight:bold; '>row</span> affected<span style='color:#808030; '>)</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#797997; '>@@trancount</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>

 <span style='color:#696969; '>-----------</span>
           <span style='color:#008c00; '>1</span>

<span style='color:#808030; '>(</span><span style='color:#008c00; '>1</span> <span style='color:#800000; font-weight:bold; '>row</span> affected<span style='color:#808030; '>)</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>commit</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#008c00; '>1</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#797997; '>@@trancount</span>
<span style='color:#008c00; '>2</span><span style='color:#808030; '>></span> <span style='color:#800000; font-weight:bold; '>go</span>

 <span style='color:#696969; '>-----------</span>
           <span style='color:#008c00; '>0</span>
</pre>