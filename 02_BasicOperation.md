# Basic Operation

## Index APIs

### Get Index Information

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

### Verbose

```
GET /_cat/master?v
```

### Header
Force the columns to appear

```
GET /_cat/nodes?h=ip,port,heapPercent,name
```

### Sort
Sort the column specified, where `desc` inver the ordering of column, `:asc` exhibits the same behavior as the defauklt sort order.

```
GET _cat/templates?v&s=order:desc, index_patterns
```

