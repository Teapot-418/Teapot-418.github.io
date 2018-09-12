---
layout: post
title: Hadoop - Taming the elephant, Part 1
tags: hadoop big-data tutorial
---

> Nowadays people are talking a lot about big data, machine learning and other trendy topics. This blog-post-series offers an introduction to Hadoop.

## What can you expect from this tutorial?

This blog-posts are written to give a _quick and dirty_ introduction to Hadoop, with a lot of hands-on. The main goal is no deep understanding of Hadoop internals, the main goal is to have a practical example on how to work with Hadoop without the need for a big Hadoop-cluster-setup.

## What exactly is Hadoop?

Hadoop is used for storing and analysing data. The term Hadoop is an umbrella-term for multiple applications. Four applications are building the Hadoop's core:

* **Hadoop Common** - Common application parts
* **Hadoop Distributed File System (HDFS)** - Distributed file system
* **Hadoop YARN (Yet Another Resource Negotiator)** - Framework for job-scheduling and cluster resource management
* **Hadoop MapReduce** - Parallel processing of big data sets

This should be enough theory, let's start with some practical stuff. If you are interested in more details about Hadoop, it's core components or the multiple other applications, which are related to Hadoop, don't hesitate to visit the [Apache website](http://hadoop.apache.org/).

## Prerequisites

If you're interested in doing the steps on your own, you will need:

* Docker
* Dataset from Kaggle
* JDK with your favorite IDE

## Start it

> You've already read, that I am using a Docker image for this demo/tutorial. This is something, that happens on purpose. Hadoop itself is created to run on big clusters, but I wanted to have a _quick and dirty_ introduction. There is nothing more _quick and dirty_ as Docker.

[sequenceiq](https://github.com/sequenceiq/hadoop-docker) is offering a Hadoop Docker image, which contains everything, we are going to use.

Get your container up and running, by using the following command:

```
docker run -it -p 50070:50070 -p 8088:8088 -p 8020:8020 sequenceiq/hadoop-docker:2.7.0 /etc/bootstrap.sh -bash
```

## Test if everything worked

Inside the Docker container, there are already some test-files and also test-MapReduce-jobs available. They can help to identify, if everything is working correctly. Let's test it:

1. Check the ID of the Docker container, by running ```docker container ls``` or ```docker ps```
2. Go into the bash of the running container, by running ```docker exec -it <container-id> /bin/bash```
3. Now you can run one of the examples:
    ```
    cd $HADOOP_PREFIX
    # run the mapreduce
    bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep input output 'dfs[a-z.]+'

    # check the output
    bin/hdfs dfs -cat output/*
    ```