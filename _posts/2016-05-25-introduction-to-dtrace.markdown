---
published: true
title: Introduction to Dtrace
layout: post
tags: [dtrace]
categories: [system]
---
*Dtrace is a tool developped by SUN to trace any function call, from the application to the system and is very usefull to debug an application. It's working on Solaris Mac OSx and FreeBSD*

<!--excerpt-->

## D usage

D can be called in a one-liner mode

<pre style='color:#000000;background:#ffffff;'>
dtrace <span style='color:#44aadd; '>-n</span> program
</pre>

D code can also be saved in a file which is then called

<pre style='color:#000000;background:#ffffff;'>
dtrace <span style='color:#44aadd; '>-s</span> file<span style='color:#800000; font-weight:bold; '>.</span>d
</pre>

Last, if your file start with `#!/usr/sbin/dtrace -s` , you can just call your file for execution

<pre style='color:#000000;background:#ffffff;'>
<span style='color:#800000; font-weight:bold; '>.</span><span style='color:#40015a; '>/file.d</span>
</pre>

<br>

## D structure

D has the following syntax

~~~
probes
probes {actions}
probes /predicate/ {actions}
~~~

### Probe syntax

A probe is made of 4 objects  `provider:module:function:name`

You can leave a blank if you don't want to specify any object. `*` can be used to filter. 

`name`  object often has the value `entry` and `return` when entering and quiting a function

#### Usefull probes

<pre style='color:#000000;background:#ffffff;'>profile<span style='color:#808030; '>:</span><span style='color:#808030; '>:</span><span style='color:#808030; '>:</span>tick-<span style='color:#008c00; '>5</span><span style='color:#808030; '>[</span><span style='color:#0000e6; '>s|m</span><span style='color:#808030; '>]</span> <span style='color:#696969; '>#this allows to fire a probe every x seconds or minutes </span>
syscall<span style='color:#808030; '>:</span><span style='color:#808030; '>:</span><span style='color:#808030; '>:</span> <span style='color:#696969; '>#provider to probe system calls</span>
</pre>

### Predicates

A predicate allow to apply a specific filter when a probe is triggered

<pre style='color:#000000;background:#ffffff;'><span style='color:#40015a; '>/uid</span> <span style='color:#44aadd; '>==</span> <span style='color:#008c00; '>101</span><span style='color:#40015a; '>/</span> <span style='color:#696969; '>#check for a specific uid</span>
<span style='color:#40015a; '>/pid</span><span style='color:#40015a; '>/</span> <span style='color:#696969; '>#check for a non zero pid</span>
<span style='color:#40015a; '>/uid</span> <span style='color:#44aadd; '>==</span> <span style='color:#008c00; '>101</span> <span style='color:#800080; '>&amp;&amp;</span> pid<span style='color:#40015a; '>/</span> <span style='color:#696969; '># check for uid 101 AND non zero pid</span>
</pre>

### Actions

An action syntax is `{action one; action two; action three}`

### D variables
Variable type is kind of dynamic but you can cast the type as in `C` language

<pre style='color:#000000;background:#ffffff;'>a <span style='color:#44aadd; '>=</span> <span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span> <span style='color:#696969; '># a will be of type int</span>
b <span style='color:#44aadd; '>=</span> <span style='color:#0000e6; '>'foo'</span> <span style='color:#696969; '># b will be of type string</span>
<span style='color:#40015a; '>/b</span> <span style='color:#44aadd; '>!=</span> NULL<span style='color:#40015a; '>/</span> /<span style='color:#808030; '>*</span> <span style='color:#bb7977; font-weight:bold; '>test</span> to check that b contains data <span style='color:#800080; '>(</span>not NULL<span style='color:#800080; '>)</span> <span style='color:#808030; '>*</span><span style='color:#40015a; '>/</span>
</pre>

integer types know by Dtrace are

~~~
char: 8-bit characters
short / int16_t: signed 16-bit integer
int / int32_t: signed 32-bit integer
long long / int64_t: signed 64-bit integer
floating-point are used for tracing and formatting but operators cannot be applied on it
~~~

### D operators

`C` operators work with Dtrace

<pre style='color:#000000;background:#ffffff;'>a <span style='color:#44aadd; '>=</span> <span style='color:#800080; '>(</span>b + c<span style='color:#800080; '>)</span> <span style='color:#808030; '>*</span><span style='color:#008c00; '>2</span><span style='color:#800080; '>;</span>
x <span style='color:#44aadd; '>=</span> x + <span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span>  x+=<span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span>  x++<span style='color:#800080; '>;</span>
</pre>

relational operators (`== ; != ; < ; <= ; > ; >=`), boolean operators (`&& (AND), || (OR), ^^ (XOR)`), bitwise operators (`& (and), | (or), ^ (xor), << (shift left), >> (shift right)`) are allowed.

