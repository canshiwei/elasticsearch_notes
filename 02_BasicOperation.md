# Basic Operation

## Document CRUD

|  Operation    |    Command     |     Notes      |
| ------------- | -------------- | -------------- |
| Index         |  PUT my_index/_doc/1 <br> { <br> &emsp; "user": "canshi", <br> &emsp; "comment": "helloworld" <br>} |  If ID already exist, delete the current document then create a new one with +1 `_version` number. Otherwise, create a new one directly |
| Create        |  PUT my_index/_create/1 <br> { <br> &emsp; "user": "canshi", <br> &emsp; "comment": "helloworld" <br>} <br> <br> POST my_index/_doc <br> { <br> &emsp; "user": "canshi", <br> &emsp; "comment": "helloworld" <br> } <br> <br> PUT my_index/_doc/1?op_type=create <br> { <br> &emsp; "user": "canshi", <br> &emsp; "comment": "helloworld" <br>}    |  If ID already exist, then operation fail. <br> `Post` indicate ES would generate ID automatically. |
| Read          |   GET my_index/_doc/1     |       |
| Update        |   POST my_index/_update/1 <br> { <br> &emsp; "doc": { <br> &emsp; &emsp; "user": "canshi", <br> &emsp; &emsp; "comment": "helloworld" <br> &emsp; } <br>} | Document must exist for Update, only manipulate the field specified. <br> PayLoad must wrapped in `doc` |
| Delete        |   DELETE my_index/_doc/1  | Lazy Delete, version number set to 1 when delete is called. If new document with the index created, version number increase based on the version number of deleted document.  |

* note: update cannot change a field to a different format

```
PUT my_index/_create/1
{
  "name": "canshi",
  "age": 24
}

# would result failure of operation
POST my_index/_update/1
{
  "name":{
    "f": "canshi",
    "l": "wei"
  }
}

# but this is a valid update
POST my_index/_update/1
{
  "doc": {
    "name": 123
  }
}
```

## Batch write API - bulk
* Support 4 types of operation
    * Index
    * Create
    * Update
    * Delete
* For a operation, meta data of index shoule be specified on the first line, the second line specify the payload
    ```
    POST _bulk
    { "create": {"_index": "test", "_id": "1"} }
    { "field1": "value1" }
    ```
    is the same as 
    ```
    PUT test/_doc/1
    {
        "field1": "value1"
    }
    ```
* The failure of one operation does not affect the others


## Batch read API - mget
* Search on the meta data of index
* Example: 
    ```
    GET _mget
    {
    "docs": [
        {
        "_index": "my_index",
        "_id": "1"
        },
        
        {
        "_index": "my_index",
        "_id": "1"
        }
    ]
    }
    ```

## Batch search API - msearch

* TODO:
* Example:

```
POST users/_msearch
{}
{"query": {"match_all": {}}, "from": 0, "size": 10}
{}
{"query": {"match_all": {}}}
{"index": "twitter"}
{"query": {"match_all": {}}
```

## Index APIs

#### Get Index Information

```
GET movies
```

#### Get Number of Document 

```
GET movies/_count
```

#### Get Documents of Index

```
GET movies/_search
```

```
POST movies/_search
{
}
```

### Get Index infromation

* Get all indices

  ```
  GET /_cat/indices?v
  ```

* Get indices with pattern 'movies*'

  ```
  GET /_cat/indices/movies*?v
  ```

### Check specific segment

```
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt
```




## cat APIs

* [Detailed API document](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html)

#### Get Node information

```
GET _cat/nodes
```

#### Get Shards information

```
GET _cat/shards
```

### Common parameters

#### Verbose

```
GET /_cat/master?v
```

#### Header
Force the columns to appear

```
GET /_cat/nodes?h=ip,port,heapPercent,name
```

#### Sort
Sort the column specified, where `desc` inver the ordering of column, `:asc` exhibits the same behavior as the defauklt sort order.

```
GET _cat/templates?v&s=order:desc, index_patterns
```

## Common Error message

| Problem | Cause |
| ------- | ----- |
| Cannot connect | Internet error; Cluster dead |
| Connection cannot be closed | Internet error; Node error |
| 429 | Buzy cluster |
| 4xx | Request format does not qualify |
| 500 | Internal error of cluster |


## APPENDIX: LAB

```
PUT my_index2/_doc/1
{
  "name":{
    "f": "canshi",
    "l": "wei"
  }
}

PUT my_index/_doc/1?op_type=create
{
  "name": "canshi",
  "age": 23
}


PUT my_index/_create/1
{
  "name": "canshi",
  "age": 24
}

PUT my_index/_doc/1
{
  "name": "canshi",
  "age": 24
}

POST my_index/_doc
{
  "name": "canshi",
  "age": 25
}

POST my_index/_update/1
{
  "doc": {
    "birth": "19970807"
  }
}

GET my_index/_doc/1

GET my_index/_doc/_search

DELETE my_index/_doc/1

POST _bulk
{ "create": {"_index": "test", "_id": "1"} }
{ "field1": "value3" }
{ "index": {"_index": "test", "_id": "1"} }
{ "field1": "value1" }
{ "update": {"_index": "test", "_id": "1"} }
{ "doc": { "field2": "value2" } }
{ "delete": {"_index": "test", "_id": "1"} }

```