# Demo

POST _bulk
{ "create": {"_index": "test", "_id": "1"} }
{ "field1": "value3" }
{ "index": {"_index": "test", "_id": "1"} }
{ "field1": "value1" }
{ "update": {"_index": "test", "_id": "1"} }
{ "doc": { "field2": "value2" } }


GET test/_doc/1

POST test/_doc/1
{
  "field1" : "value1.1",
  "field2" : "value2.2"
}

DELETE test/_doc/1

GET _mget
{
  "docs": [
    {
      "_index": "my_index",
      "_id": "1"
    },
    
    {
      "_index": "my_index",
      "_id": "2"
    }
  ]
}


POST kibana_sample_data_ecommerce/_search
{
  "_source":["order_date", "nonexist_field"],
  "query":{
    "match_all": {}
  }
}

GET movies/_search

GET movies/_search
{
  "script_fields": {
    "my_new_field": {
      "script": {
        "source": "doc['year'].value + ' is good'"
      }
    }
  }
}

POST movies/_search
{
  "query": {
    "match": {
      "title": "last christmas"
    }
  }
}

POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "last christmas",
        "operator": "and"
      }
    }
  }
}

GET _cat/indices?v

# Analyzer

GET /_analyze
{
  "analyzer": "standard",
  "text": "Mastering Elasticsearch, elasticsearch in Action 11"
}

GET /_analyze
{
  "analyzer": "simple",
  "text": "Mastering Elasticsearch, elasticsearch in Action 11"
}

POST _analyze
{
  "tokenizer": "keyword",
  "char_filter": ["html_strip"],
  "text": "<b>hello world</b>"
}

POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "mapping",
        "mappings" : [ "- => _"]
      }
    ],
  "text": "123-456, I-test! test-990 650-555-1234"
}

POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "pattern_replace",
        "pattern" : "http://(.*)",
        "replacement" : "good"
      }
    ],
    "text" : "http://www.elastic.co"
}

POST _analyze
{
"tokenizer":"path_hierarchy",
"text":"/user/canshi/a/b/c/d/e"
}

POST _analyze
{
  "tokenizer": "uax_url_email",
  "text": ["weic2@rpi.edumy email"]
}

GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["stop", "lowercase"],
  "text": ["The rain in Spain falls mainly on the plain."]
}

DELETE my_analyzer

POST my_analyzer/_analyze
{
"analyzer": "my_custom_analyzer",
"text": "Is this <b>déjà vu</b>?"
}

PUT my_analyzer 
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
            "type": "custom",
            "tokenizer": "standard",
            "char_filter": ["html_strip"],
            "filter": ["lowercase", "asciifolding"]
          }
      }
    }
  }
}

POST my_analyzer/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I am :(.Are you?"
}

PUT my_analyzer 
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
            "type": "custom",
            "tokenizer": "punctuation",
            "char_filter": ["emoticons"],
            "filter": ["lowercase", "english_stop"]
          }
      },
          
      "tokenizer": {
          "punctuation": { 
          "type": "pattern",
          "pattern": "[.?]"
          }
      },
          
      "char_filter": {
        "emoticons": { 
        "type": "mapping",
        "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      },
        
      "filter": {
        "english_stop": { 
          "type": "stop",
          "stopwords": "_english_"
        }
      }
    }
  }
}


# Mapping
# Dynamic == true: Mapping would be renewed
# Dynamic == flase: Mapping cannot be renewed, the newly added field cannot be used for search
# Dynamic == strict: Fail to write document

DELETE mapping_test

PUT dynamic_mapping_test/_mapping
{
  "dynamic": true
}

PUT mapping_test/_doc/1
{
  "firstName":"Chan",
  "lastName": "Jackie",
  "loginDate":"2018-07-24T10:29:48.103Z"
}

GET mapping_test/_doc/1

GET mapping_test/_mapping

PUT mapping_test/_mapping
{
  "dynamic": false
}

