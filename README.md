# List concept keywords
- node
- shards
- index
- cluster
- master: hieu^? nhu* primary key member
- replica: hieu^? nhu* copies tu master
- query dsl (11:59): flexible, powerful query language


# How to config & use?

## Create Index
To add data, need an **index** (1 or n *shards*)

A *shard* can be: **primary** or **replica**
```
curl -XPUT 'https://localhost:9200/logs'
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}
```

## Single node cluster
one **node** with three primary shards

creates a **cluster** of one node

node is elected to **master** role within the cluster **replica** shards not allocated

## Add repliency
2nd **node** started with same **cluster.name**

node joins cluster (**discovery unicast/multicast)

**repica** shards **automatically** allocated to 2nd node

## Scale Horizontally
add another node

ELS **automatically** balances data

## Scaling out more (number_of_replicas: n) 7:32

number of **primary** shard **fixed** at index creation

can **dynamically** increase the number of **replica** shards

more copies of your data means higher **read** throughput

## Coping with failure 8:50
**previous** master node fails

triggers a new **master node election**

new master instantly **promotes** replicas to primary

## Distributed 10:18
- **Replication**: Data duplication
    + read scalability
    + high-availability
- **Sharding**: Data partitioning
    + split logical data over several machines
    + write scalebility
    + control data flow

# Some specific advantages

## query dsl 12:05
flexible, powerful query language

### queries
- relevance
- full text
- not cached
- slower

### filters
- boolean yes/no
- exact values
- cached
- faster

> Filter first, then query remaining docs

### basic query 13:30
```
GET /_search
{
    "query": {...}
}
```

Example:
```
GET /_search
{
    "query": {"match": {"title":"search"}}
}
```

### filtered query 13:35
```
GET /_search
{
    "query": {
        "filtered": {
            "query": {...},
            "filter": {...}
        }
    }
}
```

Example:
```
GET /_search
{
    "query": {
        "filtered": {
            "query": {"match":{"title":"search"}},
            "filter": {"term":{"status":"active"}}
        }
    }
}
```

### other filter types 13:52

#### WHERE field CONTAINS "value"
**term filter**
```
"term": {
    "title": "brown"
}
```

#### WHERE field IN ["val", ...]
**terms fitler**
```
"terms": {
    "title": ["quirk", "pets"]
}
```

#### WHERE field >= x AND field < y  14:25
**range filter**
```
"range": {
    "content": {
        "gte": 10,
        "lt": 80
    }
}
```

Example for date:
```
"range": {
    "date": {
        "gte": "2014-01-01",
        "lt": "2041-02-01"
    }
}
```

### boolean filter types 14:54
```
"bool": {
    "must": [<filters>],     AND
    "should": [<filters],    OR
    "must_not": [<filters>]  NOT
}
```

### query dsl: full example 16:35
```
{
    "filtered": {
        "query": {"match": {"title":"full text search"}},
        "filter": {
            "bool": {
                "must": {"range":{"created":{"gte":"now - 1d/d"}}},
                "should": [
                    {"term": {"featured":true}},
                    {"term": {"started":true}}
                ],
                "must_not": {"term":{"deleted":false}}
            }
        }
    }
}
```

# Analytics (aggregations dsl) 18:00
## Types of Aggregations
### Buckets
- Terms
- Date histogram
- Filter
- Range
- Nested
- Children
- ...

### Metrics
- Stats
- Percentile
- Cardinality
- Top hits
- Scripted
- Max | Min | Avg
- ...

## How do aggs work? 19:28
- 'inline' with search query
- execute in isolation on each shard
- 4 phases
    + parse
    + collect
    + combine
    + reduce

### Phase 1: Parse 20:10
- Coordinating node splits the request into shard request
- shards parse aggregation and initialize data structures

### Phase 2 + 3: Collect & Combine  20:50
- shards process all matching documents
- once done, they combine the aggregated data into an aggregation

## Aggregation DSL Example 21:25
### Request
```
"aggs": {
    "by_date":{
        "date_historgram":{
            "field":"timestamp",
            "interval":"day"
        },
        "aggs":{
            "max_temperature":{
                "max":{
                    "field":"temperature"
                }
            }
        }
    }
}
...
```

### Response
```
"aggregation":{
    "by_date":{
        "buckets":[
        {
            "key":"2015-01-01T00:00:00.000Z",
            "doc_count":24,
            "max_temperature":{
                "value":23
            }
        }]
    }
}
...
```


## Kibana 4: with full aggregation engine 23:00

# Designed for speed and scale 23:40
- Single network round-trip
- Single pass through the data on shards
- Aggregates are computed **in-memory**
- Trades accuracy for speed in some use cases
- Aggregations can be **composed**
- Near real-time response times

# Q/A time 25:30
