---
title: Linux Schedulers (A Zine)
output:
  md_document:
    variant: gfm
    preserve_yaml: TRUE
    pandoc_args: 
      - "--wrap=preserve"
knit: (function(inputFile, encoding) {
  rmarkdown::render(inputFile, encoding = encoding, output_dir = "../_posts") })
date: 2021-05-15
permalink: /posts/2021/01/linux-schedulers
excerpt_separator: <!--more-->
toc: true
header: 
  og_image: "posts/gps-gis-osm/plot_final-1.png"
tags:
  - Schedulers
  - Linux
  - Operating Systems
  - Zines
---



This is an illustrated Zine (a small-circulation self-published work) about Linux Schedulers. [Tanvi](https://tanviroy.com) and I made it as a part of our Operating Systems Course with Prof [Manu Awasthi](https://mnwsth.github.io) and we wanted to share it with the world. It's loosely inspired by the amazing ones made by [Julia Evans](https://wizardzines.com)

![](/images/posts/schedulers/cover.jpg)

<!--more-->


## Scheduler Basics

A scheduler is a manager who decides which process gets to run next on the CPU. While doing so, it tries to convince each process that it has its own CPU. Scheduling policy is a balancing act between multiple goals, such as: 

* **Low Latency:** How long does a process wait till it gets to run on the CPU?
* **Progress:** How much does the process accomplish in a given timeframe?
* **Fairness:** Who gets how much and why?

![](/images/posts/schedulers/sched1.jpg)

## Multicore Systems Issues

Since 2004, sinking transistors no longer resulted in faster processors. This led to multicore processors - they increase performance without increase clock-speed. However, that adds a bunch of new goals to attain, such as:

* **Matching Single Processor Policy:** Given enough parallel processes to run, performance in an 8-core machine should be equivalent to a 1-core machine with an 8 times faster processor
![](/images/posts/schedulers/singlematch.png)
* **Scalability:** Architecture should be such, and overheads should be low enough that additional cores translate to increased performance
![](/images/posts/schedulers/scale.png)
* **Maximizing Hardware Features:** Processes continually scheduled on the same processor are likely to find data in the processors' cache. Cache/ Processor-affinity  may thus, significantly affect performance
![](/images/posts/schedulers/hardware.png)

There are 2 main architectures in use, which address some of these issues in multi-core processors:

* **Global Queue:** Each processor selects processes from a shared global run-queue. This natively tries to match single processor policy. Drawbacks include complicated synchronization among processors and poor scalability
![](/images/posts/schedulers/global.png)
* **Distributed Queue:** Each process gets its own run-queue. This makes this approach easily scalable and also uses cache hierarchies effectively. The major drawback is the need for a complicated load-balancing mechanism across all run-queues, to match the single processor policy. 
![](/images/posts/schedulers/distributed.png)

## O(1) Scheduler

### What's this?
A  stable, timesharing scheduler, used in Linux from 2.6 through 2.6.22. It was also used internally by Google till at least 2009

### What's cool?
The next task to run is selected in O(1). In fact every step of the algorithm happens in O(1) - it's  fast!

### What's not?
Uses very complex heuristics (such as average sleep-time) to reward/penalize interactive and non-interactive processes

### Tell me more!
Each active process is given a fixed timeslice to run, after which it is moved to the expired array. When there are no active processes, the active and expired arrays are swapped. Here's the full process:
![](/images/posts/schedulers/O1.png)

## Completely Fair Scheduler
### What's this?
A  proportional-share scheduler still under active development, used as the official replacement for O(1) in Linux (from 2.6.23 onwards) 

### What's cool?
Unlike the O(1) scheduler, CFS uses Red-Black Trees to generate the order of process execution. This drastically reduces the complex heuristics from the 0(1) scheduler.

Several patches have added other features to CFS such as autogrouping (parent & child processes in the same task-group) and better load-balancing

### Tell me more!
Processes are maintained in a (self-balancing) Red-Black Tree, indexed by their vruntime (the total time a process has run for). At every context-switch, process with minimum vruntime is selected (in O(1)). Here's the full process:
![](/images/posts/schedulers/CFS.png)

## Brain F\*ck Scheduler
### What's this?
A  proportional-share scheduler, designed by Con Kolivas in 2009 as an alternative to the CFS. It wasn't intended to be integrated with the mainline Linux kernel

### What's cool?
Uses a fairly simple algorithm and a single unsorted global run-queue. Avoids all complex heuristics and tunable parameters

#### What's not?
Since it uses a single global run-queue, it scales very poorly as the number of processors increase. 

### Tell me more!
Each runnable process is maintained in a single global run-queue, along with its virtual deadline (and the CPU it was last run on). The process with the earliest deadline is chosen to run next. Here's the full process:
![](/images/posts/schedulers/BFS.png)

## Beyond Linux
In this zine, we've covered some of the most popular Linux Schedulers. However, beyond Linux there's still lots left to explore ...

![](/images/posts/schedulers/os.png)

### Solaris
Ships with both a timesharing scheduler (using a classic MLFQ) and a proportional share scheduler (based on decay-usage). Distributed run-queues are used in the multi-core setting
### FreeBSD
FreeBSD's ULE scheduler uses decay-usage to implement a timesharing policy. It also uses distributed run-queues and a push/pull load-balancer
### Windows
Uses a timesharing scheduler implemented as an MLFQ, with temporary priority boosts. Since 2003, Windows has moved from global run-queues to distributed ones for multi-core systems
### OSX
OSX uses a priority-based decay-usage scheduler to implement timesharing. It also uses distributed run-queues

<iframe src="https://www.sohamde.in/files/pdf/sched.pdf" width="100%" height="500" frameborder="no" border="0" marginwidth="0" marginheight="0"></iframe>

*That's it! If you find any errors, please let me know. Feel free to use any material from here (just let me know if you do!)*
