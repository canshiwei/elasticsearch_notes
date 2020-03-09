# Advanced Query

## Term Query vs. Full text Query

* Term Query:
    * Samllest unit in semantic
    * **No analysis** on Term query

        ```
        # post a bulk of index

        POST /products/_bulk
        { "index": { "_id": 1 }}
        { "productID" : "XHDK-A-1293-#fJ3","desc":"iPhone" }
        { "index": { "_id": 2 }}
        { "productID" : "KDKE-B-9947-#kL5","desc":"iPad" }
        { "index": { "_id": 3 }}
        { "productID" : "JODL-X-1937-#pV7","desc":"MBP" }

        # query has no match 
        POST /products/_search
        {
        "query": {
            "term": {
                "desc": {
                    "value": "iPhone"
                }
            }
        }
        }
        ```

        Because ES use `standard analyzer` for all text analysis,
        <br>
        Solution 1: use analyzed form of term
        ```
        POST /products/_search
        {
        "query": {
            "term": {
                "desc": {
                    "value": "iphone"
                }
            }
        }
        }
        ```

        Solution 2: extract the keyword instead of the term been analyized

        ```
        POST /products/_search
        {
        "query": {
            "term": {
            "desc.keyword": {
                "value":"iPhone"
            }
            }
        }
        }
        ```

        Solution 3: use text search

        ```
        POST products/_search
        {
        "query": {
            "match": {
                "desc": "iPhone"
            }
        }
        }
        ```

    * Calculate relativity score on each documents
    * Use **Constant Score** change query to a filtering, avoid relativity score calculation, use cache to improve performance
        * Avoid TF-IDF 
        ```
        POST products/_search
        {
            "explain": true,
            "query": {
                "constant_score": {
                    "filter": {
                        "term": {
                            "productID.keyword": "XHDK-A-1293-#fJ3"
                        }
                    }
                }
            }
        }
        ```


* Full Text Query
    * Match Query/ Match Phrase Query / Query String Query
        ```
        POST groups/_search
        {
            "query": {
                "match": {
                    "names": {
                        "query": "Water Water",
                        "operator": "OR"
                    }
                }
            }
        }


        POST groups/_search
        {
            "query": {
                "match_phrase": {
                    "names": "Water Smith"
                }
            }
        }
        ```

## Information Retrieval

### TF
* TF - Term frequency
    * The frequency of a term appear in a document
    * TF = time appear / total document terms
    * Example: "Blockchain is good"
        * TF = TF(Blockchain) + TF(is) + TF(good)
* Stop Word
    * Words like "is" has no contribution to relevance of document

## IDF
* DF - Document frequency
    * The term frenquency in all documents
* IDF - Inverse Document Frequency
    * IDF = log (total document count/ number of document the term appear)
* TF-IDF:
    * Example: "Blockchain is good" ("is" is a stop word, eliminated in this example)
        * TF-IDF =  TF(Blockchain) * IDF(Blockchain) +  TF(good) * IDF(good)

## Lucene TF-IDF

![](./img/lucenetfidf.png)

## BM 25

![](./img/bm25.png)


## Query Context & Filter Context
Query Context: Compute relativity score <br>
Filter Context: No relativity score, better peformance with cache 

### Bool Query
* must: must match, compute relativity score
* should: selective match, array type, compute relativity score
* must_not: **Filter context**, no relativity score
* filter: **Filter context**, no relativity score
* Example

    ```
    POST /products/_search
    {
        "query": {
            "bool": {
                "must": {
                    "term": {"price": 30}
                },
                "filter": {
                    "term": {"avaliable": "true"}
                },
                "must_not": {
                    "range": {"price": {"lte": 10}}
                },
                "should": [
                    {"term": {"productID.keyword": "JODL-X-1937"}}
                    {"term": {"productID.keyword": "JODL-X-1937"}}
                ],
                "minimum_should_match": 1
            }
        }
    }
    ```

* Use bool query to sovle "equal problem"
    * **Term Query - include but not equal**
            
            ```
            POST movies/_search
            {
                "query": {
                    "constant_score": {
                        "filter": {
                            "term": {
                                "genre.keyword": "Comedy"
                            }
                        }
                    }
                }
            }
            ```

            return

            ```
            {
                "_index": "movies",
                "_type": "_doc",
                ...
                "genre": [
                    "Comedy"
                    "Romance"
                ]
            }
            ```
    * In business overview, add "count" to solve the issue

* Different query structure influence computing score

