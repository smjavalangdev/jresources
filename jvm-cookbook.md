JVM Cookbook
=============

####JVM Safepoints
Why a Java application was spending a significant amount of time paused, even when garbage collection cycles
were only taking ~200ms? The issue turned out to be other safepoints.

the VM uses safepoints to perform a variety of internal operations, and they involve pausing every Java thread. 
Garbage collection is the most well known, but many other operations such as deoptimization, and revoking biased 
locks require a safepoint as well. Here is an article that explains more about [JVM safepoints](http://blog.ragozin.info/2012/10/safepoints-in-hotspot-jvm.html)

```bash

-XX:+PrintGCApplicationStoppedTime
-XX:+PrintSafepointStatistics
-XX:PrintSafepointStatisticsCount=1

```
The output on safepoint statistics is shown below.
```bash
[time: spin block sync   cleanup vmop] page_trap_count
[      0    3471 3583    531     5342]  0
```
The first part tells you what operation is being performed “RevokeBias” in this case,
which means that a biased lock is being revoked. The next three numbers are the total number of threads, 
the number that were running and contributed to the “spin” time, and the number of threads which 
contributed to the “block” time (shown below).

```bash
[time: spin block sync   cleanup vmop] page_trap_count
[      0    3471 3583    531     5342]  0

```
This part is the most interesting. It tells us how long (in milliseconds) the VM spun waiting for threads to
reach the safepoint. Second, it lists how long it waited for threads to block. The third number is the total time
waiting for threads to reach the safepoint (spin + block + some other time). Fourth, is the time spent in 
internal VM cleanup activities. Fifth, is the time spent in the operation itself (RevokeBias in this case).

In this example, we can see pretty clearly that the pause was caused by 3.5secs spent waiting for threads
to block, and then an additional 5secs revoking the biased lock.
