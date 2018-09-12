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

**sequenceiq** is offering a Hadoop Docker image, which contains everything, we are going to use, so, why not use this one?

Get your container up and running, by using the following command:

```
docker run -it -p 50070:50070 -p 8088:8088 -p 8020:8020 sequenceiq/hadoop-docker:2.7.0 /etc/bootstrap.sh -bash
```