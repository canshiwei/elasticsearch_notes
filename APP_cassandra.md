# Cassandra: a decentralized structured storage system

## [Data Model](https://www.tutorialspoint.com/cassandra/cassandra_data_model.htm)

* A table in Cassandra is a distributed multi dimensional map indexed by a key
* Columns are grouped together into sets called **column families**
    * Simple column families
    * Super column families: Can be visualized as a column family within a column family
    * Any column within a column family is access using the convention *column_family: column*
    * Any column within a column family that is of type super is accessed using the convention *column_family: super_column: column*

## System Architecture

* Consistent hashing
    * http://tom-e-white.com/2007/11/consistent-hashing.html
    * https://zhuanlan.zhihu.com/p/34985026
* Partition & Replication
    * https://www.instaclustr.com/cassandra-data-partitioning/
    * https://shermandigital.com/blog/designing-a-cassandra-data-model/
* Failure detection
    * [Gossip](https://en.wikipedia.org/wiki/Gossip_protocol)
* Bootstrapping
    * When a node starts for the first time, it chooses a random token for its posision in the ring
    * The token information is then gossiped around the cluster