```
POST /animals/_search
{
    "query": {
        "bool": {
            "should": [
                {"term": {"text": "brown"}},
                {"term": {"text": "red"}},
                {"term": {"text": "quick"}},
                {"term": {"text": "dog"}}
            ]
        }
    }
}
```

```
POST /animals/_search
{
    "query": {
        "bool": {
            "should": [
                {"term": {"text": "brown"}},
                {"term": {"text": "red"}},
                {"term": {"text": "quick"}},
                {"bool": {
                    "should": [
                        {"term": {"text": "brown"}},
                        {"term": {"text": "red"}},
                    ]
                }}
            ]
        }
    }
}
```

## Boosting: Control weight when computing relativity score

* Example: weight on title

```
GET products2/_search
{
  "query":{
    "bool":{
      "should": [
        {
          "match": {
            "title": {
              "query": "apple ipad",
              "boost": 100
            } 
          }
        },
        
        {
          "match": {
            "content": {
              "query": "apple ipad",
              "boost": 1
            } 
          }
        }
      ]
    }
  }
}
```

returns 

```
...
"hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 46.263073,
    "hits" : [
      {
        "_index" : "products2",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 46.263073,
        "_source" : {
          "title" : "apple ipad, apple ipad",
          "content" : "apple ipad"
        }
      },
      {
        "_index" : "products2",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 42.680244,
        "_source" : {
          "title" : "apple ipad",
          "content" : "apple ipad, apple ipad"
        }
      }
    ]
```

* Example: weight on content

```
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
```

returns

```
...

"hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 46.263073,
    "hits" : [
      {
        "_index" : "products2",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 46.263073,
        "_source" : {
          "title" : "apple ipad",
          "content" : "apple ipad, apple ipad"
        }
      },
      {
        "_index" : "products2",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 42.680244,
        "_source" : {
          "title" : "apple ipad, apple ipad",
          "content" : "apple ipad"
        }
      }
    ]
  }
```

### Boosting Query
* Example

```
POST news/_search
{
    "query": {
        "boosting": {
            "positive": {
                "match": {
                    "content": "apple"
                }
            },
            "negative": {
                "match": {
                    "content": "pie"
                }
            }
            "negative_boost": 0.5
        }
    }
}
```

## Disjunction Match
* when "should" query computing score: average score on each query 
* when Disjunction Match computing score: the highest score in all query

```
POST blogs/_search
{
    "query": {
        "dis_max": {
            "queries": [
                {"match": {"title": "Quick fox"}},
                {"match": {"body": "Quick fox"}}
            ]
        }
    }
}
```

### Tie Breaker
* "tie_breaker": 0 ~ 1 float number
* algorithm:
    * Get the highest score in "queries"
    * Multiply "tie_breaker" with other query except the highest one
    * Add result together then normalize
```
POST blogs/_search
{
    "query": {
        "dis_max": {
            "queries": [
                {"match": {"title": "Quick fox"}},
                {"match": {"body": "Quick fox"}}
            ],
            "tie_breaker": 0.2
        }
    }
}
```

## Multi Match Query

* Best Fields

    ```
    POST blogs/_search
    {
        "query": {
            "multi_match": {
                "type": "best_fields",
                "query": "Quick pets",
                "fields": ["title", "body"],
                "tie_breaker": 0.2,
                "minimum_should_match": "20%"
            }
        }
    }
    ```

* Most Fields
    * When use Englihs analyzer, fields with "-ing" would be convert to normal case. More field match the better it is.
    * Example:
        ```
        PUT titles
        {
            "mappings": {
                "properties": {
                    "title": {
                        "type": "text",
                        "analyzer": "english"
                    }
                }
            }
        }
        ```

        improve:

        ```
        PUT titles
        {
            "mappings": {
                "properties": {
                    "title": {
                        "type": "text",
                        "analyzer": "english",
                        "fields": {"std": {"type": "text", "analyzer": "standard"}}
                    }
                }
            }
        }
        ```

        Most fields

        ```
        GET titles/_search
        {
            "query": {
                "multi_match": {
                    "query": "barking dogs",
                    "type": "most_fields",
                    "fields": ["title", "title.std"]
                }
            }
        }
        ```

* Cross Field
    * Name, Address, and etc information, to find information in multiple fields.
    * If we use "most_fields" try to have **every word in query appear** in "fields", we cannot add `"operator": "and"`, use "cross_field" instead
    * Example:

    ```
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
                "fields": ["street, "city", "country", "postcode"]
            }
        }
    }
    ```