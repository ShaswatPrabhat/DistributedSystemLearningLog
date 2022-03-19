# Distributed Systems

## What is it ?

Systems that require more than one Computer systems to get their job done. Mission critical infra. Spread out physically.
> If possible for a system try implementing it on a single computer. Do not go to distributed route unless absolutely needed. Try everything else before distributed systems.

## Why do we need it ?

* Parallelism
* Risk tolerance
* Physical reasons - Commmunication over 2 computers in geographically different zones say.
* Security - Comunication over a narrowly defined network protocol to ensure both systems are safe.
* Isolation

## What challenges we face with them ?

* Different computers working concurrently means concurrency problems which makes Distributed computing hard.
* Multiple pieces + Network means unexpected failure patterns.
* Performance 

## Infrastructures
* Storage - Well defined and straight forward abstractions
* Communications
* Computation

Ways to understand abstractions to be able to make applications and hide distributed nature.

Build an interface that is say like file systems non distributed storage and computation systems.

## Implementations
RPC - mask the fact that we are communicating over unreliable network.


Threads - harness multicore computers, a way to simplify concurrent operations.
Concurrency controls like locks


## Performance

Scalable speed up: Problem on 2 computers should run in half the time as on one computer ( huge win if this can be achieved)

## Fault Tolerance

Multi-computer systems
Constant problems instead of seldom problems with a single PC
Abstractions should hide the failure as much as possible.

* Availability: for certain kinds of failure system will keep operating.
* Recoverability: maybe the service will stop working, but after the repair it will work as if nothing had gone wrong. Weaker requirement than availability.

Non volatile storage can store a check point or log to replay the state of the system.

NV Storage tends to be expensive to upgrade

Replication Managment of replicated copies tricky. Replicas drifing out of sync.
Managment of replicated copies

## Consistency

We have more than one copy of data floating around due to Fault Tolerance.

Different versions of the data.
Strong Consistency is an expensive spec to implement. Multiple copies and readers read all values and use most recent, but that is expensive.

Weak consistency might allow a stale read.

Replicas should have independent faiure patterns, say these should be put on differenet racks. Independent failure patterns, geographically distanced.
Other copy might take a while to update. 

Weak consistency gurarantees so that they are useful is a topic of research still.

## Map Reduce
* Google
* Building index of web is equivalent to a sort, sorting all the content of the web in a single computer might take tooooo long.
* For a while engineers at google wrote a web indexer or a search analysis tool.
* Have engineers who are not masters at distributed systems write the guts of the code and be able to run it on thousands of computer without having to worry about how to distribute this load to those thoussands of computers.
* Easy for non specialists
* Input is assumed to be divided into a lot of chunks, big files container crawled information.
* MapRedue will run the Map function on each of these input chunks.
* Map function will return another Key Value pair
* Collects all instances from all Map functions and collects them KeyWise.
* Sends this Keywise sorted data from Map to Reduce.
* Entire job is made of a bunch of Map tasks and a bunch of Reduce tasks.
* Map or Reduce functions in themselves do not know about the distributed systems.
* if the alogrithm is easily expressible by a Map and then shuffling of data by a Reduce funtion then it fits MapReduce framework.
* but if it does not then it will not be abl to use MapReduce
* hence expected Map and Reduce functions are pure functions
* Map emits are writeen to Map worker's local disk.
* Reduce will collect all data related to its key, gets this data from Network file system.
* GFS is used for these purposes.
* Spreads a clustered filesystem in 64MB chunks.
* Map worker normally reads the file that it takes in input over network.
* Most constraining part at the time of the paper was network throughput.
* So initially GFS and MapReduce were run on the same machine chaging the Map worker read to a local read.
* the reduce shuffle will though always need a network 
* There is a batch approach to MapReduce, we wait for certain data and so on.
  