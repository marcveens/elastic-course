# Elasticsearch

## 1. Introduction
### What is Elasticsearch
Elasticsearch can do full-text searches, but not only that. It can also query structured data such as numbers and aggregate data, and use Elasticsearch as an analytics platform. 

A quite common use case of Elasticsearch is APM, which stands for Application Performance Management. It allows you to analyze stored logs from applications and various server system metrics. You can even use alerts in case an error occurs multiple times within a certain time range. 

Elasticsearch can also be used for anomality detection. If your website normally has 50.000 visitors, and now only has 5.000, there probably is something wrong. You can figure that out manually, but that's pretty time consuming. What you can do it let machine learning learn the "norm" and let you know when there's an anormality. This is all done for you in Elastisearch.

In Elasticsearch data is stored as documents, which is just a unit of information. A document is essentially just a JSON object. 

You query documents by using the Elastic REST API. The queries are also written in JSON. 

### The Elastic stack
### Kibana
Kibana is an analytics and visualization platform, which lets you easily visualize data from Elasticsearch and analyze it to make sense of it. You can think of Kibana as a web interface of the data that is stored within Elasticsearch. 

### Logstash
Traditionally, Logstash has been used to process logs from applications and send them to Elasticsearch. That's still a popular use case, but Logstash has evolved into a more general purpose tool, meaning that Logstash is a data processing pipeline. The data that Logstash receives will be handeled as events. These events are processed by Logstash and shipped off to one or more destinations. That could be Elasticsearch, a Kafka queue, an e-mail message or an HTTP endpoint. 

### X-Pack
X-Pack is a pack of features that adds additional functionality to Elasticsearch and Kibana. A couple of its features are:
- Adding authentication and authorization to both Kibana and Elastisearch.
- Monitoring the performance of the Elastic stack.
- Alerting
- Reporting: export the Kibana visualizations and dashboards to PDF files.
- Machine learning
- Graph: adding relationships in your data.
- SQL: Use SQL queries to send to Elasticsearch.

### Beats
Beats is a collection of so-called data shippers. They are lightweight agents with a single purpose that you install on servers, which then sends data to Logstash or Elasticsearch. 

For example there is a beat called Filebeat, which is used for collecting log files and sending the log entries off to either Logstash or Elasticsearch. 

## 2. Getting started
### Installing on Windows
You can download the Elasticsearch and Kibana files from https://www.elastic.co/downloads/elasticsearch and unpack them in a directory to your choice.

To start Elasticsearch, you need to open a command prompt and navigate to the path you just unpacked and run `elasticsearch.bat` from the bin folder. 

To start Kibana, you also need to navigate to the path of the recently downloaded Kibana using a command prompt. After that, run `kibana.bat` from the bin folder. 

Everything necessary should be running now. To verify this, navigate to `http://localhost:5601` in a browser to see the Kibana UI. 

### Architecture
When starting up Elastic, you basically start up a node. A node is essentially an instance of Elasticsearch that stores data. To make sure you can store terabytes of data, you can run as many nodes as you want. Each node will then store a part of the data. This way, you can store data on multiple virtual or physical machines. 

A cluster is a collection of related nodes that together contain all of your data. You can have many clusters if you want to, but one is usually enough. 

Each unit of data that you store within your cluster is called a document. Documents are JSON objects containing whatever data you desire. When you index a document, the original JSON object that you sent to Elasticsearch is stored along with some metadata that Elasticsearch uses internally. 

Documents are organized within indices. Every document within Elasticsearch is stored within an index. An index groups documents together logically, as well as provide configuration options that are related to scalability and availability. 

### Sharding and scaling
One of the ways which Elasticsearch can scale is by having one or more nodes within a cluster. Elasticsearch does this by something called sharding. Sharding is a way to divide an index into separate pieces, where each piece is called a *shard*. Sharding is done at the index level, and not at cluster nor node level. The main reason for dividing an index into multiple shards is to be able to horizontally scale the data volume. For example, if you have an index of 600GB worth of data, but only have 2 nodes with a capacity of 500GB each, there's no way a single shard could fit on any node. This is when you can split up the index in 2 shards, and put each of them on a separate node. 

Another advantage of sharding is that it enabled queries to be distributed and parallelized across an index' shards. What this means, is that a search query can be run on multiple shards at the same time, increasing the performance and throughput. 

### Replication
Replication is like sharding also configured at the index level. Replication works by creating copies of each of the shards that an index contains. These copies are referred to as replicas or replica shards. A shard that has been replicated one or more times is referred to as a primary shard. A primary shard and its replications are referred to as a replication group. The number of replications can be configured at index creation. 

Elasticsearch also supports taking snapshots. They can be used to restore a given point in time. Snapshots can be taken at index level or for the entire cluster. You should use snapshots for backups, and replication for high availability and performance. 

## 3. Managing documents 

## 4. Mapping

## 5. Analysis & Analyzers

## 6. Introduction to searching

## 7. Term level queries

## 8. Full text queries

## 9. Adding boolean logic to queries

## 10. Joining queries

## 11. Controlling query results

## 12. Aggregations
Aggregation is a way of grouping and extracting statistics and summaries from your data. Suppose you have and index where each document corresponds to an order and each order contains an amount and a product ID. A simple example of what you can do with aggregations is to group orders by product ID and then sum the amounts to see how much each product was sold for.



## 13. Improving search results

## 14. Building a web application search engine