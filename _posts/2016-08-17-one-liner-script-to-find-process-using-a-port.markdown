---
published: true
title: One-liner script to find process using a port
layout: post
tags: [solaris, network]
categories: [system]
---
*How to find a process listnening on a port when lsof is not working?*

<!--excerpt-->

<pre style='color:#000000;background:#ffffff;'>ps <span style='color:#44aadd; '>-ef</span> <span style='color:#e34adc; '>|</span> grep <span style='color:#797997; '>${</span>USER<span style='color:#808030; '>:</span><span style='color:#008c00; '>0</span><span style='color:#808030; '>:</span><span style='color:#008c00; '>8</span><span style='color:#797997; '>}</span> <span style='color:#e34adc; '>|</span> grep <span style='color:#44aadd; '>-v</span> grep <span style='color:#e34adc; '>|</span> awk <span style='color:#0000e6; '>'{print $2}'</span> <span style='color:#e34adc; '>|</span> xargs -I <span style='color:#0000e6; '>'{}'</span> sh <span style='color:#44aadd; '>-c</span> <span style='color:#0000e6; '>'echo examining process {}; pfiles {} | grep &lt;PORT NUMBER>'</span>
</pre>