PUT mapping_test/_doc/1
{
  "newField":"someValue"
}

POST mapping_test/_search
{
  "query":{
    "match":{
      "newField":"someValue"
    }
  }
}

DELETE mapping_test

PUT mapping_test/_mapping
{
  
    "dynamic": true,
    "properties": {
      "firstName":{
        "type": "text"
      },
      "lastName":{
        "type": "text"
      },
      "mobile": {
        "type": "text",
        "index": false
      }
    }
}

PUT mapping_test/_doc/1
{
  "firstName":"Hello",
  "lastName": "World",
  "mobile": "110"
}

POST mapping_test/_search
{
  "query":{
    "match": {
      "mobile": "110"
    }
  }
}

# Index Template

PUT /_template/template_test
{
  "index_patterns": ["test*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 2
  },
  "mappings": {
    "date_detection": false,
    "numeric_detection": true
  }
}

DELETE testtemplate
PUT testtemplate/_doc/1
{
  "number": "1",
  "date": "2019/01/01"
}

GET testtemplate/_settings
GET testtemplate/_mappings


# Aggregation
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs":{
  "flight_dest":{
    "terms":{
        "field":"DestCountry"
      }
    }
  }
}


GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs":{
    "flight_dest":{
      "terms":{
        "field":"DestCountry"
      },
      "aggs":{
        "stats_price":{
          "stats":{
          "field":"AvgTicketPrice"
          }
        },
        
        "wather":{
          "terms": {
          "field": "DestWeather",
          "size": 5
          }
        }
      }
    }
  }
}

POST /products/_bulk
    { "index": { "_id": 1 }}
    { "productID" : "XHDK-A-1293-#fJ3","desc":"iPhone" }
    { "index": { "_id": 2 }}
    { "productID" : "KDKE-B-9947-#kL5","desc":"iPad" }
    { "index": { "_id": 3 }}
    { "productID" : "JODL-X-1937-#pV7","desc":"MBP" }
    
GET /products/_mapping

POST /products/_search
{
  "query": {
    "term": {
      "desc.keyword": {
        "value": "iPhone"
      }
    }
  }
}

POST products/_search
{
  "query":{
    "term": {
      "desc": "iPhone"
    }
  }
}

POST products/_search
{
  "query": {
    "match": {
        "desc": "iPhone"
    }
  }
}


PUT products2/_doc/1
{
  "title": "apple ipad",
  "content": "apple ipad, apple ipad"
}

PUT products2/_doc/2
{
  "title": "apple ipad, apple ipad",
  "content": "apple ipad"
}

GET products2/_search
{
  "query":{
    "bool":{
      "should": [
        {
          "match": {
            "title": {
              "query": "apple ipad",
              "boost": 1
            } 
          }
        },
        
        {
          "match": {
            "content": {
              "query": "apple ipad",
              "boost": 100
            } 
          }
        }
      ]
    }
  }
}

PUT products2/_doc/3
{
  "title": "brown rabbit",
  "content": "brown rabbit"
}


PUT products2/_doc/4
{
  "title": "brown fox",
  "content": "good good study"
}

GET products2/_search
{
  "query": {
    "bool": {
      "should": [
        {"match": {"title": "brown fox"}},
        {"match": {"content": "brown fox"}}
      ]
    }
  }
}

GET products2/_search
{
  "query": {
    "dis_max": {
      "queries": [
        {"match": {"title": "brown fox"}},
        {"match": {"content": "brown fox"}}
      ]
    }
  }
}

POST address/_doc/1
{
        "street": "5 Poland Street",
        "city": "London",
        "country": "United Kingdom",
        "postcode": "W1V 3DG"
}

POST address/_search
{
  "query": {
      "multi_match": {
          "query": "Poland Street W1V",
          "type": "cross_fields",
          "operator": "and",
          "fields": ["street", "city", "country", "postcode", "other"]
      }
  }
}