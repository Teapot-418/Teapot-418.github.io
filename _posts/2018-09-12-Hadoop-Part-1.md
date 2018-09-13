---
layout: post
title: Hadoop - Taming The Elephant, Part 1
tags: hadoop big-data tutorial
---

> Big data is a topic, many people talk about nowadays, one of the tools, which are used to handle big data, is Hadoop.

## What can you expect from this tutorial?

A _quick and dirty_ introduction to Hadoop, with a lot of hands-on, without the need for a complete Hadoop-cluster setup.

## What is Hadoop?

Hadoop is used for storing and analysing data. The term Hadoop is an umbrella-term for multiple applications. Four (soon five) applications are building Hadoop's core:

* **Hadoop Common**  
  Common application parts
* **Hadoop Distributed File System (HDFS)**  
  Distributed file system
* **Hadoop YARN (Yet Another Resource Negotiator)**  
  Framework for job-scheduling and cluster resource management
* **Hadoop MapReduce**  
  Parallel processing of big data sets
* _**Hadoop Ozone**_  (Coming soon)
  Object storage

If you're interested in more information on the details of Hadoop, please visit the [Apache website](http://hadoop.apache.org/).

## Prerequisites

If you want to use this tutorial as a hands-on, you need:

* Docker

There are some very useful Docker commands, which will be handy to follow this tutorial:

* **Show all running docker containers**  
  `docker container ls`, `docker ps`   
* **Get a bash in a running container**  
  `docker exec -it <container-id> /bin/bash`  
  (or in Alpine-based image: `docker exec -it <container-id> /bin/ash`)

## Start it

Please be aware, that it is not the default way to run Hadoop within a Docker container. Hadoop is designed to run in big clusters. The Docker container is great for getting started and having some easy-hands-on-experience, but not for production!

[sequenceiq](https://github.com/sequenceiq/hadoop-docker) is offering a Hadoop Docker image, which contains everything, we need, to get started.

Get your container up and running:

```
docker run -it -p 50070:50070 -p 8088:8088 -p 8020:8020 sequenceiq/hadoop-docker:2.7.0 /etc/bootstrap.sh -bash
```

## Test if everything is working

Inside the Docker container are already test-files and MapReduce-jobs. They can help to identify, if everything is working correctly. 
Let's test it:

1. Check the ID of the Docker container (use the command to discover running Docker containers)
2. Go into the bash of the running container (use the command to get a bash inside of a running container)
3. Now you can run one of the examples MapReduce-jobs:
    ```
    cd $HADOOP_PREFIX
    # run the mapreduce-job
    bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep input output 'dfs[a-z.]+'

    # check the output
    bin/hdfs dfs -cat output/*
    ```

