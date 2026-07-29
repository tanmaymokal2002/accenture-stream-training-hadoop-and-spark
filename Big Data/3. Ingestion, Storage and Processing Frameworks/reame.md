We will specifically dive deep into the following:

Storage Layer (HDFS Architecture, read and write algorithms, fault tolerance)
Ingestion Layer (Sqoop and Kafka)
Processing Layer (MapReduce, Pig, Hive, Spark - Internals, and demos)
Resource Management (YARN Architecture)

---

Summary
HDFS acts as a storage layer for the whole Hadoop stack
Open-source version of Google FIle System
System design assumptions
Usage of commodity hardware
Failures are not an anomaly, its expectation, system to automatically handle failures
Record appends for writes instead of an inline update to records
HDFS Architecture
Simple architecture

Master node (name node) - holds metadata about the data stored on data nodes. Never contains actual data
Data node (chunk servers, slave nodes, child nodes) - holds actual data

HDFS and GFS are extremely similar, so we can use this diagram to look at how the architecture is structured behind the scenes. The Application initiates a transaction, which is then handled by the Master, which interacts with the chunk server(data nodes) from the actual data

---

Summary
All stored data is broken up into smaller "chunks" by Hadoop.
Chunks are also known as a block
Block size is 128 MB. You can configure the block size of the cluster by changing dfs.blocksize property
Replication Factor: To provide fault tolerance, there are 3 copies of each chunk, which are stored across multiple nodes.
You can think of it as a book.

The Table of content is the 'master node'
Each page of the book is a 'data node'
Each page's content is the actual 'data'
Master(Name) nodes interact with data nodes in order to store data in triplicates. Use the following diagram as you follow along in the video

---

Summary
HDFS Read Algorithm

Application sends a request to HDFS client with the file name and byte-range (i.e what part of the file application needs to read)
HDFS client converts the byte range into the block index number and forwards the request to HDFS master nodes
Master nodes look up the metadata and respond back with locations of servers who have the copy of the block requested by the application
Client connects directly to the available data nodes and requests data. If a data node is not available, the client will retry to other data nodes from the list provided by the master. Retry is tried 3 times before failure
Data node responds with the file to HDFS Client
HDFS client sends the data back to the application

---
