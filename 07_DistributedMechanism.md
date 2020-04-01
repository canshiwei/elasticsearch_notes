# Distributed Mechanism

## Master Election
* Bully Algorithm: Choose the node with largest ID as master
* Constraint:
    * With more than **half of candidates**, node reach the quorum would be elected as **temporary master**
        * Why temporary master? Suppose 5 nodes with ID 1, 2, 3, 4, 5. When network partition happen, the list of node 1 shows only 1,2,3,4, then node 1 select 4 as master; node 2 shows 2, 3, 4, 5, then node 2 select 5. In this case brain split happen
    * With more than **half of vote number**: if node is selected as temporary master, it must check if the vote number is more than half of the total node. This constraint solve the first problem
    * When detect node **leaving event, check the current node list is more than half of total node**. If cannot reach quorum, quit master and rejoin the cluster using discovery
        * Why do this? Suppose 5 nodes, 2 nodes in a group, another 3 nodes in a group, and before network partition, master is in 2 node group. After network partition, 3 nodes group would select a new master, then brain split happen.
        * Set the total node number for node:
        ```
        discovery.zen.minimum_master_nodes = (total node) / 2 + 1
        ```

## Failover

* Suppose In cluster below, Node 1(master) fail --> Cluster Read
* Node 2 and Node 3 detect leaving event, select master again (Node 2)
* R0 in Node 3 upgrade to primary shard P0  --> Cluster yellow
* R0, R1 allocate to nodes --> Cluster green

    ```
    +----------------------------------------------+
    |Cluster:                                      |
    |                                              |
    |  Node 1(master)   Node 2        Node 3       |
    |   +--------+    +--------+    +--------+     |
    |   |   P0   |    |   P1   |    |   P2   |     |
    |   |   R1   |    |   R2   |    |   R0   |     |
    |   +--------+    +--------+    +--------+     |
    +----------------------------------------------+

    +----------------------------------------------+
    |Cluster:                                      |
    |                                              |
    |   Node 1 Dead   Node 2(master)  Node 3       |
    |   +--------+    +--------+    +--------+     |
    |   |   --   |    |   P1   |    |   P2   |     |
    |   |   --   |    |   R2   |    |  [P0]  |     |
    |   +--------+    +--------+    +--------+     |
    |                   ^-R0            ^-R1       |
    +----------------------------------------------+
    ```

## Sharding mechanism
* shard = hash(_routing) % number_of_primary_shards
    * _routing is document id
* ES is a **real time** search engine
    * **Immutable inverted index**
        * **Segment**: Use inverted index, cannot be modified
        * Multiple segment become a **shard**
        * When writing new document, new segment generated
        * **Commit point** used to record segment information
        * Deleted document will not be truly deleted, record in ".del"
    * **Refresh**
        * Refresh: write index buffer into segment
        * Refresh frequency: 1s. Can be set via `index.refresh_interval`. After refresh, data could be searched
        * Index buffer is filled, do refresh

    ```
    time
    |                         Index buffer
    |    Index Document    +----------------+
    |   ---------------->  | [doc]          |
    |                      +----------------+
    |
    |                         Index buffer
    |    Index Document    +----------------+
    |   ---------------->  | [doc] [doc]    |
    |                      +----------------+
    |
    |                         Index buffer
    |                      +----------------+           +----------------+
    |                      |                |  ------>  |  Segment       |
    |                      +----------------+           +----------------+
    |
    ^
    ```
* ES would not loss data when power off
    * **Transaction Log**
        * When writing into index buffer, write transaction log
        * ES Refresh, index buffer would be cleared, but transaction buffer would not
    
    ```
     time
    |                         Index buffer
    |    Index Document    +----------------+
    |   ---------------->  | [doc] [doc]    |
    |   |                  +----------------+
    |   |                    Transaction Log
    |   |                  +----------------+
    |   +------------->    | [log] [log] ...|  
    |                      +----------------+ 
    |
    |                         Index buffer
    |                      +----------------+           +----------------+
    |                      |                |  ------>  |  Segment       |
    |                      +----------------+           +----------------+
    |
    |                        Transaction Log
    |                      +----------------+
    |                      | [log] [log] ...|  
    |                      +----------------+ 
    ^
    ```

    * **Flush**
        * Call fsync, write segment into disk
        * Clear transaction log
        * Flush happen every 30 minute or transaction log is full (default 512 MB)

* **Merge**:
    * Too much segment, should be merge
        * decrease segment/deleted the "deleted" document
    * ES and Lucent would merge segment automatically
        * Or use force merge: POST my_index/_forcemerge