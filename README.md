# Elasticsearch

## 1. Introduction
### What is Elasticsearch
Elasticsearch can do full-text searches, but not only that. It can also query structured data such as numbers and aggregate data, and use Elasticsearch as an analytics platform. 

A quite common use case of Elasticsearch is APM, which stands for Application Performance Management. It allows you to analyze stored logs from applications and various server system metrics. You can even use alerts in case an error occurs multiple times within a certain time range. 

Elasticsearch can also be used for anomality detection. If your website normally has 50.000 visitors, and now only has 5.000, there probably is something wrong. You can figure that out manually, but that's pretty time consuming. What you can do it let machine learning learn the "norm" and let you know when there's an anomality. This is all done for you in Elasticsearch.

In Elasticsearch data is stored as documents, which is just a unit of information. A document is essentially just a JSON object. 

You query documents by using the Elastic REST API. The queries are also written in JSON. 

### The Elastic stack
#### Kibana
Kibana is an analytics and visualization platform, which lets you easily visualize data from Elasticsearch and analyze it to make sense of it. You can think of Kibana as a web interface of the data that is stored within Elasticsearch. 

#### Logstash
Traditionally, Logstash has been used to process logs from applications and send them to Elasticsearch. That's still a popular use case, but Logstash has evolved into a more general purpose tool, meaning that Logstash is a data processing pipeline. The data that Logstash receives will be handled as events. These events are processed by Logstash and shipped off to one or more destinations. That could be Elasticsearch, a Kafka queue, an e-mail message or an HTTP endpoint. 

#### X-Pack
X-Pack is a pack of features that adds additional functionality to Elasticsearch and Kibana. A couple of its features are:
- Adding authentication and authorization to both Kibana and Elasticsearch.
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

### Retrieving documents by ID
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
When indexing a document its full-text fields are run through an analysis process. By full-text fields we're referring to fields of the type `text` and not `keyword` fields. `keyword` fields are not analysed.

Basically it involves tokenizing text into terms, lowercasing text etc. More generally speaking, the analysis process involves tokenizing and normalizing a block of text. This is done to make the text easier to search. You have full control over the analyzer process because it's possible to control which analyzer is used. The standard analyzer is sufficient in most cases though. 

### A closer look at analyzers
An analyzer consists of three things:
- Character filters
- Token filters
- Tokenizer

An analyzer is basically a package of these building blocks with each one of them changing the input stream. So when indexing a document it goes through the following flow. 

First, zero or more character filters. A character filter receives a text fields original text and can then transform the value by adding, removing or changing characters. An example of this could be to strip out any HTML markup. 

Afterwards, a tokenizer splits the text into individual tokens, which will usually be words. An analyzer may only have 1 tokenizer. It basically splits by whitespace and also removes most symbols such as commas, periods and semicolons. That's because most symbols are not useful when it comes to searching as they are intended for being read by humans.

After splitting the text into tokens, it runs through zero or more token filters. A token filter may add, remove or change tokens. This is kind of similar to a character filter, but token filters work with the token stream instead of a character stream. There are a couple of different token filters with the simplest one being a lowercase token filter, which just converts all characters to lowercase. Another token filter that you can make use of is named "stop". It removes common words which are referred to as stop words. These are words such as "the", "a", "and", "at", etc. These are words that don't really provide any value to a field, in terms of searchability, because each word gives a document very little significance in terms of relevance. Another token filter worth mentioning is one named "synonym", which is useful for giving similar words the same meaning. 

## 6. Introduction to searching
When writing search queries there are two methods in which you can do this. One of them is by writing a search query within the request body. This is done by using something called the Query DSL. This is the most flexible and common way of writing search queries. 

For example:
```
GET /product/doc/_search
{
  "query": {
    "match": {
      "description": "red wine"
    }
  }
}
```

Another method of executing a search query is by using the request URI. That is, to embed the search query directly in the request URI. This is also referred to as query string queries.

For example:
```
GET /product/doc/_search?q=name:pasta
```

You can still perform quite advanced queries with this approach but it's less expressive and can quickly become difficult to read. There are also things you just can't do with the request URI approach. However, this way of searching can be very useful for running quick searches, perhaps for debugging from the command line or while developing. 

