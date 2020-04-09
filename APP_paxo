# Paxos Made Simple (Leslie Lamport)

## The Problem
* Model:
    * asynchronous
    * non-Byzantine
* Requirements:
    * Only a value has been proposed is chosen
    * Only **a single value** is chosen
    * A process never learns that a value has been chosen unless it actually has been (learner can only **learn chosen value**)
* Roles (process may act as more than one agent):
    * proposers
    * acceptors
    * learners

## Choosing Value
* Why not single acceptor agent?
    * **Single point failure**
* Multiple acceptor agents:
    * The value is chosen when **a large enough set** of acceptors have accepted it.
        * How large is large enough:
            * A large enough set consist of **any majority of the agents**
            * Any **two majorities** have **at least one acceptor in common**
* Steps strengthen the requirement:
    * **P1: An acceptor must accept the first proposal that it receives**
        * Problem: single value may not be accepted by a majority of them
            ```
            P1,
            (A value is chosen only when it is accepted by a majority of acceptors) --> 
            ---------------------------------------
            (An acceptor must be allowed to accept more than one proposal)
            ```
        * what is acceptor always send and accept the proposal of itself? retry will never make any progress
            * In each round, only one proposer issue proposal?
    * **P2: If a proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v**
        * P1 and P2 ensure the Requirement 2: A single value is chosen
        * P2 prevent the chosen value been modified
    * **P2a: If a proposal with value v is chosen, then every higher-numbered proposal accepted by any acceptor has value v**
        * To be chosen, proposal must by accepted by at least one acceptor
            * Strengthen P2 to P2a
        * Thus P2a --> P2
    * **P2b: If a proposal with value v is chosen, then every higher-numbered proposal issued by any proposer has value v**
        * If a new proposer wake up and send a higher-numbered proposal with different value other than chosen "v", P1 force acceptor to accept the value, violet P2a
        * Strengthen P2a to P2b: if proposer would never send a higher-numbered proposal with different value other than chosen "v"
    * **P2b --> P2a --> P2**
    * **P2C: For any v and n, if a proposal with value va and number n is issued, then there is a set S consisting of a majority of acceptors such that either (a) no acceptor in S has accepted any proposal numbered less than n, or (b) v is the value of the highest-numbered proposal among all proposals numbered less than n accepted by the acceptors in S**
    * **P1a: An acceptor can accept a proposal numbered n iff it has not responded to a prepare request having a number greater than n**
        * P1a subsumes P1

## Algorithm
* Phase 1.
    * proposer selects a proposal number n, and sends a prepare request with number n to a **majority of acceptors**
    * If an acceptor receives a prepare request with number n greater than any prepare request to which it has already reponded, then it respond to the request with a **promise not to accept any more proposals numbered less than n**, and with the **highest-numbered proposal (if any) that it has accepted**
* Phase 2.
    * If proposer reveives a response to its prepare requests (numbered n) from a majority of acceptors, then it sends an **accept request** to each of those acceptors for a proposal numbered n with a value v, where **v is the value of the highest-numbered proposal among the responses, or is any value if the responses reported no proposals**
    * If an acceptor receives an accept request for a proposal numbered n, it accepts the proposal unless it has already reponded to a prepare request having a number greater than n

## Learning a Chosen Value
* Learner must find out that a proposal has been accepted by a majority of acceptors
* Naive approach:
    * Each acceptors response to all learners
* Better solution:
    * Acceptors could respond with their acceptances to some set of distinguished learnerm each of which can then inform all the learners when a value has been chosen

## Progress
* May be always in prepare phase
    * A distinguished proposer must be selected as the only one to try issuing proposals
    * If the distinguished proposer can communicate successfully with a majority of acceptors, and if it uses a proposal with number greater than any already used, then it will succeed in issuing a proposal that is accepted

## WARO
* **Suppose there are N nodes, N - 1 nodes dead, in this case, read operation can still be used; But if only one nodes dead, write operation cannot success**
* Read: Only need one read on a live node
* Write: Write operation only success if all nodes record the data

## Quorum
* Suppose there are N nodes, data records on W nodes success, then the renew operation success, where W + R > N and **W + R = N + 1**
* Read:
    * Read at most R times, the latest data appear
    * Read until W data are the same to make sure the data is **latest version**
* Write: 
    * **Write data on W nodes**, then write operation success


## References
[Paxos Made Simple](https://www.microsoft.com/en-us/research/uploads/prod/2016/12/paxos-simple-Copy.pdf) <br>
[Paxos in detail](https://zhuanlan.zhihu.com/p/31780743)
[Quorum](https://www.cnblogs.com/hapjin/p/5626889.html)