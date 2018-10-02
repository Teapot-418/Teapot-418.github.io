---
layout: post
title: Hadoop - Taming The Elephant, Part 4
tags: hadoop big-data tutorial
url: Hadoop-Part-4
description: MapReduce without Java? Yes, it is possible!
image: Hadoop-4.png
---

Is it possible to write a MapReduce job without Java? The answer is yes.

This post will show you, how to...

* ... use Hadoop's Streaming API
* ... write a MapReduce job in Python

![Hadoop 4 cover picture](../images/Hadoop-4.png){:class="img-responsive"}

## What is Hadoop's Streaming API?

Hadoop Streaming provides an API to write MapReduce jobs in any language, that supports Unix standard input and output. Testing can be done in the command line with Unix pipes, which comes in as a handy feature.

A big difference between the Java API and Hadoop Streaming is, that the Java API provides an iterator over each key-group while in Hadoop Streaming the programmer has to find key-group boundaries.

> Good to know:  
> The MapReduce framework ensures the order of the keys

## Prerequisites

This part is only related to [part 1](../Hadoop-Part-1) of this series, the only requirement is to have the Hadoop docker container up and running.

## How to make use of Hadoop's Streaming API

Both, mapper and reducer will receive the data via Unix standard input. The mapper will receive it in the order it appears in the input file, the reducer will receive data ordered by key, which makes it handier to go on with reducing the data.

### Mapper

We will write a Python script, handling the standard output line by line. It has to split the data at the given separator (`,` in this case) to receive an array with the data. To make the data reducable, we generate a unique id, which is the line-number. To hand the data over to our reducer (via MapReduce), we only have to print the results into Unix standard output.

```python
#!/usr/bin/env python

import sys

id = 0

for line in sys.stdin:
  val = line.strip()
  data = val.split(',')
  category = data[1]
  print(category + '\t' + str(id))
  id = id + 1
```

### Reducer

The reducer is also written in Python. We will receive the data ordered by key, but we have to make sure, that the counting is done correctly by remembering the line before, to check, if we are still looking at the same category as before. The results are handed over by using Unix standard output.

```python
#!/usr/bin/env python

import sys

previous_key = None
count = 0

for line in sys.stdin:
    (key, val) = line.strip().split('\t')
    if previous_key is None:
        previous_key = key
    if key == previous_key:
        count = count + 1
    else:
        print(key + '\t' + str(count))
        count = 1
    previous_key = key
```

### Run it

The first step is to transfer mapper and reducer into the running Docker container (`docker cp`). To run mapper and reducer with Hadoop Streaming API, call:

```shell
hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-*.jar
	-files <mapper-file>,<reducer-file>
	-input <input-file>
	-output <output-name>
	-mapper <mapper-file>
	-reducer <reducer-file>
```

* `-files` ships the listed files to the cluster
* `-input` defines the input-file for the MapReduce job
* `-output` defines the output-file of the MapReduce job
* `-mapper` defines the mapper for MapReduce
* `-reducer` defines the reducer for MapReduce

## Nice to know: Combiner

If you're reading something about Hadoop and MapReduce, you may run into *combiners*. For the small Docker-Hadoop-installation it is not interesting, but if you want to run a Hadoop cluster, combiners can make your MapReduce execution faster.

A combiner combines the content of the map output and sends the already combined output to the reducer, which minimizes network-traffic. The combiner would usually perform the same steps a reducer would perform, so you are often able to use your reducer as combiner. Combiners do not replace the reducer.

## What's part 5 about?

In [part 2](../Hadoop-Part-2) we've created a MapReduce job with Java. The program was written in Java API 1. Next time, we have a look at the differences between API 1 and 2 and we are going to create a MapReduce job in API version 2 to spot the differences.