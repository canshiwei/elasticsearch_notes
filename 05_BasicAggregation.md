# Basic Aggregation

## Type of Aggregation
* Bucket Aggregation
    * A set of document collection fits specific requirment
    * Split into bucket according to `terms`
    * Example:
        * `flight_dest` is the name of this aggregation

        ```
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
        ```

* Metric Aggregation
    * Mathematics and statistical computation
    * min/max/sum/avg/cardinality
    * Example show based on aggregation of bucket, do aggregation on each bucket documents (nested aggregation)

    ```
    GET kibana_sample_data_flights/_search
    {
        "size": 0,
        "aggs":{
            "flight_dest":{
                "terms":{
                    "field":"DestCountry"
                },
                "aggs":{
                    "avg_price":{
                        "avg":{
                            "field":"AvgTicketPrice"
                        }
                    },
                    "max_price":{
                        "max":{
                            "field":"AvgTicketPrice"
                        }
                    },
                    "min_price":{
                        "min":{
                            "field":"AvgTicketPrice"
                        }
                    }
                }
            }
        }
    }
    ```
* Pipeline Aggregation
    * The second aggregation for an aggregation result
* Matrix Aggregation
    * Matrix operation
* SQL vs. Bucket & Metric

```
SELECT COUNT(brand)  <---- Metric: Mathematics and statistical computation

FROM cars

GROUP by brand      <---- Bucket: A set of document collection fits specific requirment
```