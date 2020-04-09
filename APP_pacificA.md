# PacificA: Replication in Log-Based Distributed Storage Systems

## Introduction

* PacificA
    * A prototype of a distributed log-based system for storing structured and semi-structured web data

* Three principles through the replication framwork
    * The separation of replica group configuration management from data replication, which Paxos for managing configurations and primary/backup
    * Decentralized monitoring for detecting failures and triggering reconfiguration, with monitoring traffic follows the communication patterns of data replication
    * A general and abstract model that clarifies correctness and allows different practical instantiations

## Pacifica Replication Framework

### Assumptions

* Replication protocols of the system achieve **strong consistency**
    * Strong consistency ensures that replicated system behaves the same as its non-replicated counterpart that achieves semantics such as linearizability
    * Strong consistency can be achieved if all servers in a replica group process **the same set of requests** in the **same order**
* Servers might fail
    * Assume fail-stop failures
    * Messages could be dropped or re-orderded, but not injected or modified
    * Network partitions could also occur
    * Clocks on servers are not necessarily sunchronized or even loosely sunchronized, but clock drifts are assumed to be bounded
* System maintiains a large set of data
    * Each piece of data is replicated on a set of servers, referred to as the **replica group**
    * The data replication protocol follows the primary backup paradigm
    * One replica in replica group is designated as **primary**, while others are called the **secondaries**
    * The **Configuration** for a replica group changes due to replica failures or additions
    * **Versions** are used to track the change in configuration

### Primary/Backup Data Replication

* Primary/backup paradigm
    * All requests are sent to the primary
    * The primary processes all queries locally and involves all secondaries in processing updates
* Model
    * Nodes maintains a **prepared list** of requests and a **committed point** to the list
        * **Only one committed point may applied to ensure the order of modification of data (data + 1 * 2 != data * 2 + 1)**
    * The prefix of the prepared list up to the committed point is referred to as the **committed list**
* Normal-Case Query
    * Primary receives a query, it processes the query against the state represented by the current committed list and returns the response immediately
* Normal-Case Update
    * Primary p assigns the **next available serial number** to the request
    * Primary p sends request with its **configuration versoin** and the **serial number** in a prepare message to all replicas
    * Replica r receives the prepare message then inserts the request to is prepared list in the serial number order
        * Then the request is considered **prepared** on the replica
    * Replica sends an acknowledgment in a prepared message to the primary
    * Primary view a request is **committed** after it receives acknowledgments from **all** replicas
        * Then primary moves its committed point forward to the highest point
    * Primary then sends the acknowledgement to the client indicating a successful completion
    * Replica further move its committed point accordingly
* **Commit Invariant**
    * Let p be the primary and q be any replica in the current configuration, committed_q \subseteq committed_p \subseteq prepared_q holds