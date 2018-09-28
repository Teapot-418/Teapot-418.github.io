---
layout: post
title: Hadoop - Taming The Elephant, Part 3
tags: hadoop big-data tutorial
url: Hadoop-Part-3
description: Access the results, you produced, from "outside"
image: Hadoop-3.png
---

This post will show you, how to...

* ... start a Docker container linked to the previously used one
* ... use HttpFS for receiving data
* ... receive results from the previous container via cURL
* ... create an application working with the previous generated data

![hadoop-3-title](../images/Hadoop-3.png){:class="img-responsive"}

## What is HttpFS?

In a nutshell HttpFS provides a REST HTTP gateway, that supports all filesystem operations, that can be performes in HDFS.


## Prerequisites

This tutorial is based on the two posts before, therefore please follow the instructions of:
* [Part 1](../Hadoop-Part-1)
* [Part 2](../Hadoop-Part-2)

## Use HttpFS

### Prerequisites

To explore HttpFS, we are going to use another Docker container, linked to the Docker container of [part 1](../Hadoop-Part-1) of the tutorial series. The real exploring is then done with the data created in [part 2](../Hadoop-Part-2) of these tutorial series.

Let's start a Docker container linked to the already running container, which contains Hadoop.

1. Get the ID or name of the container running Hadoop. (`docker ps` or `docker container ls` will help you)
2. To start a linked container run:
    ```shell
        docker run -it --link <docker-container-name>:<internal-name> --name <name-of-container-starting> <image-to-start>
    ```
    Explanation of variables:
    * `docker-container-name` - The name or ID, you discovered in step 1 (*sleepy_hopper*)  
    * `internal-name` - The name, we will reference the already running container internally with (*hadoop*)  
    * `name-of-container-starting` - How should the new container be named? (*explorer*)  
    * `image-to-start` - Which image should the new container start? (*openjdk:8-jre-alpine*)  

### Receive results via cURL

### Handle results in application

## What's up next?

In the next part, I will show, how to build a MapReduce job without using Java. Yes, there is another way besides the Java API.