### Introducing the Query DSL
There are two main groups of queries in the Query DSL: leaf queries and compound queries. The basic idea is that leaf queries search for values in particular fields whereas compound queries consist of multiple leaf queries, or compound queries themselves. Compound queries are therefor recursive in nature. 

A simple leaf query can look like this:
```
GET products/doc/_search
{
  "query": {
    "match_all": {}
  }
}
```

In this case `match_all` is the name of the query. As it implies, it matches all documents.

### How searching works
You will have some kind of client, which will often be a server. That client communicates with Elasticsearch by sending search queries over HTTP. The cluster then does its magic based on the index and query you have specified in the HTTP request. When the results are ready, the cluster responds with the results, and the client can use the results for whatever purpose. 

Now we'll go in a little more depth. A search query hits a given node in a cluster. This node becomes the coordinating node for the query. It broadcasts the request to all shards in the index that the query refers to. This can be both primary shards and replica shards. These shards then respond with the results, and the coordinating node will merge all of the results together into a single result, which is then sorted and returned to the client. 

### Understanding query results
This is what an Elasticsearch result can look like:
```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1011,
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_type": "doc",
        "_id": "14",
        "_score": 1,
        "_source": {
          "name": "Nori Sea Weed - Gold Label",
          "price": 177,
          "in_stock": 21,
          "sold": 411,
          "tags": [],
          "description": "Nulla ac enim. In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy. Integer non velit. Donec diam neque, vestibulum eget, vulputate ut, ultrices vel, augue. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi. Integer ac neque. Duis bibendum.",
          "is_active": true,
          "created": "2007/02/16"
        }
      }
    ]
  }
}
```

Now we'll go through the returned properties:
- `took`: an integer representing the number of milliseconds the query took to execute.
- `timed_out`: a flag indicating whether or not the search request timed out.
- `shards`: this object contains the total number of shards that were searched and the number of shards that completed successfully or failed. 
- `hits`: contains the search results.
  - `total`: total number of documents that match the search criteria.
  - `max_score`: contains the highest score for any of the matched documents. By default, matched are sorted by their relevant score.
  - `hits`: an array containing the matched documents. By default, the first 10 documents are returned. 
    - `_score`: each matching document has a `_score` property which is a number how well the document matched the search query.  

### Understanding relevance scores
Until fairly recently Elasticsearch has made use of an algorithm named TF/IDF, short for Term Frequency / Inverse Document Frequency. Now an algorithm named Okapi BM25 is used. This algorithm does share many similarities with the older one. We'll discuss how the old algorithm works and then show the differences that PM25 adds. 

First, we have something called a Term Frequency. It looks at how many times a given term appears in the field that we're searching for in particular documents. The more times the term appears the more relevant the document is, at least for that term. 

The second factor is the Inverse Document Frequency (IDF). This refers to how often a term appears within the index. The logic here is that if a term appears in many documents, it has a lower weight. This means that words that appear many times are less significant, such as words like "the", "this", "if", etc. So if a document contains a term and it's not a common term for the fields, then this is a signal that the document is relevant.

The third factor is called Field-length norm. This simply refers to how long the field is. The longer the field, the less likely to words within the field are to be relevant. For example, the term "salad" in a title of 50 characters is more significant than in a 5000 character description. Therefor a term appearing in a short field has more weight than in a long field. 

The Term Frequency, Inverse Document Frequency and Field-length norm are calculated and stored at index time, i.e. when a document is added or updated. These stored values are then used to calculate the weight of a given term for a particular document. Individual queries may include other factors for calculating the score of a match such as the term proximity or fuzziness for accounting for typos. 

Those were the basics of the TF/IDF algorithm. Now let's look at how the BM25 algorithm compares.

It used to be common practice to remove stop words when analyzing text fields with the reason being that they didn't provide any clues for calculating the relevance anyway. That has since changed, because although the value of stop words are limited, they do have some value. It's therefor no longer very common to remove stop words, which is also why you see the stop token filter for being disabled by default for the standard analyzer. The relevance algorithm then needs to handle this because otherwise we would see the weight of stop words being boosted, artificially for large fields that contain many of them. With the TF/IDF this would lead to the stop words being boosted more than they should because they occur so many times. 

