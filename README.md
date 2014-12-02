# statsd-jvm-profiler

`statsd-jvm-profiler` is a JVM agent profiler that sends profiling data to StatsD.  Inspired by [riemann-jvm-profiler](https://github.com/riemann/riemann-jvm-profiler), it was primarily built for profiling Hadoop jobs, but can be used with any JVM process.

## Installation

You will need the statsd-jvm-profiler JAR on the machine where the JVM will be running.  If you are profiling Hadoop jobs, that means the JAR will need to be on all of the datanodes.

The JAR can be built with `mvn package`.  You will need a relatively recent Maven (at least Maven 3).

## Usage

The profiler is enabled using the JVM's `-javaagent` argument.  You are required to specify at least the StatsD host and port number to use.  You can also specify the prefix for metrics and a whitelist of packages to be included in the CPU profiling.  Arguments can be specified like so:
```
-javaagent:/usr/etsy/statsd-jvm-profiler/statsd-jvm-profiler.jar=server=%s,port=%s,prefix=bigdata.profiler.%s.%s.%s.%s,filterPackages=com.etsy:com.twitter.scalding:cascading
```

An example of setting up Cascading/Scalding jobs to use the profiler can be found in the `example` directory.

## Metrics

`statsd-jvm-profiler` will profiling the following:

1. Heap and non-heap memory usage
2. Number of GC pauses and GC time
3. Time spent in each function

Assuming you use the default prefix of `statsd-jvm-profiler`, the memory usage metrics will be under `statsd-jvm-profiler.heap` and `statsd-jvm-profiler.nonheap`, the GC metrics will be under `statsd-jvm-profiler.gc`, and the CPU time metrics will be under `statsd-jvm-profiler.cpu.trace`.

Memory and GC metrics are reported once every 10 seconds.  The CPU time is sampled every millisecond, but only reported every 10 seconds.  The CPU time metrics represent the total time spent in that function.

Profiling a long-running process or a lot of processes simultaneously will produce a lot of data, so be careful with the capacity of your StatsD instance.

## Visualization

The `visualization` directory contains some utilities for visualizing the output of the profiler.