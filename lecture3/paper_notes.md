# GFS Paper

## Features
* Abstraction over cheap storage
* High **aggregate** performance
  
## Design choices
* Component failures are a norm and not an exception.
* Inexpensive commodity hardware means there will always be failure somewhere. 
* Hence the system should be able to repair itself constantly.
* Files are huge ( with data and web indices ) hence we have to revisit block sizes. 
* Optimizations are also meant for big sized files.
* **Write Patterns** : Pattern of more append means we want to focus more on append only files rather than random edits within the file. 
* This also means we dont want to cache data blocks at the client as data is too huge and appended. 
* Small writes do not necessarily have to be efficient
* **Read Patterns** : Normally reads are either large streaming reads or small random reads. 
* Large streaming reads normally read from the same location again and again.
* Concurrent appending to a file must be handled with least synchronization overhead.
* Hight sustained bandwidth is more important than low latency.
* Files are organized hierarchically in directories and identified by path-names. 
* Usual operations to create, delete, open, close, read, and write files are supported. 
* Snapshot creates a copy of a file/directory at low cost.
* Record append allows multiple clients to append data to the same file concurrently while guaranteeing the atomicity of each individual client’s append.
  

## Architecture
* Single master
* Mutiple chunk servers
* Mutiple clients
* Files are divided into chunks
* Each of these chunks has a globally unique 64-bit handle
* Chunk servers store these chunks on Linux FS
* Read from Chunk Server is based on Chunk handle and number of bytes needed to be read
* Each chunk is replicated on several Chunk servers
* Master maintains all file system metadata
* This in- cludes the namespace, access control information, the mapping from files to chunks, and the current locations of chunks.
* garbage collection of orphaned chunks, and chunk migration between chunkservers is also master's task.
* A Client interact with master for metadata operation, get the chunk information and then communicate directly with Chunk server to get the data.
* Client caches offer little benefit because most applications stream through huge files or have working sets too large to be cached.
* Reduce involvement of Master so that it does not become a bottleneck. Hence all reads are from ChunkServers directly.
* Client calculates chunk index and file name
* Master gives the ChunkServer index
* Client typically asks a lot of serial ChunkServer indices in one go.
* Chunk Size of 64 MB is much bigger than normal.
* This helps in reducing number of Client Master communication.
* Even for small random reads, the client can comfortably cache all the chunk location information for a multi-TB working set.
* Also the same TCP connection can be used to read data from ChunkServer over a long time.
* Also this means Metadata size can be less on master.
* Disadvantage with a large Chunk Size is that a small file which might be having just one chunk can quickly become a hotspot with several requests for it.
* Replication and help in such a case but only to a degree.
* Allowing clients to read from other clients can make life simpler as well.
* Master saves: file and chunk namespace, mapping from file to chunk and chunk replica wherever it is kept.
* First two are saved in a log file
* ChunkServer information is not stored persistently.
* Master asks a chunkserver about its chunk information on startup of when the chunk server joins the cluster.
* Master metadata is stored in main memory for fast and efficent performance.
* Master keeps polling the ChunkServers to be able to get ChunkInfo, better than keeping all information within main memory of Master.
* Thus eliminating the sync problem between chunk servers and master.
* Another way to understand this design decision is to realize that a chunkserver has the final word over what chunks it does or does not have on its own disks.
* Operation log only pesistent record of metadata, also kind of a log of all Concurrent operations that have taken place.
* This log is also replicated across remote. Any operation is responded to only after its log entry has been flushed to disk remotely and locally.
* On Logs growing beyond a size Master checkpoints it and is able load the latest checkpoint and only replaying logs post that.
* The Checkpoints are compacted in B Tree kind of structure and written into memory.
* Recovery needs only the latest check point and logs post that so older data is deleted.
* The master switches to a new log file and creates the new checkpoint in a separate thread. The new checkpoint includes all mutations before the switch.
* **Consistency Model** File namespace mutations like file creation are atomic and handled by the master
* GFS applies mutations to all replicas of a chunk server in the same order.
* GFS maintains chunk version number to indentify stale chunks
* Stale replicas/chunks are then garbage collected.
* A chunk is lost irreversibly only if all its replicas are lost before GFS can react. 
* In this case, it becomes unavailable, not corrupted: applications receive clear errors rather than corrupt data.
* Aimed at minimizing master's involvement.
  