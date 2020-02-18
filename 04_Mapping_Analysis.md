# Mapping & Analysis

## Inverted Index
* [Inverted Index](./reference/inverted_index.pdf)

## Analysis
* What is analysis: convert document to a series of term/token using **Analyzer**
* Analyzer component:
    * Character Fileters: Deal with original context. E.g. Delete html tag
    * Tokenizer: split words using a specific rule
    * Token Filter: Delete stop words, process words, etc.

    ```
    +----------------------------------------------------+
    | Character Filters --> Tokenizer --> Token Filters  |
    +----------------------------------------------------+
    ``` 

* Type of Analyzer:
    * Tokenizer: Standard, Lower Case, divided by word
        * Token Filters: Stop word (default off)
    * Tokenizer: Simple, Lower Case, divided by non-char
    * Tokenizer: Stop analyzer, divided by non-char
        * Token Filters: stop word filter
    * Tokenizer: Whitespace, divided by white space
    * Tokenizer: Keyword, treat as a sinlge term
    * Tokenizer: Pattern, Lower Case, divided by regular expression
        * Token Filters: stop word filter
    
    
* _analyzer API
    ```
    GET /_analyze
    {
        "analyzer": "standard",
        "text": "Mastering Elasticsearch, elasticsearch in Action"
    }
    ```

    ```
    {
        "tokens" : [
            {
                "token" : "mastering",
                "start_offset" : 0,
                "end_offset" : 9,
                "type" : "<ALPHANUM>",
                "position" : 0
            },
            {
                "token" : "elasticsearch",
                "start_offset" : 10,
                "end_offset" : 23,
                "type" : "<ALPHANUM>",
                "position" : 1
            },
            ....
        ]
    }
    ```

## Mapping
* Mapping similar to schema in Database system
    * Define name of the field
    * Define type of data (string, integer, boolean)
    * Define setting for inverted index (Analyzed or not analyzed, Analyzer)
* A Mapping belong to the Type of index
    * Every document is a Type
    * Every Type contains a definition of Mapping
* Data type
    * Primitive data type:
        * Text/ Keyword
        * Date
        * Integer/ Floating
        * Boolean
        * IPv4 & IPv6
    * Complex data type:
        * Object/ nested object
    * Special type:
        * geo_point & geo_shape/ percolator

### Dynamic Mapping
* If index does not exist, create the index automatically
* Dynamic Mapping does not need to manually create Mappings
* ES can infer the type of data, but not exactly correct
* Can field of mapping be edited?
    * New Field:
        * **Dynamic == true**: Mapping would be renewed
        * **Dynamic == flase**: Mapping cannot be renewed, the newly added field cannot be used for search
        * **Dynamic == strict**: Fail to write document
    * Existed Field: Onece the data is written in, no more supported on modifying definition of field
* Example:

    ```
    # put a document into the system

    PUT dynamic_mapping_test/_doc/1
    {
        "newField":"someValue"
    }

    # defualt the dynamic mapping set as true
    # if set the dynamic mapping to false

    PUT dynamic_mapping_test/_mapping
    {
        "dynamic": false
    }

    # if update the document, the mapping still use only "newField"
    # and no "anotherField" would be included

    PUT dynamic_mapping_test/_doc/1
    {
        "anotherField":"another"
    }
    ```

### Mapping Operation

* Define a mapping

    ```
    PUT movies
    {
        "mappings": {
            # ....
        }
    }
    ```

* Field indexing setting
    * Field indexing setting defualt to be true
    * If set the index to false, then this field cannot be searched

    ```
    PUT users
    {
        "mappings": {
            "properties": {
                "firstName": {
                    "type": "text"
                },
                "lastName": {
                    "type": "text"
                },
                "mobile": {
                    "type": "text",
                    "index": false
                }
            }
        }
    }

    # The search below would fail
    POST /users/_search
    {
        "query": {
            "match": {
            "mobile":"12345678"
            }
        }
    }

    # GET users/_search?q=mobile:12345678
    ```

* Index Options
    * 4 types of index options
        * docs: record `doc id`
        * freqs: record `doc id and term frequencies`
        * positions: record `doc id/ term frequencies/ term position`
        * offsets: record `doc id/ term frequencies/ term position/ character offects`
    * Text default as positions, remaining are docs

    ```
    PUT users
    {
        "mappings": {
            "properties": {
                "firstName": {
                    "type": "text"
                },
                "lastName": {
                    "type": "text"
                },
                "mobile": {
                    "type": "text",
                    "index": false
                }
                "bio": {
                    "type": "text",
                    "index_options": "offsets"
                }
            }
        }
    }
    ```

* null_value:
    * Enable a field to be null and could be search

    ```
    PUT users
    {
        "mappings": {
            "properties": {
                "firstName": {
                    "type": "text"
                },
                "lastName": {
                    "type": "text"
                },
                "mobile": {
                    "type": "text",
                    "null_value": "NULL"
                }
            }
        }
    }
    ```

    returns:

    ```
    ...

    "_source": {
        "firstName": "canshi",
        "lastName": "wei",
        "mobile": null
    }
    ```

* copy_to
    * allows a non-existed filed to be search

    ```
    PUT users
    {
        "mappings": {
            "properties": {
                "firstName": {
                    "type": "text",
                    "copy_to": "fullName"
                },
                "lastName": {
                    "type": "text",
                    "copy_to": "fullName"
                },
            }
        }
    }

    GET users/_search?q=fullName:(Canshi Wei)
    ```

* Array
    * Not a true array, but a field contains multiple value
    * The type of the field associated with multiple value is the type of the value

    ```
    PUT users/_doc/1
    {
        "name": "onebird",
        "interests": ["reading", "music"]
    }
    ```