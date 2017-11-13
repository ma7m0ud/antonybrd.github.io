---
title: "About Apache Kafka @ Simple Machine"
image: "/assets/img/opening/opera.png"
tags:
  - Event
  - Kafka
last_modified_at: 2017-11-13T22:17:49-04:00
---

Going to event or presentation is always a good way to learn the latest news about a particular topic. Tonight's event
is "KSQL for Apache Kafka, by Nick Dearden from Confluent", I could not decline the invitation as it is one of the
hotest topic at the moment. For sure, the kind of event it also a great opportunity to meet inspiring people 


Quick highlight of the host: [*Simple machine*](http://simplemachines.com.au/) is a consulting company mainly focused
on Big Data. It exits in Sydney since 2009

Quick highlight of the speaker: [*Nick Dearden*](https://www.linkedin.com/in/ndearden/) works for Confluent, they have their own Kafka distribution and they
support it. We could say that Confluent is to Kafka what Databricks is to Spark.


# "KSQL for Apache Kafka, by Nick Dearden from Confluent"

If you have never heard of KSQL, no worries it is quite a new project currently in version 0.1.X, developer preview only.
It has been launched a couple of months ago by Confluent but it is a free and open source project. 

As a reminder, [Kafka](https://kafka.apache.org/documentation/) it a message broker that was first developed at Linked
In. It is one the more spread framework with Spark and for a long time Kafka, as an Apache project, did not want to go
to 1.0.0 version (maybe to avoid supporting backwards compatibility) but from November 1st the version 1.0.0 has
officially been released.


## What is KSQL?

Let's have a look of the official documentation definition:
 
 > KSQL is an open source streaming SQL engine that implements continuous, interactive queries against Apache Kafka.
 It allows you to query, read, write, and process data in Apache Kafka in real-time, at scale using SQL commands.
 KSQL interacts directly with the Kafka Streams API, removing the requirement of building a Java app.


Confluent wants to create an easier way to interact with Kafka processes, because not everyone in every organisation is
able to use the Java API. There is a well known trade off between Flexibility and Simplicity, KSQL definitely take the
simplicity side.


### What kind of use cases ?

There are many possible use cases, as:
- Enrich data on the flow.
- Real Time monitoring
- Derivation: simply copy a topic to have a new version of it (maybe with only the latest message)

What is really interesting is that your are not query a snapshot of the current state of a table but you can access to
all the changes that occurred in a certain period of time (not too far from now). So you can deal with problem happening
a the very moment to your customer.

Be aware that KSQL is not for:

- Ad-hoc query (No index, limited span of time in Kafka: last X hours or days)
- BI reports (No JDBC and most of the BI tools are not good with continuous results)
- You are not querying table, so no _rank_ or _order by_ concept, because you are querying a flow that is moving.


### How does it work?

You have to provide a schema for the streaming data.
You can mainly do aggregation and windowing.

How the data is distributed during the processing ? 
If the topic key is the same, easy they are on the same node. If not we have to re-shuffle, in real time, KSQL do it
under the wood. For now you can only join 2 different topics. So there is a lot of data duplication, it generates a lot
of traffic and this is why this is a developer only preview.

What's happening it is that Kafka is creating a new topic for each KSQL query.

### Demo: Hands on time

A few basic KSQL commands:

- List the existing topics into the cluster ```show topics;```.
- List the existing streams into the cluster ```show stream;```.
- Describe a stream and its structure ```describe <stream name>;```.

A use-case:

Let's create a table.
```
CREATE STREAM poor-rating AS SELECT * FROM ratings WHERE rating<2 AND user_id>0;
```
And query it.
```
SELECT user_id, rating FROM poor-rating;
```
Here we have all the new records arriving in the topic from the moment we have executed the query.

```
set 'auto.offset.reset'='earliest';
SELECT user_id, rating FROM poor-rating;
```
And now we have all the new records from the beginning of the topic.


## A great implementation of Kafka

The second part of the event is a presentation by Stephane Maarek. The content of this presentation can be found in his
[Blog post on Medium](https://medium.com/@stephane.maarek/how-to-use-apache-kafka-to-transform-a-batch-pipeline-into-a-real-time-one-831b48a6ad85).


Stephane is a Kafka evangelist, go check his [Github](https://github.com/simplesteph) to learn about this awesome message
broker.


Good to know, Schema registry by Confluent take care of schema evolution in Avro !


## Resources and references

<p align="center">
  <iframe src="https://www.youtube.com/embed/A45uRzJiv7I?rel=0&amp;showinfo=0" width="560" height="315" frameborder="0" allowfullscreen="allowfullscreen">
  </iframe>
</p>

- [KSQL Github repository](https://github.com/confluentinc/ksql)

- [Official Kafka documentation](https://kafka.apache.org/documentation/)

- [Stephan Maarek's blog post on Kafka](https://medium.com/@stephane.maarek/how-to-use-apache-kafka-to-transform-a-batch-pipeline-into-a-real-time-one-831b48a6ad85)

- [Confluent's introduction to KSQL](https://www.confluent.io/blog/ksql-open-source-streaming-sql-for-apache-kafka/)
