---
title: Approaches to Anomaly Detection 
output:
  md_document:
    variant: gfm
    preserve_yaml: TRUE
    pandoc_args: 
      - "--wrap=preserve"
knit: (function(inputFile, encoding) {
  rmarkdown::render(inputFile, encoding = encoding, output_dir = "../_posts") })
date: 2021-08-01
permalink: /posts/2021/08/anomaly-detection
excerpt_separator: <!--more-->
toc: true
header: 
  og_image: "posts/plot_final-1.png"
tags:
  - Anomaly Detection
  - Fraud Detection
  - Isolation Forests
  - DBSCAN
---

In working as an intern for the National Stock Exchange I had the chance to learn a little about anomaly detection. This post aims to describe some ideas and approaches to anomaly detection that I learned about. 

[Edit: In updating this post to my website, I decided to once again read recent advances in anomaly detection and I found an excellent paper (see below) that provides an overview of current algorithms for anomaly detection.]

![](/images/anomaly.png)

<!--more-->


## Anomaly Detection Basics

Anomaly detection is the process of identifying rare items, events or observations which deviate significantly from the majority of data. Anomaly detection is relevant in diverse fields like cyber security( identifying atypical user activity)law enforcement ( flagging suspicious behavioiur), financial fraud (isolating suspicious transactions), machine vision, climate modelling ( flagging low likelihood climatic events). 


* **Unsupervised:** Given some data, does some subset of the data not subscribe to the normal form expectation of that data. 
* **Supervised** Training a model on labelled data to create a classifier that can separate anomalies on unseen data.
* **Semi Supervised** Training model on subset of labelled data that also relies on identifying underlying structure of the data. 

![](/images/anomaly.png)

## Types of Anomalies and Outliers


* **Local:** 


* **Global:** 


* **Dependancy:** 

* **Clustered:** 

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

<iframe src="https://arxiv.org/pdf/2206.09426.pdf" width="100%" height="500" frameborder="no" border="0" marginwidth="0" marginheight="0"></iframe>
