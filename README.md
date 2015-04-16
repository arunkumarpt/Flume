# Flume
1. Scalability
2. Parallel processing
3. Redundancy
4. Order
5. Reply

The most obvious change in Flume 1.X is that the centralized configuration master(s) and Zookeeper are gone. 

Another major difference in Flume 1.X is that the reading of input data and the writing of output data are now handled by different worker threads (called Runners)

The Flume agent's architecture can be viewed in this simple diagram. Inputs are called sources and outputs are called sinks. Channels provide the glue between sources and sinks. All of these run inside a daemon called an agent.

##Sources, channels, and sinks

A source writes events to one or more channels.
A channel is the holding area as events are passed from a source to a sink.
A sink receives events from one channel only.
An agent can have many channels.

##Flume events
The basic payload of data transported by Flume is called an event. An event is composed of zero or more headers and a body

The headers are key/value pairs that can be used to make routing decisions or carry other structured information (such as the timestamp of the event or the hostname of the server from which the event originated). You can think of it as serving the same function as HTTP headers—a way to pass additional information that is distinct from the body.

The body is an array of bytes that contains the actual payload. If your input is comprised of tailed log files, the array is most likely a UTF-8-encoded string containing a line of text.

##Interceptors, channel selectors, and sink processors

## The Kite SDK
A Morphline as a series of commands chained together to form a data transformation pipe. (ETL)

##Flume configuration file
A Flume agent's default configuration provider uses a simple Java property file of key/value pairs that you pass as an argument to the agent upon startup

## Hello World

```
agent.sources=s1
agent.channels=c1
agent.sinks=k1

agent.sources.s1.type=netcat
agent.sources.s1.channels=c1
agent.sources.s1.bind=0.0.0.0
agent.sources.s1.port=12345

agent.channels.c1.type=memory

agent.sinks.k1.type=logger
agent.sinks.k1.channel=c1
```
#Sample Run

```
./bin/flume-ng agent -n agent -c conf -f conf/hw.conf -Dflume.root.logger=INFO,console
```

```
[hue@sandbox ~]$ jps
32297 Jps
23787 Application
[hue@sandbox ~]$ nc localhost 12345
Hello World
OK


15/04/16 02:08:08 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 57 6F 72 6C 64                Hello World }

```

##Channels

A memory-backed/nondurable channel and a local-filesystem-backed/durable channel

Flume only provides transactional guarantees for each channel in each individual agent. In a multiagent, multichannel configuration, duplicates and out-of-order delivery are likely but should not be considered the norm. If you are getting duplicates in nonfailure conditions, it means that you need to continue tuning your Flume configurations.

For File, This durability is provided by a combination of a Write Ahead Log (WAL) and one or more file storage directories

Data stored in
```
agent.channels.c1.checkpointDir=/flume/c1/checkpoint
agent.channels.c1.dataDirs=/flume/c1/data
```

Be sure to set HADOOP_PREFIX and JAVA_HOME environment variables when using the file channel. While we seemingly haven't used anything Hadoop-specific (such as writing to HDFS), the file channel uses Hadoop Writables as an on-disk serialization format. If Flume can't find the Hadoop libraries, you might see this in your startup, so check your environment variables:

java.lang.NoClassDefFoundError: org/apache/hadoop/io/Writable

## Splillable memory channels

Introduced in Flume 1.5, the Spillable Memory Channel is a channel that acts like a memory channel until it is full. At that point, it acts like a file channel that is configured with a much larger capacity than its memory counterpart but runs at the speed of your disks 

##Sinks and Sink Processors

HDFS sink
