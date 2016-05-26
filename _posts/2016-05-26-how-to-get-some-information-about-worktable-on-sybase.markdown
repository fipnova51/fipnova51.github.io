---
published: true
title: How to get some information about worktable on Sybase
layout: post
tags: [sybase, worktable, set, option]
categories: [database]
---
*It's sometime interesting to get some information about worktables involved in SQL statement. Below are the command to set before executing your query*

<!--excerpt-->

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>dbcc</span> traceon<span style='color:#808030; '>(</span><span style='color:#008c00; '>3604</span><span style='color:#808030; '>)</span>
<span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#800000; font-weight:bold; '>set</span> statement_cache <span style='color:#800000; font-weight:bold; '>off</span>
<span style='color:#800000; font-weight:bold; '>set</span> showplan <span style='color:#800000; font-weight:bold; '>on</span>
<span style='color:#800000; font-weight:bold; '>set</span> <span style='color:#800000; font-weight:bold; '>option</span> show_code_gen <span style='color:#800000; font-weight:bold; '>long</span>
<span style='color:#800000; font-weight:bold; '>set</span> <span style='color:#800000; font-weight:bold; '>option</span> show_best_plan
<span style='color:#800000; font-weight:bold; '>go</span>
<span style='color:#808030; '>&lt;</span>YOUR <span style='color:#800000; font-weight:bold; '>SQL</span><span style='color:#808030; '>></span>
<span style='color:#800000; font-weight:bold; '>go</span>
</pre>
<br/>