# Elasticsearch

## 1. Introduction
### What is Elasticsearch
Elasticsearch can do full-text searches, but not only that. It can also query structured data such as numbers and aggregate data, and use Elasticsearch as an analytics platform. 

A quite common use case of Elasticsearch is APM, which stands for Application Performance Management. It allows you to analyze stored logs from applications and various server system metrics. You can even use alerts in case an error occurs multiple times within a certain time range. 

Elasticsearch can also be used for anomality detection. If your website normally has 50.000 visitors, and now only has 5.000, there probably is something wrong. You can figure that out manually, but that's pretty time consuming. What you can do it let machine learning learn the "norm" and let you know when there's an anormality. This is all done for you in Elastisearch.

In Elasticsearch data is stored as documents, which is just a unit of information. A document is essentially just a JSON object. 

You query documents by using the Elastic REST API. The queries are also written in JSON. 

### The Elastic stack
#### Kibana
Kibana is an analytics and visualization platform, which lets you easily visualize data from Elasticsearch and analyze it to make sense of it. You can think of Kibana as a web interface of the data that is stored within Elasticsearch. 

#### Logstash
Traditionally, Logstash has been used to process logs from applications and send them to Elasticsearch. That's still a popular use case, but Logstash has evolved into a more general purpose tool, meaning that Logstash is a data processing pipeline. The data that Logstash receives will be handeled as events. These events are processed by Logstash and shipped off to one or more destinations. That could be Elasticsearch, a Kafka queue, an e-mail message or an HTTP endpoint. 

#### X-Pack
X-Pack is a pack of features that adds additional functionality to Elasticsearch and Kibana. A couple of its features are:
- Adding authentication and authorization to both Kibana and Elastisearch.
- Monitoring the performance of the Elastic stack.
- Alerting
- Reporting: export the Kibana visualizations and dashboards to PDF files.
- Machine learning
- Graph: adding relationships in your data.
- SQL: Use SQL queries to send to Elasticsearch.

#### Beats
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
Please not that all examples are made for Elasticsearch version 5.6, because that's what's running for a project at Macaw I'm working on.

### Creating and deleting indices
Create an index by running `PUT /products`. Delete it by running `DELETE /product`. 

You can set the number of shards and the number of replica shards by running 
```
PUT /products 
{
    "number_of_shards": 2,
    "number_of_replicas": 2
}
```

### Indexing documents
```
POST /products/doc
{
    "name": "Coffee Maker",
    "price": 50,
    "in_stock": 10
}
```

The response we get:
```
{
  "_index": "products",
  "_type": "doc",
  "_id": "AW9BvCbl9lp_qflGeFCe",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "created": true
}
```

The ID is important for updating the document later on.

### Retreiving documents by ID
A simple request like this:
```
GET /products/doc/AW9BvCbl9lp_qflGeFCe
```

Returns the selected document:
```
{
  "_index": "products",
  "_type": "doc",
  "_id": "AW9BvCbl9lp_qflGeFCe",
  "_version": 1,
  "found": true,
  "_source": {
    "name": "Coffee Makers",
    "price": 50,
    "in_stock": 10
  }
}
```

### Updating documents
```
POST /products/doc/AW9BvCbl9lp_qflGeFCe/_update
{
  "doc": {
    "in_stock": 9
  }
}
```

### Scripted updates
Elasticsearch supports scripting, which enables you to write custom logic while accessing a document's values. For example, if you want to reduce the `in_stock` by 1, without having to know what the previous `in_stock` value was. 

```
POST /products/doc/AW9BvCbl9lp_qflGeFCe/_update
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```

Ctx stands for context. Whe can access the source document through its `_source` property, which gives us an object containing the document's fields. In this case, we accessed the `in_stock` property and subtracted one from it. 

It is also possible to provide parameters to the script, so you can subtract an exact value from `in_stock`, for example.

```
POST /products/doc/AW9BvCbl9lp_qflGeFCe/_update
{
  "script": {
    "source": "ctx._source.in_stock -= params.count",
    "params": {
        "count": 4
    }
  }
}
```

### Deleting a document
Due to Elasticsearch using REST for their API, it's easily guessable what command should be run for deleting a document:
```
DELETE /products/doc/AW9BvCbl9lp_qflGeFCe
```

## 4. Mapping
In Elasticsearch, mappings are used to define how documents and their fields should be stored and indexed. The point of doing this, is to store and index data in a way that is appropriate for how we want to search our data. A couple of examples of what mappings can be used for, could be to define which fields should be treated as full text fields, which fields contain numbers, dates, or geographical locations. You can also specify the date formats for date fields, and also specify analyzers for full text fields. You can kind of think of mappings in Elasticsearch as the equivalent of defining a schema for a table in a relational database, such as MySQL. 

