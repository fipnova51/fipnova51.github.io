---
published: true
title: An introduction to Oracle optimizer and statistics
layout: post
tags: [oracle, optimzer, statistics, performance]
categories: [database]
---
*This article is an introduction to Oracle statistics and optimizer, very important topics about the good execution of a SQL statement. We'll first look at the statistics then how to check the execution plan of a statement*

<!--excerpt-->

## How to generate statistics on tables and index

Oracle stores statistics used by the optimizer in mainly in two tables

* USER_TAB_STATISTICS: store general statistics information about a table (number of rows, number of blocks, avg row length)
* USER_TAB_COL_STATISTICS: store statistics about the data itself  per column (histogram, frequency cell)

you can use the query below to get information about statistics (whether a column has some or not, when they were generated...)

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> TABLE_NAME<span style='color:#808030; '>,</span> OBJECT_TYPE<span style='color:#808030; '>,</span> NUM_ROWS<span style='color:#808030; '>,</span> LAST_ANALYZED<span style='color:#808030; '>,</span> USER_STATS<span style='color:#808030; '>,</span> STALE_STATS <span style='color:#800000; font-weight:bold; '>from</span> USER_TAB_STATISTICS <span style='color:#800000; font-weight:bold; '>where</span> table_name<span style='color:#808030; '>=</span><span style='color:#0000e6; '>'T'</span><span style='color:#808030; '>;</span> <span style='color:#696969; '>-- USER_TAB_STATISTICS displays optimizer statistics for the tables owned by the current user, especially check column LAST_ANALYZED and STALE_STATS (check table definition for additional field to display)</span>
<span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#808030; '>*</span> <span style='color:#800000; font-weight:bold; '>from</span> USER_TAB_COLUMNS <span style='color:#800000; font-weight:bold; '>where</span> TABLE_NAME<span style='color:#808030; '>=</span><span style='color:#0000e6; '>'T'</span><span style='color:#808030; '>;</span> <span style='color:#696969; '>-- USER_TAB_COLUMNS describes the columns of the tables, check column HISTOGRAM to see whether statistics exist or not</span>
<span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#808030; '>*</span> <span style='color:#800000; font-weight:bold; '>from</span> USER_TAB_COL_STATISTICS <span style='color:#800000; font-weight:bold; '>where</span> TABLE_NAME<span style='color:#808030; '>=</span><span style='color:#0000e6; '>'T'</span><span style='color:#808030; '>;</span> <span style='color:#696969; '>-- USER_TAB_COL_STATISTICS contains column statistics and histogram information extracted from "USER_TAB_COLUMNS" (check table definition for additional field to display)</span>
<span style='color:#800000; font-weight:bold; '>select</span> INDEX_NAME<span style='color:#808030; '>,</span>STATUS<span style='color:#808030; '>,</span> LAST_ANALYZED<span style='color:#808030; '>,</span> CLUSTERING_FACTOR<span style='color:#808030; '>,</span> NUM_ROWS <span style='color:#800000; font-weight:bold; '>from</span> USER_INDEXES <span style='color:#800000; font-weight:bold; '>where</span> INDEX_NAME <span style='color:#800000; font-weight:bold; '>like</span> <span style='color:#0000e6; '>'ACT_BAT_%'</span><span style='color:#808030; '>;</span> <span style='color:#696969; '>-- check if index statistics are up-to-date</span>
</pre>
<br>
The statistics must be present and up-to-date.

* If USER_TAB_STATISTCS.STALE_STATS is empty it means the table has no statistics, you must gather them
* If USER_TAB_STATISTCS.STALE_STATS is equal to YES it means the statistics are not up to date, you must gather them
* for an index, it's possible to check when its statistics were updated with USER_INDEXES.LAST_ANALYZED

*Statistics are considered STALLED when more than STALE_PERCENT(default 10%) of the rows are changed (total number of inserts, deletes, updates) in a table. Those informations are stored in table USER_TAB_MODIFICATIONS and you can use procedure DBMS_STATS.FLUSH_DATABASE_MONITORING_INFO in order to recompute STALE_PERCENT and indirectly STALE_STATS.*

### How to update statistics?

<pre style='color:#000000;background:#ffffff;'><span style='color:#696969; '>-- check the possible arguments for command dbms_stats.gather_table_stats and dbms_stats.gather_index_stats</span>
<span style='color:#696969; '>-- you need to be the owner of the objects or have the ANALYZE ANY privilege or the DBA role</span>
<span style='color:#696969; '>-- for a table</span>
dbms_stats<span style='color:#808030; '>.</span>gather_table_stats<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'&lt;SCHEMA_NAME>'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'&lt;TABLE_NAME>'</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>degree</span><span style='color:#808030; '>=</span><span style='color:#808030; '>></span><span style='color:#0000e6; '>'AUTO_DEGREE'</span><span style='color:#808030; '>,</span> no_invalidate<span style='color:#808030; '>=</span><span style='color:#808030; '>></span><span style='color:#800000; font-weight:bold; '>false</span><span style='color:#808030; '>)</span><span style='color:#808030; '>;</span>
<span style='color:#696969; '>-- for an index</span>
dbms_stats<span style='color:#808030; '>.</span>gather_index_stats<span style='color:#808030; '>(</span>ownname <span style='color:#808030; '>=</span><span style='color:#808030; '>></span><span style='color:#0000e6; '>'&lt;SCHEMA_NAME>'</span><span style='color:#808030; '>,</span> indname <span style='color:#808030; '>=</span><span style='color:#808030; '>></span> <span style='color:#0000e6; '>'&lt;INDEX_NAME>'</span><span style='color:#808030; '>)</span>
</pre>
<br>
By default, statistics are generated per column, it's sometime interesting to generate some statistics on a **group of columns**, you can use the command below

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> dbms_stats<span style='color:#808030; '>.</span>create_extended_stats<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'&lt;SCHEMA_NAME>'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'&lt;TABLE_NAME>'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'(&lt;LIST_OF_COLUMNS>)'</span><span style='color:#808030; '>)</span> <span style='color:#800000; font-weight:bold; '>from</span> dual<span style='color:#808030; '>;</span>
<span style='color:#800000; font-weight:bold; '>exec</span> dbms_stats<span style='color:#808030; '>.</span>gather_table_stats<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'&lt;SCHEMA_NAME>'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'&lt;TABLE_NAME>'</span><span style='color:#808030; '>)</span>
</pre>
<br>
This will generate a new entry in USER_TAB_COL_STATISTICS with COLUMN_NAME value a bit random

<table border="1" style="width:100%">
<tr>
<th>COLUMN_NAME</th><th>NUM_DISTINCT</th><th>NUM_NULLS</th><th>HISTOGRAM</th>
</tr>
<tr>
<td>1</td><td>2</td><td>3</td><td>4</td>
</tr>
</table>

!({{ site.url }}/assets/pictures/stats_grouped_cols.png)