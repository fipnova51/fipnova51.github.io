---
published: true
title: What are the parameters to set to understand a show plan in Sybase?
layout: post
tags: [sybase, performance, showplan, optimizer, statistics]
categories: [database]
---
*In this post, we'll review the different parameters to set in a session to analyze the performance of a SQL statement*

<!--excerpt-->

### Disable statement cache

`set statement_cache off`

### Enable statistics for execution time, IO and plancost

**plancost** is important as it will display the estimated rows returned based on statistics vs the actual rows. Big differences mean bad statistics.

`set statistics time, io, plancost on`

### enable option tracing

~~~
set option show_lio_costing on
set option show_missing_stats on
set show_sqltext on
~~~

### this option is to be enable at a next stage of analysis

`set option show long`

### Do not display result

If your query returns a lot of rows, just don't display them

`set nodata on` 

### Enable showplan

`set showplan on`

### Enable dbcc flags

Flag 9528 formats the output of plancost in case it's too large

`dbcc traceon(3604,9528)`

#### optionnaly disable the execution if you just want to see the plan

`set noexec on`