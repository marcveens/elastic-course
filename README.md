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

## 13. Improving search results

## 14. Building a web application search engine