Finally, ternary operators for simple conditional expressions can be used

<pre style='color:#000000;background:#ffffff;'>a <span style='color:#44aadd; '>=</span> b <span style='color:#e34adc; '>></span>= <span style='color:#008c00; '>0</span> <span style='color:#808030; '>?</span> b <span style='color:#808030; '>:</span> <span style='color:#44aadd; '>-b</span> <span style='color:#696969; '># a receives the absolute value of b</span>
</pre>

#### Scalar

Scalar stores individual values and are global meaning it can be accessed from anywhere  with all the possible risks (two threads accessing modifying the value at the same time).

Their usage is discouraged when you can use other types (thread-local or aggregation variables) 

<pre style='color:#000000;background:#ffffff;'>a <span style='color:#44aadd; '>=</span> <span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span>
</pre>

#### Associative arrays

<pre style='color:#000000;background:#ffffff;'>name<span style='color:#808030; '>[</span><span style='color:#0000e6; '>key</span><span style='color:#808030; '>]</span> <span style='color:#44aadd; '>=</span> expression<span style='color:#800080; '>;</span>
</pre>

`key` can contain several values

<pre style='color:#000000;background:#ffffff;'>a<span style='color:#808030; '>[</span><span style='color:#0000e6; '>123, "foo"</span><span style='color:#808030; '>]</span>=<span style='color:#008c00; '>456</span> <span style='color:#696969; '># array a with key 123 and "foo" stores the value 456</span>
</pre>

Associative array has the same problem as scalar as the same key/value can be modified at the same time by multiple CPUs

#### Thread local

Thread-local variables are stored with the current thread of execution. they use prefix `self->`

Thread local variables are prefered to scalar and associative arrays

<pre style='color:#000000;background:#ffffff;'>self-<span style='color:#e34adc; '>></span>x <span style='color:#44aadd; '>=</span> <span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span> <span style='color:#696969; '># declare a thread-local variable x containing value 1 </span>
self-<span style='color:#e34adc; '>></span>x <span style='color:#44aadd; '>=</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span> <span style='color:#696969; '># free the thread-local variable , this is MANDATORY</span>
</pre>

#### Clause local

A `clause local` is a variable to be used within a single of action group `{}`. It uses prefix `this->`

<pre style='color:#000000;background:#ffffff;'>this-<span style='color:#e34adc; '>></span>y <span style='color:#44aadd; '>=</span> <span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span> <span style='color:#696969; '>#declare a clause-variable y and assigns the value of 1. The variable is destroyed at the end of execution</span>
</pre>
<br/>

#### Built-in

Several built-in variables are availabe as scalar global. Example

![dtrace built-ins]({{ site.url }}/assets/pictures/dtrace_builtins.png)

#### Macros

macro allows accessing variables defined outside Dtrace

`/pid == $target/`

`$target` is initialized with the arguments `-p`

`./file.d -p 123`


|   var name   |   var types   |   description   |
|   ------------   |   -----------   |   ---------------   |
| $target | pid_t | process ID specified using -p PID or -c command	|
| $1...$N | int or string | command-line args passed to Dtrace (1M) |
| $$1...$$N |string (forced) | command-line args passed to Dtrace (1M)	|

### Aggregations

Aggregations variables are prefixed with `@`. As some CPUs/Threads may write to the same `global scalar` variable, aggregations prevent this as it will populate per CPUs/Threads buffers which are combined when printed.

Aggregations may have keys

`@a[pid] = count();` will count events separately per `pid`. The aggregations will be printed as a table with the `keys` on the left and the `value`  on the right

![dtrace aggregating functions]({{ site.url }}/assets/pictures/dtrace_aggregating_functions.png)

#### quantize

`quantize` shows a graphical distribution of values with a frequency scale of pwer-two

<pre style='color:#000000;background:#ffffff;'><span style='color:#808030; '>[</span><span style='color:#0000e6; '>root@pcbsd</span><span style='color:#808030; '>-</span><span style='color:#0000e6; '>simon</span><span style='color:#808030; '>]</span> ~<span style='color:#696969; '># dtrace -n 'callout_execute:::callout-start { self->cstart = timestamp; } callout_execute:::callout-end { @length = quantize(timestamp - self->cstart); }'</span>
dtrace<span style='color:#808030; '>:</span> description <span style='color:#0000e6; '>'callout_execute:::callout-start '</span> matched <span style='color:#008c00; '>2</span> probes
^C
</pre>
<br/>
![dtrace quantize functions]({{ site.url }}/assets/pictures/dtrace_quantize.png)

In this example, the most count (599) was for value between 8192 and 16383

