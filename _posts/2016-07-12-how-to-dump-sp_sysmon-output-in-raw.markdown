---
published: true
title: How to dump sp_sysmon output in raw?
layout: post
tags: [sybase]
---
*sp_sysmon output if difficult to parse because it generates a 'human readable' format and its content can change from version to version. There's an option to dump the output in raw format that could help processing the information* 


<!--excerpt-->

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>exec</span> sp_sysmon <span style='color:#0000e6; '>'00:30:00'</span><span style='color:#808030; '>,</span><span style='color:#797997; '>@dumpcounters</span><span style='color:#808030; '>=</span><span style='color:#0000e6; '>'y'</span>
</pre>