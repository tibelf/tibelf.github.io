# Azure Cosmos DB
> Globally distributed, multi-model database service

* ***Turnkey global distribution***: easily build globally-distributed applications
* ***Multi-model + multi-API*** : 
	* support key-value, graph, column-family, and document data in one service
	* SQL API, MongoDB API, Table API, Graph(Gremlin) API, Cassandra API
* ***limitless elastic scale around the globe***: only pay for the throughput and storage you need
* ***Multiple, well-defined consistency choices***: offer five well-defined consistency levels
* ***guaranteed low latency at 99th percentile***: less than 10-ms latencies on reads and less than 15-ms latencies on writes at the 99th percentile
* ***Industry-leading, enterprise-grade SLAs***: SLA for 99.999%


## Data migration tool
the migration code is opensource on Github, offer a solution to import data to Azure CosmosDB from variety sources, including:

* Azure Tables
* JSON files
* MongoDB
* SQL Server
* CSV files
* RavenDB
* Amazon DynamoDB
* Hbase
* DocumentDB

[Github repository](https://github.com/azure/azure-documentdb-datamigrationtool)

### Capability

* ***SQL API***: support any of the source options
* ***Table API***: support any of the source options
* ***MongoDB API***: not support
* ***Graph API***: not support

## concepts
### distribute globally
across 30+ geographical regions

### Partitioning
Azure Cosmos DB provides containers for storing data called collections, graphs, or tables.

Containers are logical resources and can span one or more physical partitions or servers.

The number of partitions is determined by Azure Cosmos DB based on the storage size and the provisioned throughput of the container.
### unique keys
A unique keys can be consisted of one or more values, and the unique  key is scoped to the partition key.

Once a container has been created, the unique keys cannot be updated.

### Consistency
Azure Cosmos DB provides five consisstency levels: strong，bounded-staleness, session, consistent prefix, and eventual. 

bouded-staleness, session, consistent prefix, and eventual are referred as "relaxed consistency models" as they provide less consitency than strong.

|Consistency Level|Guarantees|
|---|---|
|Strong|Linearizability. Reads are guaranteed to return the most recent version of an item|
|Bounded Staleness|Consistent Prefix. Reads lag behind the write by k prefixes or t interval|
|Session|Consistent Prefix. Monotonic reads, monotonic writes, read-your-writes, write-follows-reads|
|Consistent Prefix|Updates returned are some prefix of all the updates, with no gaps|
|Eventual|Out of order reads|

#### Service level agreements
***99.99% SLA***: guarantee throughput, consistency, availability and latency for Azure Cosmos DB accounts scoped to a single region with any of the five consisitency levels, or multi-regions with any of the four relaxed consistency levels.

***99.999% SLA***: read availability for database accounts spanning two or more Azure regions.

### Security
* Network security, using the IP firewall
* Authorization, each request is hashed using secret account key
* https
* encryption at rest
* ISO 27001， European Model Clauses(EUMC), HIPAA certifications

 