### Dynamic mapping
Elasticsearch supports a thing called dynamic mapping. This way it detects the data type by looking at the data provided. If a property contains a value "2000/01/01", it will detect and store it as a date.

To look at which mappings have been added automatically through dynamic mapping, run the query `GET /products/doc/_mapping`

You can see the results for yourself, but one thing I want to mention is how text fields are treated. 

A part of the mapping response looks like this:
```
"name": {
    "type": "text",
    "fields": {
        "keyword": {
        "type": "keyword",
        "ignore_above": 256
        }
    }
}
```

Notice how the "name" field is mapped as the type "text", but also notice that it has a "fields" property containing a field named "keyword" with a type of "keyword". What this means is that the field has two mappings. By default, each text field is mapped using both the "text" type and the "keyword" type. The difference between the two, is that "text" type is used for full-text searches, and the "keyword" type for exact matches, aggregations and such. 

### Meta fields
Every document that is stored within an Elasticsearch cluster, has some meta-data associated with them. 
- `_index`: Contains the name of the index to which a document belongs.
- `_id`: Stores the ID of a document.
- `_source`: Contains the original JSON object used when indexing a document. 
- `_field_names`: Contains the names of every field that contains a non-null value.
- `_routing`: Stores the value used to route a document to a shard.
- `_version`: Stores the internal version of a document.
- `_meta`: May be used to store custom data that is left untouched by Elasticsearch.

### Field data types
#### Core data types
- Text data type: `text` - used to index full-text value such as descriptions. Values are analyzed. 
- Keyword data type: `keyword` - used for structured data (tags, categories, e-mail addresses, ...). Not analyzed. Typically for filtering and aggregations.
- Numeric data types: `float`, `long`, `short` and more.. - Basic numeric data types, most of which are found in various programming languages.
- Date data type: `date` - Represents dates as either a string, long or integer. The date format may be configured.
- Boolean data type: `boolean` - Stores boolean values.
- Binary data type: `binary` - Accepts a Base64 encoded binary value. Not stored by default. 
- Range data type: `integer_range`, `float_range`, `long_range`, `double_range`, `date_range` - Used for range values such as date ranges or numeric ranges (e.g. 10-20).

#### Complex data types
- Object data type - Added as JSON objects; stored as key-value pairs internally. May contain nested objects. 
- Array "data type" - Not an actual data type, because each field may contain multiple values by default.
- Nested data type: `nested` - Specialized version of the `object` data type. Enables arrays of objects to be querified independently of each other. 

#### Geo data types
- Geo-point data type: `geo_point` - Accepts latitude-longitude pairs. Used for various geographical operations.
- Geo-shape data type: `geo_shape` - Used for geographical shapes such as polygons, circles etc.

#### Specialized data types
- IP data type: `ip` - Used for storing IPv4 and IPv6 IP addresses.
- Completion data type: `completion` - Used to provide auto-completion ("search-as-you-type") functionality. Optimized for quick lookups.
- Attachment data type: `attachment` - Used to make text from various document formats searchable (e.g. PPT, PDF, RTF, ...). Requires the Ingest Attachment Processor Plugin.

## 5. Analysis & Analyzers
When indexing a document its full-text fields are run through an analysis process. By full-text fields we're refering to fields of the type `text` and not `keyword` fields. `keyword` fields are not analysed.

Basically it involves tokenizing text into terms, lowercasing text etc. More generally speaking, the analysis process involves tokenizing and normalizing a block of text. This is done to make the text easier to search. You have full control over the analyzer process because it's possible to control which analyzer is used. The standard analyzer is sufficient in most cases though. 

### A closer look at analyzers
An analyzer consists of three things:
- Character filters
- Token filters
- Tokenizer

An analyzer is basically a package of these building blocks with each one of them changing the input stream. So when indexing a document it goes through the following flow. 

First, zero or more character filters. A character filter receives a text fields original text and can then transform the value by adding, removing or changing characters. An example of this could be to strip out any HTML markup. 

Afterwards, a tokenizer splits the text into individual tokens, which will usually be words. An analyzer may only have 1 tokenizer. It basically splits by whitespace and also removes most symbols such as commas, periods and semicolons. That's because most symbols are not useful when it comes to searching as they are intended for being read by humans.

After splitting the text into tokens, it runs through zero or more token filters. A token filter may add, remove or change tokens. This is kind of similar to a character filter, but token filters work with the token stream instead of a character stream. There are a couple of different token filters with the simplest one beign a lowercase token filter, which just converts all characters to lowercase. Another token filter that you can make use of is named "stop". It removes common words which are referred to as stop words. These are words such as "the", "a", "and", "at", etc. These are words that don't really provide any value to a field, in terms of searchability, because each word gives a document very little significance in terms of relevance. Another token filter worth mentioning is one named "synonym", which is useful for giving similar words the same meaning. 

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