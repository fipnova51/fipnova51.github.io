---
published: true
title: An introduction to Oracle optimizer and statistics
layout: post
tags: [oracle, optimzer, statistics, performance]
categories: [database]
---
*This article is an introduction to Oracle statistics and optimizer, very important topics about the good execution of a SQL statement. We'll first look at the statistics then how to check the execution plan of a statement*

<!--excerpt-->

# Let's start wit few words about statistics

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

![my helpfull post]({{ site.url }}/assets/pictures/stats_grouped_cols.png)

You can retrieve this 'pseudo' column definition in table USER_STAT_EXTENSION

![my helpfull post]({{ site.url }}/assets/pictures/grouped_cols_definition.png)
<br>
## Few recommendations about gathering statistics

Use default parameter for *ESTIMATE_PERCEN* and *METHOD_OPT*

### DEGREE parameter

This parameter specifies the number of server processes to use for statistics gathering, by default it derives the value from the *CREATE TABLE* attribute specify if any. The default value is 1. It can be interesting to set this value to *AUTO_DEGREE* to let Oracle compute the best parallelization degree

### Statistics gathering precedence

The way statistics are gathered can be controlled at table level (SET_TABLE_PREFS), schema level (SET_SCHEMA_PREFS), database level (SET_DATABASE_PREFS) or globally (SET_GLOBAL_PREFS).

![my helpfull post]({{ site.url }}/assets/pictures/stats_gathering_precedence.png)

To retrieve the valued that will be used, execute

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> dbms_stats<span style='color:#808030; '>.</span>get_prefs<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'&lt;PARAMETER_NAME>'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'SCHEMA_NAME'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'TABLE_NAME'</span><span style='color:#808030; '>)</span> <span style='color:#800000; font-weight:bold; '>from</span> dual<span style='color:#808030; '>;</span>
</pre>
<br>

### Gathering statistics consideration

Note also that Oracle automatically gather statitistics during Oracle maintenance windows time using *DBMS_STATS.GATHER_DATABASE_JOB_PROC*. You can check the activation of this automatic gathering by querying *DBA_AUTOTASK_CLIENT_JOBB* view or with OEM. In case you need to disable this job, always keep one job that will gather statistics for dictionary tables. You can change the setting with

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>BEGIN</span>
    DBMS_STATS<span style='color:#808030; '>.</span>SET_GLOBAL_PREFS<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'AUTOSTATS_TARGET'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'ORACLE'</span><span style='color:#808030; '>)</span><span style='color:#808030; '>;</span>
<span style='color:#800000; font-weight:bold; '>END</span><span style='color:#808030; '>;</span>
<span style='color:#808030; '>/</span>
</pre>
<br>

Concurrent statistics gathering *DBMS_STATS.GATHER_DATABASE_STATS, DBMS_STATS.GATHER_SCHEMA_STATS or DBMS_STATS.GATHER_DICTIONARY_STATS* allows gathering statistics on multiple tables. This is controlled by the global parameter *CONCURRENT* (default FALSE) and *JOB_QUEUE_PROCESSES* initialization parameter (number of individual job that can be executed at the same time, parameter to be changed at system level with command *ALTER SYSTEM*... or in init.ora file).

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>BEGIN</span>
DBMS_STATS<span style='color:#808030; '>.</span>SET_GLOBAL_PREFS<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'CONCURRENT'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'TRUE'</span><span style='color:#808030; '>)</span><span style='color:#808030; '>;</span>
<span style='color:#800000; font-weight:bold; '>END</span><span style='color:#808030; '>;</span>
<span style='color:#808030; '>/</span>
</pre>
<br>

It's then also possible to parallelize the execution of a single job with DEGREE parameter

Once the statistics are up-to-date, we now need to check the plan used by a SQL statement but how?

# Let's continue with getting the execution plan

