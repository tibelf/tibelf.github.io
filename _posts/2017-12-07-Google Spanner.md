# Google Spanner
> a fully managed,mission-critical,relational database service

## Features

* strong consistency
* SQL support
* Managed instances with high availability

## Replication
### Benefits
* data availability
* geographic locality
* single database experience
* easier applicatoin development

## Types

|Replica type|can vote|can become leader|can serve reads|
|---|---|---|---|
|Read-write|yes|yes|yes|
|Read-only|no|no|yes|
|witness|yes|no|no|

read-write is the only type used in single-region instances;

## Instances
instance creation includes two important choices: instance configuration and node count;

### Node 
 * each node provide up to 2 TB of storage
 * minimum of 3 nodes is recommended for production environments
 * the peak read and write throughput values depend on the instance configuration
 
### Regional configurations
If your user and service are located within a single region, choose a regional instance configuration for the lowest latency read and write. 

For each regional configuration, cloud spanner maintains 3 read-write replicas, each within a different GCP AZ.

#### benifits
* 99.99% availability

#### performance
10,000 QPS(read), 2,000 QPS(write);

writing single rows at 1KB of data per row

### Multi-region configurations
Multi-region configurations allow you to replicate the database's data in multiple zones across multiple regions.

multi-region configurations enable your application to achieve faster reads

#### Available configurations
one continent or three continent 

#### benifits
* 99.999% availability
* data distribution
* external consistency

#### performance
* ***one continent***, 7000 QPS(read), 1800 QPS(write)
* ***three continent***, 7000 QPS(read), 1000 QPS(write)

writing single rows at 1KB of data per row

## Reads
### Types
* strong read, the default manner
* stale read

## Transactions
### transactions modes
* locking read-write
* read-only

## IAM
allows you to control user and group access to cloud spanner.

## Monitoring using stackdriver

