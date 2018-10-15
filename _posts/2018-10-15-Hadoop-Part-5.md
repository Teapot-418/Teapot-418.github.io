---
layout: post
title: Hadoop - Taming The Elephant, Part 5
tags: hadoop big-data tutorial
url: Hadoop-Part-5
description: Hadoop has two Java APIs, where are the differences?
image: Hadoop-5.png
---

Hadoop offers two APIs for Java, an old one and a new one, where are the differences? And how can we make use of the new one?

This post will show you...

* ... the differences between Java API 1 and Java API 2
* ... how to write a MapReduce job in Java API 2

![Hadoop 5 cover picture](../images/Hadoop-5.png){:class="img-responsive"}

## Introduction

Hadoop 2 came along with an updated version of the Java API, this new API is also called "new API", while the older one is also called "old API". The new API replaces the older one, and both are functionally fully equal.

Both APIs are shipped with Hadoop, but they cannot be used together.

## Differences between API 1 and API 2

Of course, you want to know, how to identify, if something is written with the new or the old API, therefore  you need to know the main differences between them.

* Both APIs are stored within their own package inside Hadoop:
    * New API: `org.apache.hadoop.mapreduce`
    * Old API: `org.apache.hadoop.mapred`
* The new API uses abstract classes over interfaces for easier evolving
* The new API uses context objects, which allow the user to communicate with MapReduce
* The new API allows mappers and reducers to control the execution flow
* In the new API, `Job` performs job control. `JobClient` does not exist anymore
* Now there are mapper-output-files `part-m-nnnnn` and reducer-output-files `part-r-nnnnn`
* The new API offers a way to support graceful cancellation of long-running operations on interrupts, by declaring user-overridable methods to throw `java.lang.InterruptedException`
* The new API allows the usage of for-each-loops by using `java.lang.Iterable` as parameter-type in `reduce()`-method

*This is a summary of 'Appendix D. The Old and New Java MapReduce APIs' ('Hadoop, The Definitive Guide (5th Edition), Tom White')*

## MapReduce job in Java API 2

To create a MapReduce-job with Java API 2, aka the new API, it is necessary to have the correct dependencies in your pom-file.

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>3.1.1</version>
</dependency>

<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-client-core</artifactId>
    <version>3.1.1</version>
</dependency>
```

The old and new API both have in common, that you have to bundle your application as a jar, that it can be used. This has also been done in [Part 2](/Hadoop-Part-2) of the tutorial series.

Now, that we have included the needed dependencies, we can start writing mapper, reducer and runner, like we did in [Part 2](/Hadoop-Part-2), but now we will use the new API.

The example we will build now, extracts main categories from DarkNet offers and counts, how often they appear within the dataset.

### Mapper

The mapper takes an identifier as key of each csv-line and the line itself as value. It transfers the line-data into an array and splits category at it's separator `/`. If this split has more items than zero, there is a main-category present, this will be added to ths category-list, which combines main-category and line-key.

The first difference between those APIs can be identified quickly, the output is written to `context`, not to `output`.

```java
package mappers;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import shared.Data;

import java.io.IOException;

public class MainCategoryMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

    @Override
    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();

        String[] lineData = line.split(",");

        String[] categories = lineData[1].split("/");

        if(categories.length > 0) {
            context.write(new Text(categories[0]), key);
        }
    }
}
```

### Reducer

The first thing, you'll notice, is that the Reducer is not named `MainCategoryReducer`, the reason is, that you can use this reducer also for reducing the category count.

This reducer takes a `Text` as key and provides an `Iterable` as values. This eases the handling of values, because we can simply use a for-each-loop to loop over the entries. We have an iterator, counting the elements in each main-category.

The reducer's output is written to `context` like the mapper output before.

```java
package reducers;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class CategoryCountReducer extends Reducer<Text, LongWritable, Text, IntWritable> {

    @Override
    public void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;

        for (LongWritable value : values) {
            count++;
        }

        context.write(key, new IntWritable(count));
    }
}
```

### Runner

To show you a possibility of handling multiple execution-types with one MapReduce-application, we will create an interface for runners.

#### Interface

The interface contains a run-method, which needs an input- and output-path as parameters. Every runner will have to implement this interface.

```java
package shared;

public interface Runner {
    public void run(String inputPath, String outputPath) throws Exception;
}
```

#### Implementation

The `MainCategoryCount`-Runner implements the previously created `Runner`-interface, which means, that we have to implement the `run()`-method.

While the job-configuration in the old API was done by using `JobClient` and `JobConf`, we can now use `Job` for everything. We have to configure a name, input and output path, mapper and reducer class, mapper and reducer output key and value class.

Running the job is done with `System.exit(job.waitForCompletion(true) ? 0 : 1);`.

```java
package runners;

import reducers.CategoryCountReducer;
import mappers.MainCategoryMapper;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import shared.Runner;

public class MainCategoryCount implements Runner {
    @Override
    public void run(String inputPath, String outputPath) throws Exception {
        Job job = Job.getInstance();
        job.setJobName("Main category count");

        FileInputFormat.addInputPath(job, new Path(inputPath));
        FileOutputFormat.setOutputPath(job, new Path((outputPath)));

        job.setMapperClass(MainCategoryMapper.class);
        job.setReducerClass(CategoryCountReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

#### Call it

Since we have introduced the interface `Runner`, we can create a new `Runner`, which is in this case the specialization `MainCategoryCount()` and `run()` it, by handing over input and output path.

```java
Runner runner = new MainCategoryCount();
runner.run(inputPath, outputPath);
```

### Conclusion

The usage of Java API 2 is not that much different than the usage of Java API 1. Due to the fact, that both are possible to use,  I think, it is good to know, that there are two APIs and how to use them.

## What's up next?

Next time, we will see, what YARN is all about.