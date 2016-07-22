---
published: true
title: Quick script to monitor network lost packets
layout: post
tags: [solaris, network]
categories: [system]
---
*How can I quickly check that I'm not facing any netwok issue while contacting a host?*

<!--excerpt-->

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>while</span> <span style='color:#800080; '>(</span><span style='color:#44aadd; '>true</span><span style='color:#800080; '>)</span><span style='color:#800080; '>;</span><span style='color:#800000; font-weight:bold; '>do</span> ping <span style='color:#44aadd; '>-s</span> mx553zn <span style='color:#008c00; '>64</span> <span style='color:#008c00; '>10</span> <span style='color:#e34adc; '>|</span> nawk <span style='color:#44aadd; '>-v</span> <span style='color:#797997; '>ddate</span><span style='color:#808030; '>=</span><span style='color:#0000e6; '>"$(date)"</span> <span style='color:#0000e6; '>'$0~/loss/ {if ($(NF-2) != "0%") {print ddate,"-- percentage of packets lost is " $(NF-2)} }'</span><span style='color:#800080; '>;</span>sleep <span style='color:#008c00; '>30</span><span style='color:#800080; '>;</span><span style='color:#800000; font-weight:bold; '>done</span>
</pre>