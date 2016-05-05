---
published: true
title: Introduction to Dtrace
layout: post
tags: [dtrace]
categories: [system]
---
*Dtrace is a tool developped by SUN to trace any function call, from the application to the system and is very usefull to debug an application. It's working on Solaris Mac OSx and FreeBSD*

## D usage

D can be called in a one-liner mode

<pre style='color:#000000;background:#ffffff;'>dtrace <span style='color:#44aadd; '>-n</span> program
</pre>

D code can also be saved in a file which is then called

<pre style='color:#000000;background:#ffffff;'>dtrace <span style='color:#44aadd; '>-s</span> file<span style='color:#800000; font-weight:bold; '>.</span>d
</pre>

Last, if your file start with `#!/usr/sbin/dtrace -s` , you can just call your file for execution

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>.</span><span style='color:#40015a; '>/file.d</span>
</pre>

<br>

## D structure