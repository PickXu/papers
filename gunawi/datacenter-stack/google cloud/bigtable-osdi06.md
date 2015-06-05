# Bigtable: A Distributed Storage Systems for Structured Data

### Summary

* Format: (row:string, column:string, time:int64) -> string
* Units
  * Row: atomic to read/write operations
  * Tablet: a row range. Unit of distribution and load balance
  * Column family: unit of access control
* Components
  * Utilize **GFS** to store log and data files
  * Depends on cluster management system for scheduling, failure handling and monitoring
  * Store data in **SSTable** file format for efficient key-value pair operations
  * **Chubby**, the distributed lock service
    * Ensure there is at most one active master at anytime
    * Store bootstrap location of Bigtable data
    * Discover tablet servers and finalize tablet server deaths
    * Store Bigtable schema information
    * Store access control lists
* Tablet Location: three level B+-tree like location hierarchy
  * A Chubby file points to the location of the *root tablet*
  * The root tablet points to the locations of all tablets of the METADATA table
  * Each METADATA tablet points to locations of a set of user tablets
* Tablet Assignment
  * Master and tablet servers each has an exclusive lock on Chubby to declare its liveness in the cluster
  * On startup, a master grabs a unique lock in Chubby. Then, it scans the server directory on Chubby to identify live tablet server. Then, the master communicate with each live tablet server to discover what tablets are already assigned. Finally, the master scans the METADATA table to learn all the tablets
* Tablet Serving
  * Use *memtable* to buffer recent committed updates to tablets in sorted manner
  * Recovering a tablet requires reading the METADATA table to get the list of SSTABLES that comprises the tablet and a set of pointers to commit logs that may contain data for the tablet
  * Writes to a tablet is checked, e.g. form and authorization, before being committed to the log
* Compaction
  * Minor compaction: Compact memtable and SSTables to bound the number of SSTables
  * Major compaction: Compact SSTables and remove delete data persistently
* Refinement
  * Locality groups: Aggregate column families if the content are accessed together frequently
  * Caching: i) Scan Cache; ii) Block Cache
  * Use bloom filter to fast check whether a SSTable contain data of a row/column pair
  * Parallel log commit; Distributed log sorting by tablet ID to eliminate duplicate log reads; Two threads for each log
  * 

### Strenghs

### Weaknesses
