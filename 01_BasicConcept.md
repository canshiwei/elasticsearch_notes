# Basic Concept

## Overview

* Elasticsearch is developed in Java
* Elasticsearch provides real-time search and analytics for all types of data
* Use Apache Lucene as search framwork

## Key Concept

## Document:

* The smallest unit for searching
  * Log
    * Meta data for movie
    * PDF/Music
* Using JSON format
  * Document is a group of fields.
    * Field is the smallest unit of data in elasticsearch. It can contain a list of multiple values of the same type
  * Every field has a type
  * ElasticSearch is able to infer the type of field
* Every document as unique ID
  * Specified by user
  * Generate by system
* MetaData for Document
  * _index: logical namespace of one or multiple physical sharding.
  * _type: type of the document
  * _id: unique id for document
  * _source: original JSON data
  * _version: version of the document.
  * _score: searching related score
* Example: 
    ```
    {
      "name": "Avengers: Endgame"
      "producer"‎: "‎Kevin Feige"	
      "company"‎: ‎"Marvel Studios"
    }
    ```

    is stored as

    ```
    {
      "_index": "movies",
      "_type": "_meta",
      "_id": "123",
      "_version": 1,
      "_seq_no": 0,
      "_primary_term": 1,
      "_source": {
        "name": "Avengers: Endgame"
        "producer"‎: "‎Kevin Feige"	
        "company"‎: ‎"Marvel Studios"
      }
    }
    ```

### Index
* Index: Container of the documents
  * Index represent logical namespace: every index has their defination of mapping, used to define the field and field type
  * Shard represent physical namespace: data of index are distributed to different shard
* Index Mapping and setting:
  * Mapping: define document field
  * Setting: define data distribution
* Example: 

  ```
  {
    "movies": {
      "settings": {
        "index": {
          "creation_date": "1552737458545",
          "number_of_replicas": "0",
          "uuid": "Qn13esMNposKDF",
          "version": {
            "created": "6060299"
          },
          "provided_name": "movies"
        }
      }
    }
  }
  ```
  

### Shard
* Shard is basic r/w unit, enables parallel r/w opration
* Two types of data: 
  * primary shard
  * replica shard
  * Elasticsearch ensures that primary and the replica of the same shard will not collocate in the same node
  ![](./img/shard.png)
* Relationship between index and shard
  * Every index contains multiple shards
  * Every shards is a lucene index
  * Every lucene index contains multiple segment, using inverted indexing
  * Every segment composed of multiple field
  * Every field associate with multiple term
  ![](./img/index_shard.png)
* Setting of index sharding:
  * Primary shards cannot be modified after created
  ```
  PUT /blogs
  {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1
    }
  }
  ```
* Example: shards stored on nodes
  ![](./img/shardeg.png)
    

### Cluster
![](./img/cluster.png)
* Distributed cluster: 
  * Master-Slave: 
    * ES, HDFS, HBase
  * Dynamo:
    * Cassandra
* Role of Nodes
  * **Master-eligible Node**:
    * Default as Master-eligible Node when a node starts
    * Master-eligible Node can be elected as master
    * Each Master-eligible node should know the number of Master-eligible nodes. 
    * To avoid multiple master been elected (brain split) when network partition, setting of `discovery.zen.minimum_master_nodes` as the result of `(master_eligible_nodes / 2) + 1`
  * **Master Node**:
    * Master node stores the status of cluster, only cluster node can modify the cluster status
      * Cluster status maintain the necessary informatino for cluster
        * Information of every nodes
        * Every mapping and setting information of index
        * Route table for shards
    * Unique in the cluster
  * **Data Node**: 
    * store data that contains indexed document.
  * **Ingest Node**:
    * A way to process a document in pipeline mode before indexing the document. Set `node.ingest: true` to enable ingest.
  * **Coordinating Node**:
    * If all three roles are disabled, the node will only act as a coordination node that perform routing request.
    * Nodes are coordinating nodes by default
* Node Configuration
  * Nodes can be multiple role
  * In production, it's better to have a node with only one role

  | Node role       | Setting     | Default   |
  | :-------        | :--------   | :-------  |
  | master eligible | node.master | true      |
  | data            | node.data   | true      |
  | ingest          | node.ingest | true      |
  | coordinating    | /           | true      |
  | machine learning  | node.ml   | true (enable x-pack) |
* Health of cluster
  * Green: primary shards and replica are allocated successfully
  * Yellow: primary shards are allocated successfully, but not for replica
  * Red: primary shards are not allocated successfully
  
  ```
  GET _cluster/health
  ```

  returns

  ```
  {
    "cluster_name" : "mycluster",
    "status" : "green",
    "timed_out" : false,
    "number_of_nodes" : 3,
    "number_of_data_nodes" : 3,
    "active_primary_shards" : 9,
    "active_shards" : 18,
    "relocating_shards" : 0,
    "initializing_shards" : 0,
    "unassigned_shards" : 0,
    "delayed_unassigned_shards" : 0,
    "number_of_pending_tasks" : 0,
    "number_of_in_flight_fetch" : 0,
    "task_max_waiting_in_queue_millis" : 0,
    "active_shards_percent_as_number" : 100.0
  }
  ```
  
## Basic Elasticsearch configuration
* config folder: 
  * elasticsearch.keystore
  * elasticsearch.yml: main configuration file, which contains settings of clusters, nodes, and paths
    ![](./img/esyml.png)
  * jvm.options
  * users_roles
  * roles.yml
  * role_mapping.yml
  * log4j2.properties: Uses Log4j 2 for logging
  * users


## Running Elasticsearch
 
* Run

	```
	cd elasticsearch-7.0.0
	./bin/elasticsearch
	```

	![](./img/run.png)

* Test if start successfully

	```
	curl http://localhost:9200
	```
	The response is a json objects

	```
  {
    "name" : "Canshis-MBP",
    "cluster_name" : "elasticsearch",
    "cluster_uuid" : "-qZP4yCCSgmRf9ODqOMDcw",
    "version" : {
      "number" : "7.5.1",
      "build_flavor" : "default",
      "build_type" : "tar",
      "build_hash" : "3ae9ac9a93c95bd0cdc054951cf95d88e1e18d96",
      "build_date" : "2019-12-16T22:57:37.835892Z",
      "build_snapshot" : false,
      "lucene_version" : "8.3.0",
      "minimum_wire_compatibility_version" : "6.8.0",
      "minimum_index_compatibility_version" : "6.0.0-beta1"
    },
    "tagline" : "You Know, for Search"
  }
	```

* Install plugin

  ```
  bin/elasticsearch-plugin list
  bin/elasticsearch-plugin install analysis-icu
  ```

  Check if the plugin installed successfully

  ```
  curl localhost:9200/_cat/plugins
  >> Canshis-MBP analysis-icu 7.5.1
  ```

* Run multiple nodes on a cluster

  ```
  bin/elasticsearch -E node.name=node1 -E cluster.name=mycluster -E path.data=node1_data -d
  bin/elasticsearch -E node.name=node2 -E cluster.name=mycluster -E path.data=node2_data -d
  bin/elasticsearch -E node.name=node3 -E cluster.name=mycluster -E path.data=node3_data -d
  ```

  Test if the nodes run successfully

  ```
  curl localhost:9200/_cat/nodes
  
  127.0.0.1 25 100 14 4.15   dilm - node1
  127.0.0.1 25 100 14 4.15   dilm * node2
  127.0.0.1 25 100 14 4.15   dilm - node3
  ```