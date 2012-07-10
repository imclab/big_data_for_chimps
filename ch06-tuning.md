
There are enough knobs and twiddles on a hadoop installation to fully stock the cockpit of a 747, and some of them interact surprisingly. Here's our approach to configuring and optimizing Hadoop.

Fundamentally, what you want to know is

* What are the maximum practical capabilities of my system, and are they reasonable?
* How do I tell what constraints a job is hitting, and whether it's reasonable to optimize it?
* If I need to optimize a job, what settings are relevant and what are their tradeoffs?

Coarsely speaking, jobs are constrained by one of these four capabilities:

* RAM: Available memory per node,
* Disk IO: Disk throughput,
* Network IO: Network throughput, and
* CPU: Computational throughput.

Your job is to

* **Recognize when your job significantly underperforms** the practical expected throughput, and if so, whether you should worry about it. If your job's throughput on a small cluster is within a factor of two of a job that does nothing, it's not worth tuning. If that job runs nightly and costs $1000 per run, it is.
* **Identify the limiting capability**.
* **Ensure there's enough RAM**. If there isn't, you can adjust your the memory per machine, the number of machines, or your algorithm design.
* **Not get in Hadoop's way**. There are a few easily-remedied things to watch for that will significantly hamper throughput by causing unneccesary disk writes or network traffic.
* **When reasonable, adjust the RAM/IO/CPU tradeoffs**. For example, with plenty of RAM and not too much data, increasing the size of certain buffers can greatly reduce the number of disk writes: you've traded RAM for Disk IO.

### Measuring your system: theoretical limits

What we need here is a ready-reckoner for calculating the real costs of processing. We'll measure two primary metrics:

* throughput, in `GB/min`.
* machine cost in `$/TB` -- equal to `(number of nodes) * (cost per node hour) / (60 * throughput)`. This figure accounts for tradeoffs such as spinning up twice as many nodes versus using nodes with twice as much RAM. To be concrete, we'll use the 2012 Amazon AWS node pricing; later in this chapter we'll show how to make a comparable estimate for physical hardware.

If your cluster has a fixed capacity, throughput has a fixed proportion to cost and to engineer time. For an on-demand cluster, you should 

_note: I may go with min/TB, to have them be directly comparable. Throughput is typically rendered as quantity/time, so min/TB will seem weird to some. However, min/TB varies directly with $/TB, and is slightly easier to use for a rough calculation in your head._

* Measure disk throughput by using the `cp` (copy) command to copy a large file from one disk to another on the same machine, compressed and uncompressed.
* Measure network throughput by using `nc` (netcat) and  `scp` (ssh copy) to copy a large file across the network, compressed and uncompressed.
* Do some increasingly expensive computations to see where CPU begins to dominate IO. 
* Get a rough understanding of how much RAM you should reserve for the operating system's caches and buffers, and other overhead -- it's more than you think.



### Measuring your system: practical limits

* Understand the practical maximum throughput baseline performance against the fundamental limits of the system


* If your runtime departs significantly from the practical maximum throughput

Tuning your cluster to your job makes life simple
* If you are hitting a hard constraint (typically, not enough RAM)