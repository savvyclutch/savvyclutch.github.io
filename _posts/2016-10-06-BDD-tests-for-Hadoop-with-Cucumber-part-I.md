---
title: BDD tests for Hadoop with Cucumber. Part I
description: "Tutorial how to create behaviour tests for Hadoop application with Cucumber. Part I: introduction and environment"
layout: post
author: bogdan
tags: [big data, java, hadoop, qa]
modified: 2016-10-07
image:
    feature: posts/bdd_test_hadoop/title.jpg
    
---

Big Data is a processing of a huge amount of structured (more or less) data that should be filtered or sorted for future analysis or, 
actually made analysis. So much data that you can’t process it on the one computer for a reasonable time. 
This can be logs from services, like call service, or logs from web servers that contain billions of records, 
so you should process data in parallel  on a bunch of computers. To do it so we use specific software based on MapReduce pattern. 
In our case, we use Hadoop with Cascading framework. Hadoop implements MapReduce and Cascading framework contains a lot of useful tools, 
for example abstraction to Amazon S3, so we can use S3 buckets as input/output folders for processing jobs, 
but at the same time we allow to use regular file systems on developer machines for development and test purposes.

As any other piece of software it should be tested before using in production, especially because of high cost of logical mistakes in processing a lot amount of data. 
Unfortunately there are no much information about BDD testing of Hadoop and Hive jobs. 
So I decided to write how we do it in our Cascading project here, in [Intelliarts](http://intelliarts.com/).

<!-- more -->

# What do we have

Assume we have a lot of data about phone calls in a bunch of files. File contains lines in the following format: `Id,country,time,duration` 
Where `id` is a call unique id, `country` is a caller country, `time` is a time when call started (timestamp) and `duration` is a call duration in milliseconds. 

For example: 

```
0000,UA,1433998201,60000
0001,US,1433998201,30000
0001,GB,1433998301,30000
...
```

# What we want to do?

Let's sort calls from these records by country (each country will be in a separate folder) and change the format of records - replace delimiters from `,` with tab symbol `\t`. 
From the data of previous example so we should get the folders `US`, `GB`, `UA` and `UA` will contain file with records looks like this:

```
id	country	time	duration
0000	UA	1433998201	60000
```

And, we want to be sure that we did not make any mistakes during implementation, so we want to test the results. 
What we will use:  

* [Hadoop](http://hadoop.apache.org/) + [Cascading](http://www.cascading.org/) for data processing
* [Gradle](https://gradle.org/) for build management
* [Cucumber](https://cucumber.io/) for tests
* [Chunk Templates](https://github.com/tomj74/chunk-templates) for test data fixture generation
* [Docker](https://www.docker.com/) for environment isolation

First of all we should isolate our testing environment and make it portable between dev machines and CI. We use Docker for this propose. 
Inside the container we will:

1. Install Java
2. Install Hadoop
3. Install Gradle
4. Add codebase

Here is Dockerfile content:

{% highlight groovy %}
FROM ubuntu:14.04

MAINTAINER Bogdan Frankovskyi <bogdan@savvyclutch.com>

RUN apt-get update ; apt-get upgrade -y

### INSTALL JAVA
RUN apt-get install -y default-jdk wget software-properties-common

### ADD HADOOP
RUN mkdir -p /usr/local/hadoop
RUN cd /tmp/ ; wget http://mirrors.sonic.net/apache/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz ; tar xzf hadoop-2.6.0.tar.gz ; mv hadoop-2.6.0/* /usr/local/hadoop

### ADD GRADLE
RUN add-apt-repository ppa:cwchien/gradle
RUN apt-get update ; apt-get install -y gradle

ENV GRADLE_USER_HOME /cache

#### ADD THE DIRECTORY FOR CODEBASE
ADD . /opt/bdd_test_hadoop/
WORKDIR /opt/bdd_test_hadoop/

#### DOWNLOAD AND INSTALL DEPENDENCIES
RUN gradle dependencies && gradle install

#### RUN TESTS

CMD gradle cucumber
{% endhighlight %}

Okey, now we have the environment. Let’s create a project gradle config to download all dependencies for Hadoop application:

{% highlight groovy %}
{% raw %}

buildscript {
  repositories {
    jcenter()
  }

  dependencies {
    classpath 'de.undercouch:gradle-download-task:2.0.0'
    classpath 'org.apache.hadoop:hadoop-common:2.6.0'
  }
}


apply plugin: 'java'
apply plugin: 'de.undercouch.download'
apply plugin: 'maven'

sourceCompatibility = 1.7
targetCompatibility = 1.7

ext.cascadingVersion = '3.1.0'
ext.hadoop2Version = '2.6.0'
ext.hiveVersion = '1.0.0'

repositories {
    mavenLocal()
    mavenCentral()
    maven{ url 'http://conjars.org/repo/' }
}

configurations {
    providedCompile

    cucumberRuntime {
        exclude group: 'commons-httpclient'
        extendsFrom testRuntime
    }
}

sourceSets {
    main {
        compileClasspath += configurations.providedCompile
    }

    test {
        output.resourcesDir = 'build/classes/test/resources'
        output.classesDir   = 'build/classes/test/java'
    }
}

dependencies {
    // install cascading
    compile( group: 'cascading', name: 'cascading-core', version: cascadingVersion )
    compile( group: 'cascading', name: 'cascading-local', version: cascadingVersion )
    compile( group: 'cascading', name: 'cascading-hadoop2-mr1', version: cascadingVersion )
    compile( group: 'cascading', name: 'cascading-hadoop2-tez', version: cascadingVersion )

    compile( group: 'com.hotels', name: 'corc-cascading', version: '1.0.0' )

    // install hadoop
    providedCompile( group: 'org.apache.hadoop', name: 'hadoop-common', version: hadoop2Version )
    providedCompile( group: 'org.apache.hadoop', name: 'hadoop-client', version: hadoop2Version )
    providedCompile( group: 'org.apache.hadoop', name: 'hadoop-mapreduce-client-core', version: hadoop2Version )

    compile( group: 'org.apache.hive', name: 'hive-exec', version: hiveVersion )
    compile( group: 'org.apache.hive', name: 'hive-serde', version: hiveVersion )

    cucumberRuntime files("${jar.archivePath}")
    testCompile( group: 'org.apache.hadoop', name: 'hadoop-common', version: hadoop2Version )
    testCompile( group: 'org.apache.hadoop', name: 'hadoop-client', version: hadoop2Version )
    testCompile( group: 'org.apache.hadoop', name: 'hadoop-mapreduce-client-core', version: hadoop2Version )

    // install cucumber
    testCompile( group: 'info.cukes', name: 'cucumber-java', version: '1.2.4' )
    testCompile( group: 'info.cukes', name: 'cucumber-junit', version: '1.2.4' )
    testCompile( group: 'info.cukes', name: 'cucumber-picocontainer', version: '1.2.4' )

    // install chunk templates for test data generation
    testCompile( group: 'com.x5dev', name: 'chunk-templates', version: '3.1.0' )
}


task cucumber() {
    dependsOn assemble, compileTestJava, processTestResources
    doLast {
        javaexec {
            main = "cucumber.api.cli.Main"
            classpath = configurations.cucumberRuntime + sourceSets.test.runtimeClasspath
            args = ["--plugin", "pretty",  "--tags", "~@ignore", "--glue", "com.steps", "src/test/resources"]
        }
    }
}


jar {
    description = "Assembles a Hadoop ready jar file"
    doFirst {
        into( 'lib' ) {
            from configurations.compile
        }
    }

    manifest {

    }
}

{% endraw %}
{% endhighlight %}

As you can see there are two custom tasks - `cucumber` and `jar`.  First one runs cucumber tests and the second one compiles the source to the production jar-file.
Now we can build the image:

    $ docker build -t bdd_hadoop .

Time to build processing application. Let’s create a directory structure: `src > main > java > com > processing` and add a file CallStream.java with content:

{% highlight java %}
package com.processing;

import cascading.flow.FlowConnector;
import cascading.flow.hadoop2.Hadoop2MR1FlowConnector;
import cascading.operation.regex.RegexParser;
import cascading.pipe.Each;
import cascading.pipe.Pipe;
import cascading.property.AppProps;
import cascading.scheme.hadoop.TextDelimited;
import cascading.scheme.hadoop.TextLine;
import cascading.tap.SinkMode;
import cascading.tap.Tap;
import cascading.tap.hadoop.GlobHfs;

import org.apache.hadoop.mapred.JobConf;

import cascading.tap.hadoop.Hfs;
import cascading.tap.hadoop.PartitionTap;
import cascading.tap.partition.DelimitedPartition;
import cascading.tuple.Fields;


import java.util.*;

// main processor class
public class CallStream {

    private static Tap createOutputTap(String path) {
        TextDelimited scheme = new TextDelimited( true, "\t");

        Hfs hfsTap = new Hfs(scheme, path);
        DelimitedPartition partition = new DelimitedPartition(new Fields("country"), "/");
        return new PartitionTap(hfsTap, partition, SinkMode.REPLACE);
    }

    /**
     * Gets the input and output path as arguments 
     */
    public static void main(String[] args) {
        final List<Pipe> pipes = new ArrayList<Pipe>();
        final Map<String, Tap> sinks = new HashMap<String, Tap>();

        String inputPath = args[0];
        String outputPath = args[1];

        Fields lines = new Fields("line");
        Tap inTap = new GlobHfs(new TextLine(lines), inputPath);

        // Declare the field names used to parse out of the log file
        Fields callstreamFields = new Fields("id", "country", "time", "duration");

        // Define the regular expression used to parse the log file
        String logRegex = "^(\\d+),([a-zA-Z]{2}?),(\\d+),(\\d+)$";

        // Declare the groups from the above regex. Each group will be given a field name from 'callstreamFields'
        int[] allGroups = {1, 2, 3, 4};

        // Create the parser
        RegexParser parser = new RegexParser(callstreamFields, logRegex, allGroups);
        Pipe streamPipe = new Each("CallStream", lines, parser, callstreamFields);

        // Add new pipe
        pipes.add(streamPipe);

        // give the partition to output
        sinks.put("CallStream", createOutputTap(outputPath));

        // configure cascading
        JobConf jobConf = new JobConf();

        // pass class to the flow connector
        Properties properties;
        properties = AppProps.appProps()
                .setName(CallStream.class.toString())
                .setJarClass(CallStream.class)
                .addTags("app:cascading")
                .buildProperties(jobConf);

        // execute flow
        FlowConnector flowConnector = new Hadoop2MR1FlowConnector(properties);
        flowConnector.connect(inTap, sinks, pipes).complete();
    }
}

{% endhighlight %}

Let’s try to build jar file:

    $ docker run --rm bdd_hadoop gradle jar

Now we need to create some acceptance tests. This will be in the the Part II.

