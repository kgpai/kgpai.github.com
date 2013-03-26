---
layout: post
title: "Optimizing for Large EMR Clusters"
description: "Tips to optimize for large EMR Clusters"
category: posts
tags: [aws, hadoop, map, reduce, s3, ganglia, emr]
---
{% include JB/setup %}
These are some tips that the EMR (Elastic Map Reduce) team came up with for running large running EMR clusters. While most of these tips are specifically for EMR, they should be applicable to managed clusters also. 

* If you have a reduce phase in your job, try to insure that you configure a number of reducers (mapred.reduce.tasks) that is a prime number which is roughly 95% of hsr, where h is the number of hosts in your cluster, s is the number of reducer slots per host, and r is some whole number (stands for “rounds”) large enough to produce output files that are of reasonable size.  Prime numbers help the hashing work out, and the 5% cushion allows some failures.  There’s nothing more annoying than having the last round of reduction leave most of an expensive cluster sitting idle.
*       Change the somaxconn setting and ipc.server.listen.queue.size to a much larger value than the default of 128.  We use 16384.
*       Use a beefy master, and tweak the memory allocated to the JobTracker and NameNode processes if necessary.  We use m2.2xlarge for our master and 350 m1.xlarge slaves.
*       Profile using Ganglia on a smaller cluster first to determine the right type of host to use.  (Note: last I tried, Ganglia did not work on a cluster of 350 hosts.  I have seen it work at 225 though.)
*       Be aware of large number of tasks hitting same data nodes at once. If you must hit same file from large number of tasks (ie. Manifest file), either break it into even smaller manifest files, or increase replication factor of the file accordingly, so the poor 3 (default replication) data nodes don’t get slammed.
*       Some nodes will fail/degrade. Do use speculative execution, but only if your tasks are idempotent.
*       S3 is inconsistent, so use manifests (file that contains the names of other files).  You can atomically write a manifest to S3: write to a new file, if you can read it then it is consistent, otherwise wait until it appears).  Two options for producing a manifest: Option 1. Write job output to HDFS, use s3distcp to upload to s3, output a manifest.  Option 2. Write job directly to S3, predict the output file names from the number of reducer tasks, write them to a manifest.
*       Aggregate map side if you can.  For example:
 map(String line) {
  for(String word: words(line)) {
    counts.add(word, 1);
  }
  emitCountsIfFull(counts);
 }
 close() {
  emitCounts(counts);
 }
*       Conserve memory: If your map or reduce tasks need more memory reduce the number of tasks running per node.  Always limit the size of any in-memory data structures.  If you spawn child processes account for their memory usage (reduce the number of task slots per node or memory given to tasks)
*   Avoid skewed reducer keys (watch out for nulls which are likely skewed; if one of your reducers runs a lot longer than others most likely it is skewed)
*   Try to have short run tasks (tasks running 5-10 minutes is good).  Check that your files are split correctly (e.g. you have enough tasks)
*   To debug jobs: Use the Hadoop UI (e.g. see if your job has straggler tasks). Install foxyproxy.  Use the job history file (which is uploaded to S3 if you specify a log URI in your jobflow, contains info about when each task ran as well as exceptions raised for each failed task).
*   Make sure your code is not causing the cluster to fail (e.g. write to /tmp and fill up the root partition causing the OS functions to fail, using a streaming program that uses up all the memory and invokes the OOM killer)
*   Hadoop dislikes lots of small files (files and blocks are name objects in HDFS and occupy namespace-about 150 bytes each- so small files occupy a large portion of the namespace and disk space is underutilized; having lots of small files also produces too many trivial maps and too much hopping from datanode to datanode to retrieve each file) so you should: aggregate files, use larger instance sizes, run multiple map tasks in one JVM (themapred.job.reuse.jvm.num.tasks property), use MultiFileInputSplit which can run more than one split per map, configure number of mappers, use sequence files (e.g key:value pairs where the file contents is the value; you can write a program to put files into a single sequence file and then process it in a streaming fashion)
*   The map output can be compressed before writing to the disk for faster disk writing, lesser disk space, and to reduce the amount of data to transfer to the Reducer. By default the output is not compressed, but it is easy
to enable by setting mapred.compress.map.output to true. The compression library to use is specified by  apred.map.output.compression.codec.
*   io.sort.factor can improve reduce side operations, but also effects map side operations
*   Give shuffle as much memory as possible (make sure mappers and reducers have enough)
*   Avoid spills to disk (one is ok); can use map/reduce side counters for spills
*   If the intermediate data on reduce side can be placed in memory, reduce-side performance will improve
*   If you’re worried about performance you can quantify with Ganglia