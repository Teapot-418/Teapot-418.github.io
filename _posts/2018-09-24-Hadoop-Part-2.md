---
layout: post
title: Hadoop - Taming The Elephant, Part 2
tags: hadoop big-data tutorial
url: Hadoop-Part-2
description: Want to know, how to write a MapReduce job and run it? Read this post.
image: Hadoop-2.png
---

This post will show you, how to...

* ... put data into HDFS
* ... write a MapReduce job in Java
* ... run the job on the imported data
* ... read data from HDFS
* ... remove data from HDFS

![hadoop-2-title](../images/Hadoop-2.png){:class="img-responsive"}

## What is MapReduce?

MapReduce is a software framework, which enables you to write jobs, that can easily process a big set of data. MapReduce consists of two parts, a mapper and a reducer. The mapper splits up the data into independent data-parts, the reducer combines the splitted parts. Both, mapper and reducer, have key-value pairs for input and output.

## Prerequisites

If you want to use this tutorial as a hands-on, you'll need:

* Running Docker-container from part 1 of the series
* JDK (8+) and a Java-IDE of your choice
* Maven

## Get data into HDFS

I will use a dataset from Kaggle, but slightly changed for easier handling.
* [Original dataset](https://www.kaggle.com/philipjames11/dark-net-marketplace-drug-data-agora-20142015)
* [Modified dataset](https://github.com/Teapot-418/hadoop-taming-the-elephant/blob/master/darknet-data.csv)

If you want to use your own dataset, don't hesitate.

### Get it in

1. Copy the csv-file into the running Docker container (`docker cp`).
2. Add the dataset into HDFS (perform this command inside the running Docker container):
    ```
    /usr/local/hadoop/bin/hdfs dfs -put <file-path> /user/root/input/<name-in-hdfs>
    ```

### Did it work?

There are two ways to check it out:

#### Command Line

```
/usr/local/hadoop/bin/hdfs dfs -ls /user/root/input
```
This command will show all the available datasets within the folder `input` in the root's user directory. If `<name-in-hdfs>` is listed, it's and indicator, that everything worked fine.

#### UI

Visit the Hadoop Overview page at `localhost:50070`, navigate to `"Utilities"/"Browse the file-system"` and browse to the root's user-directory. Check, if `<name-in-hdfs>` is listed.

## Write MapReduce job in Java

The Java MapReduce job is going to consist of three main parts, the mapper, the reducer and a start-up class.  
We are going to create a MapReduce-job, that counts how often a category appears within the dataset.

### Prerequisites

To get the basics for building a MapReduce-job, Hadoop-core has to be included as a Maven-dependency:

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-core</artifactId>
    <version>${hadoop.version}</version>
</dependency>
```
*I've used version 1.2.1.*

MapReduce-jobs has to be bundled as a jar to be runnable: 

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>Entrypoint</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```
*`Entrypoint` is the class-name of the class containing the main-method.*

### The Mapper

The mapper-class is going to split the data into independent chunks. It handles key-value-pairs as input and it will produce key-value-pairs as output. Here is my example Mapper-class:

```java
package categorycount;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reporter;
import shared.Data;

import java.io.IOException;

public class CategoryCountMapper extends MapReduceBase implements Mapper<LongWritable, Text, Text, LongWritable> {

    @Override
    public void map(LongWritable key, Text value, OutputCollector<Text, LongWritable> output, Reporter reporter) throws IOException {
        String csvLine = value.toString();

        String[] splitCsvLine = csvLine.split(Data.SEPARATOR); // Data.SEPARATOR = ","

        String category = splitCsvLine[Data.CATEGORY]; // Data.CATEGORY = 1

        output.collect(new Text(category), key);
    }
}
```

Mapper and Reducer, both have to extend MapReduceBase. The Mapper has to implement the Mapper-interface, which needs the specification for key-input, value-input, key-output and value-output. By implementing the Mapper-interface, the `map`-method has to be overridden. The method takes key and value as input, as well as a output collector, which collects the mapper's output, and a reporter, it allows additional output for tracking the job's state, while it is running.

This mapper is going to handle one line of a csv-file at a time. The key is a value of type long, which equals the line-id, the value is text, which equals the csv-line.

Hadoop uses special types for input and output keys and values, f.e.
* Long - LongWritable
* Int - IntWritable
* String - Text

The mapper above takes the csv-line, splits it at it's separator and afterwards maps a category with it's corresponding line id.

### The Reducer

The reducer-class is going to combine the splitted data. The results of mapping will be handed over as the keys and a list of corresponding values, so to say. Have a look at the example reducer-class:

```java
package categorycount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reducer;
import org.apache.hadoop.mapred.Reporter;

import java.io.IOException;
import java.util.Iterator;

public class CategoryCountReducer extends MapReduceBase implements Reducer<Text, LongWritable, Text, IntWritable> {

    @Override
    public void reduce(Text key, Iterator<LongWritable> values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
        int count = 0;

        while(values.hasNext()) {
            count++;
            values.next();
        }

        output.collect(key, new IntWritable(count));
    }
}
```

Instead of the Mapper-interface, the reducer implements the Reducer-interface, which also needs the specification of key-input, value-input, key-output and value-output. By implementing the Reducer-interface, the reduce-method has to be overridden. The method takes key and an iterator of the value's type as input, as well as an output collector and reporter.

The reducer will get the categories as keys and a list with ids as values. We initialize a count-variable, which will be incremented while there is another item in the list.

This results in having a map with category and corresponding quantity.

### Start-up

The start-up part is going to be split up into two classes. One containing the start of the job, and the other one, containing the configuration for this job. This enables us to easily add multiple ways of execution to this job, by passing an additional command line argument later on.

#### Definition of a Runner class

This class contains all necessary configuration, which has to be made, to run the job.  

```java
package categorycount;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.*;

public class CategoryCountRunner {

    public void run(String inputFile, String outputFile) throws Exception {
        JobClient client = new JobClient();

        JobConf conf = new JobConf(CategoryCountRunner.class);
        conf.setJobName("Category Count");
        conf.setMapOutputKeyClass(Text.class);
        conf.setMapOutputValueClass(LongWritable.class);
        conf.setOutputKeyClass(Text.class);
        conf.setOutputValueClass(IntWritable.class);
        conf.setMapperClass(CategoryCountMapper.class);
        conf.setReducerClass(CategoryCountReducer.class);
        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(TextOutputFormat.class);

        FileInputFormat.setInputPaths(conf, new Path(inputFile));
        FileOutputFormat.setOutputPath(conf, new Path(outputFile));

        client.setConf(conf);

        try {
            JobClient.runJob(conf);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

As you can see, we pass two arguments to the `run`-method, an input- and an output-path. These define, where the data to work on, is located in HDFS, and where the processed data has to be stored in HDFS. Additionally we define things like job name, which is the mapper and which the reducer class, what's the type of map output etc.

#### Definition of Entrypoint

This class calls the `run`-method of the runner class.

```java
import categorycount.CategoryCountRunner;

public class Entrypoint {
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("Usage <input-path> <output-path>");
            System.exit(-1);
        }

        String inputFile = args[0];
        String outputFile = args[1];

        CategoryCountRunner analysisRunner = new CategoryCountRunner();
        analysisRunner.run(inputFile, outputFile);

    }
}
```

> Please make sure to include the main-method inside the Entrypoint class.

## Run the job

Now, that we have created all classes needed, everything must be bundled as a jar. Simply run:

```
mvn clean package
```

*The jar can be found in the `target`-folder.*

### Get it into the container

To put the jar into the running Hadoop container, use the `docker cp`-command presented in tutorial part 1.

### Run it

Jump into the container and execute:

```
/usr/local/hadoop/bin/hadoop jar <jar-path> <input> <output>
```

`jar-path` describes, where the jar is located in Docker's file system.  
`input` describes, where the input-dataset is stored in HDFS.
`output` describes, where the output-dataset should be stored in HDFS.

> The arguments passed after the jar-definition are the command-line arguments, the jar is handling.

### See the results

There are two ways to check the results:

1. Check it with the command line:
   ```
   /usr/local/hadoop/bin/hdfs dfs -cat <output>/*
   ```
2. Check it at Hadoop Overview Page:
   `localhost:50070`, navigate to `"Utilities"/"Browse the file-system"` and browse to the root's user-directory. There you can find the output-folder.

> Created folders contain multiple files: of multiple parts,
> * a `_SUCCESS` description file and 
> * the results, called something like `part-XXXXX`.

### Remove the results

If you want to clean up the content of HDFS, please run:

```
/usr/local/hadoop/bin/hadoop fs -rmr <folder-name>
```

This will remove the folder called `<folder-name>`.

## What's part 3 going to be about?

Now, that we have created results with MapReduce, we are going to enter those results from another Docker container via HttpFS (Hadoop HDFS over HTTP), so that we can make use of the results within another application.