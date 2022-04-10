# GFS

## Big Storage as a Distributed System
* Storage has turned out to be a key abstraction
* It is useful, extremely general and a key concept
* Design a good interface is the aim

## Why are distributed storage systems hard ?
* Aim is to get huge aggregate performance with hundreds of machines.
* Split data over these servers performance -> Sharding which might mean more faults
* More faults and servers being down
* Tolerance for fault means replication two copies for data
* If we are not careful we might end up with 2 almost copies of the same data
* Replication risks weird inconsistencies
* That requires more work and chit chat which results in low perfomance
* Loop: Performance -> Sharding -> fault ->Tolerance( Replication mainly ) -> Inconsistencies -> Remedy for these means low performance
* Inevitable way between original goals of performance and consistency and in the end the price for it
* Strong consistency : Behaviour to applications should be as close as talking to a single server
  
## GFS
* Came out in 2004, web was beginning to become a big deal
* Academic interest in distributed computing was starting but was used very little in the industry.
* Big websites like Google started using these ideas slowly.
* Google was coming from the place of large data, say a copy of the web or large youtube videos, enormmous log files etc.
* High speed parallel access was needed to this vast amount of data to bee fed into systems like mapreduce.
* Goal was to have a file system that was Big and Fast
* Globally usable storage system anybody could read the contents
* Sharding splitting the data, automatically split data across several systems
* Automatic failure recovery 
  
### Non Goals
* Single data center so replication and network restrictions are not fully considered
* internal application consumed by google engineer written apps
* Tailored for sequential access of big files not random access
* very different from systems tailored for small data and file consumption ( say bank data of account transactions )
* The conference  where this paper was published usually had very novel ideas, but this one was a different kind of paper. The ideas werent very new, the scale was massive and in use, and it had some heretical ideas like it was okay to have weakly consistent data.
* Mindset then was to have systems that return correct data
* interesting idea was idea of a Single master not several ones for fault tolerance.
* much more tolerane for incorrect data was there in systems like these
* some compensations were expected on the application layer itself.
* We have one  master ( might be replicated ) mappping of file names and where to find the data.
* Master is all about naming and where the data is stored.
* Chunk servers actually contain the data.
* Concerns are almost seperate from each other this way .
* Master data has 2 main tables
* 1 table that maps file name to aray of chunk handles -> tells me where to find the data
* Another table maps chunk handle -> list of chunk severs, version number of each chunk, all writes for that chunk, which chunk server is the primary server, lease expiration of the chunk
* All this data is in the RAM of master.
* In order to ensure crash does not clear up the data master saves this data in the disk as well
* Master logs on Disk, appends entries on logs and on check points writes to disk
* Chunk handles needs to be on disk, list of chunkservers need not be written to disk, version should be written on the disk, primary info is not written on the disk and the same for lease expiration
* Logs are used because append action on a log file is quite effeient in terms of memory access.
* If Master crashes then to recreate its state and not have to read log file from the beginning of time master checkpoints its full state to he disk.
* Goes back to most recent checkpoint and from there till the log end
* Read operation takes in file name and offset to master
* Master send chunk handle and list of servers which has those chunks
* Client can then choose the server and send the read request to that sever
* Client caches this data from master for locality
* Client talks to one of the Chunk server, tells it the chunk handle and offset.
* Chunk server stores these handles as a simple linux file and then the offset is read.
* Writes: filename and a range of bytes I will like to append to the file this is sent by client to master
* Master will send the chunk handle where the current chunk is located
* Wiriting needs a primary Chunk server 
* If there is no primary, master will find out the set of chunk servers that have the most latest version of the data for those chunks/file
* Find up to date replicas
* Latest version numbers -> hence this data is supposed to saved in the disk
* Each chunk server also remembers their chunk version which they saved last
* Then Mastyer picks a primary of these servers.
* Master increments the version number and writes to its disk
* Tells the version to primary and secondary the version numbers on Master now
* Now the Primary is ready to write the data
* Primary is given a lease, allowed to be primary for next 60 second
* This to ensure we dont end up with multi primary
* Client knows now who the Primary and Secondaries are
* Then it sends the copy of the data to be written to all the nodes
* Then the Primary looks at the offset and writes the data to that and asks secondaries to wtite the data at the same offset
* Primary picks the offset for all the replicas
* Secondaries might or might be successful
* If Primary collects a YES from all the secondaries, then Primary replies success to the client.
* If one of he YES from secondaries does not come then Primary replies NO to the client. ( Zero or more replicas might have appended the record to the offset. )
* Then the client will reissue the full record append sequence
* If a reder reads after a failed attempt, depending on the replica they are reading from they might get inconsistent data.
* Version number only changes when Master assigns a fresh Primary. Normally there will be a Parimary preset and then no version change.
*  But some replicas might have a different versions. Reasoning behind this is in a scenario where the secondaries might have different version numbers of the data the Client must have received a NO from the Primary
* Then the Client should retry and eventually we will get the same data/ versions  across all replicas.
* Lease is the answer to the question - What if Master cannot reach Primary for a while, instead of assigning a new Primary directly Master will wait for Primary till end of Lease point.
* It might be that the Primary might still be communicating with a client.
* The error of having 2 primaries is called SPLIT BRAIN caused by NETWORK PARTITION
* Some of the hardest problems to deal with
* After the lease expires Primary cannot service more Client requests, hence by then the Master is sure that Primary is no longer Primary and then can choose a new Primary.
* What it would take to change this to a strongly consistent design ? 
  * Probably need Primary to detect duplicate requests. 
  * Probably need to design a system where Secondary should be able to complete the requests and not just blow up that request from the Primary.
  * Primary should not expose data that has not been written to all Secondaries to any reader. For this we might need 2 Phase commits.
  * If the Primary crashes, then the set of operations that the Primary had spawned, a Secondary might take over as Primary, this new Primary should resynch with all Secondaries. 