It's very hard to say that an execution plan is better than another (is using an index always better than a table scan? not sure). One way to check that an execution plan is good is to compare estimated rows returned by a query which are based on the statistics and the actual rows. If the numbers differ way too far, it means the statistics are not up-to-date meaning the optimizer is not using the right execution plan. In order to get those figures add *gather_plan_statistics* in your select statement

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#696969; '>/*+ gather_plan_statistics */</span>       
<span style='color:#bb7977; font-weight:bold; '>count</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>data</span><span style='color:#808030; '>)</span>  
<span style='color:#800000; font-weight:bold; '>from</span> t 
<span style='color:#800000; font-weight:bold; '>where</span> x <span style='color:#808030; '>=</span> <span style='color:#008c00; '>5</span>
</pre>
<br>

There are several ways to get the execution plan depending on the context

1. How to get the execution plan of a past SQL executed ?
2. How to get the execution plan of a running SQL ?
3. What would the plan for a SQL statement ?

## How to get the execution plan of a past SQL executed ?

Simple form is to give a representative expression in the WHERE clause for the query below

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> plan_table_output
<span style='color:#800000; font-weight:bold; '>from</span>   v$<span style='color:#800000; font-weight:bold; '>sql</span> <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>,</span>
       <span style='color:#800000; font-weight:bold; '>table</span><span style='color:#808030; '>(</span>dbms_xplan<span style='color:#808030; '>.</span>display_cursor<span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>sql_id<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>child_number<span style='color:#808030; '>,</span><span style='color:#0000e6; '>'[ALL | ALLSTATS LAST]'</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span> t
<span style='color:#800000; font-weight:bold; '>where</span>  <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>sql_text <span style='color:#800000; font-weight:bold; '>like</span> <span style='color:#0000e6; '>'select * from ACT_BAT_DBF'</span><span style='color:#808030; '>;</span>
</pre>
<br>

you can also use an alternate solution which is to get the *sql_id* first then pass it to the first query by changing the where clause (not sure of the interest)

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>select</span> <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>sql_id<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>sql_text <span style='color:#800000; font-weight:bold; '>from</span> V$<span style='color:#800000; font-weight:bold; '>SQL</span> <span style='color:#800000; font-weight:bold; '>s</span> <span style='color:#800000; font-weight:bold; '>where</span> <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>sql_text <span style='color:#800000; font-weight:bold; '>like</span> <span style='color:#0000e6; '>'select * from ACT_BAT_DBF'</span><span style='color:#808030; '>;</span>
<span style='color:#696969; '>--</span>
<span style='color:#800000; font-weight:bold; '>select</span> plan_table_output
<span style='color:#800000; font-weight:bold; '>from</span>   v$<span style='color:#800000; font-weight:bold; '>sql</span> <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>,</span>
       <span style='color:#800000; font-weight:bold; '>table</span><span style='color:#808030; '>(</span>dbms_xplan<span style='color:#808030; '>.</span>display_cursor<span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>sql_id<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>child_number<span style='color:#808030; '>,</span><span style='color:#0000e6; '>'[ALL | ALLSTATS LAST]'</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span> t
<span style='color:#800000; font-weight:bold; '>where</span>  <span style='color:#800000; font-weight:bold; '>s</span><span style='color:#808030; '>.</span>sql_id <span style='color:#808030; '>=</span> <span style='color:#0000e6; '>'4vj1jmvjuwxxh'</span><span style='color:#808030; '>;</span>
</pre>
<br>

## How to get the execution plan of a running SQL ?

Check previously or execute your SQL statement  then execute

    select * from table(dbms_xplan.display_cursor(format => '[ALL | ALLSTATS LAST]'));

<font color='red'>User must have "SELECT privilege on V$SQL, V$SQL_PLAN, V$SQL_PLAN_STATISTICS_ALL" to work successfully</font>

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>grant</span> <span style='color:#800000; font-weight:bold; '>SELECT</span> <span style='color:#800000; font-weight:bold; '>on</span> V_$SQL_PLAN_STATISTICS_ALL <span style='color:#800000; font-weight:bold; '>to</span> <span style='color:#808030; '>&lt;</span>USERNAME<span style='color:#808030; '>></span><span style='color:#808030; '>;</span>
<span style='color:#800000; font-weight:bold; '>grant</span> <span style='color:#800000; font-weight:bold; '>SELECT</span> <span style='color:#800000; font-weight:bold; '>on</span> V_$SQL_PLAN <span style='color:#800000; font-weight:bold; '>to</span> <span style='color:#808030; '>&lt;</span>USERNAME<span style='color:#808030; '>></span><span style='color:#808030; '>;</span>
<span style='color:#800000; font-weight:bold; '>grant</span> <span style='color:#800000; font-weight:bold; '>SELECT</span> <span style='color:#800000; font-weight:bold; '>on</span> V_$<span style='color:#800000; font-weight:bold; '>SQL</span> <span style='color:#800000; font-weight:bold; '>to</span> <span style='color:#808030; '>&lt;</span>USERNAME<span style='color:#808030; '>></span><span style='color:#808030; '>;</span>
</pre>
<br>

## What would be the plan for a SQL statement ?
<font color='red'>The PLAN returned in this section is the 'possible' execution plan but might differ from the real used plan. The later can be retrieved with point 1 and 2</font>

This is done using the syntax EXPLAIN PLAN

### Create the PLAN output table

the plan is stored in a table (default table: PLAN_TABLE) that must be created or updated before its usage. The script for its creation is provided with Oracle

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>SQL</span><span style='color:#808030; '>></span> <span style='color:#797997; '>@</span><span style='color:#808030; '>/</span>opt<span style='color:#808030; '>/</span>oracle<span style='color:#808030; '>/</span><span style='color:#008c00; '>11204</span><span style='color:#808030; '>/</span>rdbms<span style='color:#808030; '>/</span><span style='color:#800000; font-weight:bold; '>admin</span><span style='color:#808030; '>/</span>utlxplan<span style='color:#808030; '>.</span><span style='color:#800000; font-weight:bold; '>sql</span>
</pre>
<br>

It's also possible to store the information in another table, in this case you just need to rename the table created by the script *utlxplan.sql*.

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>RENAME</span> PLAN_TABLE <span style='color:#800000; font-weight:bold; '>TO</span> my_plan_table<span style='color:#808030; '>;</span>
</pre>
<br>

### Running EXPLAIN PLAN

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>EXPLAIN</span> <span style='color:#800000; font-weight:bold; '>PLAN</span>
  <span style='color:#808030; '>[</span><span style='color:#800000; font-weight:bold; '>SET</span> <span style='color:#800000; font-weight:bold; '>STATEMENT_ID</span><span style='color:#808030; '>=</span><span style='color:#0000e6; '>'any_id'</span><span style='color:#808030; '>]</span>
  <span style='color:#808030; '>[</span><span style='color:#800000; font-weight:bold; '>INTO</span> <span style='color:#808030; '>&lt;</span>your PLAN_output_table<span style='color:#808030; '>></span><span style='color:#808030; '>]</span> <span style='color:#800000; font-weight:bold; '>FOR</span>
<span style='color:#808030; '>&lt;</span>YOUR <span style='color:#800000; font-weight:bold; '>SQL</span> <span style='color:#800000; font-weight:bold; '>QUERY</span><span style='color:#808030; '>></span>
</pre>

### Displaying the PLAN_TABLE output

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>SELECT</span> PLAN_TABLE_OUTPUT <span style='color:#800000; font-weight:bold; '>FROM</span> <span style='color:#800000; font-weight:bold; '>TABLE</span><span style='color:#808030; '>(</span>DBMS_XPLAN<span style='color:#808030; '>.</span>DISPLAY<span style='color:#808030; '>(</span><span style='color:#0000e6; '>'&lt;PLAN_TABLE_NAME>'</span><span style='color:#808030; '>,</span> <span style='color:#0000e6; '>'&lt;STATEMENT_ID>'</span><span style='color:#808030; '>,</span><span style='color:#0000e6; '>'[ALL | ALLSTATS LAST]'</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>;</span>
</pre>
<br>