BM25 solves this by using something called Nonlinear Term Frequency Saturation. The idea is that BM25 has an upper limit for how much a term can be boosted based on how many times it occurs. If a term appears five to ten times it has a significantly larger impact on the relevance than if it just occurred once or twice. But as the number of occurrences increase, the relevance boost quickly becomes less significant, meaning that the boost for a term that appears 1000 times will almost be the same as if it occurred 30 times, for example. 

BM25 also improves the field-length norm. Instead of treating a field in the same way across all documents, the BM25 algorithm considers each field separately. It does this by taking the average field length into account, meaning that they can distinguish a short title fields from a long title field for instance.

### Query contexts
A query can be executed in two different contexts: in a query context or a filter context.

When used in a query context you're essentially asking Elasticsearch to question "how well do documents match this query?". Elasticsearch will still decide whether or not documents match in the first place, but a relevance score will also be calculated. 

When adding a query clause within a filter context we ask Elasticsearch "do documents match this query clause?". I.e. documents that do not match to the query clause will not be part of the results. 

The difference between the to is that with the filter context there is no relevance score calculated. That's because it's a boolean evaluation because either a document matches or it doesn't. 

### Full text queries vs term level queries
Term level queries search for exact values and are not analyzed. Full text queries are analyzed using the same analyzer that's defined for the field that is being searched. 

Therefor full text queries are most of the time case independent because of the analyzer that runs the query. 

Term queries are therefor better suited for matching enum values, numbers, dates, etc and not sentences.

## 7. Term level queries
### Introduction to term level queries
These are most commonly used for querying structured data such as dates and numbers. You can use term level queries for text fields too, for example you can search a "status" field for the term "active". Term level queries find exact matches, so this wouldn't be useful for searching a description for a certain text. 

### Searching for a term
Let's say we want to search for all documents with the `is_active` field set to `true`. That works this way:
```
GET products/doc/_search
{
  "query": {
    "term": {
      "is_active": {
        "value": true
      }
    }
  }
}
``` 

If you want to search a full text for for example an enum, you would use the query for fields of the type `keyword`, since these are not analyzed. 

### Searching for multiple terms
Now we search for multiple terms instead of one. The documents will match if it contains any of the supplied values within the field that we specify. 

```
GET products/doc/_search
{
  "query": {
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}
```

### Matching documents with range values
If, for example, you can to search for products that are almost out of stock you can use the `range` query:

```
GET products/doc/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 1,
        "lte": 5
      }
    }
  }
}
```

You can also use the `range` query with dates. Say that you want to find all products created in the year 2010:

```
GET products/doc/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
  }
}
```

### Searching with wildcards
You can make use of two characters in the term query, the * and ?. 
- \* - matches any character sequence including no characters
- \? - matches any single character

Example, we want to search for the term Vegetable, but we replace some characters with the * character:

```
GET products/doc/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg*ble"
    }
  }
}
```

### Searching with regular expressions
```
GET products/doc/_search
{
  "query": {
    "regexp": {
      "tags.keyword": "Veget[a-zA-Z]+ble"
    }
  }
}
```

## 8. Full text queries
The first query we'll look at is the `match` query. It's a powerful and simple query but it allows for very flexible matching. It's very useful for queries that users enter, such as on Google. 

```
GET /recipe/default/_search
{
  "query": {
    "match": {
      "title": "Recipes with pasta or spaghetti"
    }
  }
}
```

With the query above, it uses the analyzer to match documents relevant to the entered query. Tough, the order of the words that are entered are not relevant. If you want to make it relevant, you should use the `match_phrase` query:

```
GET /recipe/default/_search
{
  "query": {
    "match_phrase": {
      "title": "Spaghetti puttanesca"
    }
  }
}
```

This will only return documents with a title containing those two words in the exact order.

If you want to search multiple fields with a single query, you should use the `multi_match` query.

```
GET /recipe/default/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": ["title", "description"]
    }
  }
}
```

## 9. Adding boolean logic to queries

## 10. Joining queries

## 11. Controlling query results

## 12. Aggregations
Aggregation is a way of grouping and extracting statistics and summaries from your data. Suppose you have and index where each document corresponds to an order and each order contains an amount and a product ID. A simple example of what you can do with aggregations is to group orders by product ID and then sum the amounts to see how much each product was sold for.



## 13. Improving search results

## 14. Building a web application search engine