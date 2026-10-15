# NoSQL and DynamoDB

## DynamoDB Architecture

- it's a NoSQL fully managed DBaaS from AWS
- public
- key/value data & document
- can be provisioned manually or automatic/on-demand
- highly resilient, across AZs and optionally global
- single digit milliseconds access to data (SSD based)
- DynamoDB tables:

    ***todo: diagram — table/item/attribute structure***

    - grouping of items with the same PK
    - no limit on number of items in a table (each item max 400kb)
    - no rigid attribute schema — can have null, all, or a mixture of attributes
- backups:
    - on-demand (manual, full copy retained, cross-region)
    - point-in-time recovery (PITR) (continuous, 35 day recovery window, 1 second granularity)
- considerations for exam:
    - NoSQL → choose DynamoDB, relational → not DynamoDB
    - key/value → default to DynamoDB
    - access is via CLI, API, but not SQL
    - billed based on RCU, WCU, storage, and features

## DynamoDB Operations, Consistency, and Performance

- reading and writing:
    - on-demand: low admin overhead, for unknown/unpredictable demand
    - provisioned: RCU and WCU set on a per-table basis
        - every op consumes at least 1 RCU/WCU
        - 1 RCU = 1 x 4kb
        - 1 WCU = 1 x 1kb
        - every table has an RCU and WCU burst pool (300 seconds)
- query:

    ***todo: diagram — query RCU/capacity consumption***

    - can only query on PK or PK + SK
    - RCU = total size of items returned / 4kb
    - you can filter data and discard items, but capacity is still consumed
- scan:
    - least efficient op, but most flexible
    - moves through the table item by item, consuming capacity for each
    - but you have control over what data is selected (can scan for things that aren't PK/SK, like attributes)
- consistency model:

    ***todo: diagram — eventual vs strong consistency***

    - two modes: eventually consistent, or immediately consistent
    - eventual:
        - easier to implement, scales better
    - immediate/strong:
        - difficult, scales poorly
- WCU/RCU calculation:
    - if you need to store 10 items per second, 2.5k avg size:
        - WCU per item = 2.5k / 1k ⇒ 3 (rounded up)
        - WCU required = 3 x 10 = 30
    - similar for RCU calculation (but reduce it for eventual consistency model, if used)

## DynamoDB Local and Global Secondary Indexes

- indexes are alternative views on data
- LSI: different SK, GSI: different PK + SK
- LSI (local secondary indexes)

    ***todo: diagram — LSI structure***

    - must be created with the base table itself
    - max 5 LSI per base table
    - alternative SK on the table
    - shares RCU and WCU with the table
- GSI (global secondary indexes):

    ***todo: diagram — GSI structure***

    - can be created any time, more flexible
    - 20 per base table
    - alternative PK & SK
    - have their own RCU and WCU allocations
- considerations for exam:
    - careful with projections (`KEYS_ONLY`, `INCLUDE`, `ALL`)
        - queries on attributes not projected are expensive
    - use GSIs as default, use LSI only when strong consistency is needed
    - use indexes for alternative access patterns

## DynamoDB Streams and Lambda Triggers

- stream:

    ***todo: diagram — stream of item changes***

    - a time-ordered list of item changes
    - 24 hr rolling window
    - enabled per table
    - records inserts, updates, deletes
    - streams are the foundation for triggers
- triggers:

    ***todo: diagram — stream + lambda trigger flow***

    - item change generates an event → action taken using event data
    - streams + Lambda
    - use cases:
        - reporting and analytics
        - aggregation, messaging, notification

## DynamoDB Global Tables

***todo: diagram — multi-region global table replication***

- provides multi-master cross-region replication
- tables created in multiple regions, and added to the same global table
- "last writer wins" method for conflict resolution
- reads/writes can occur in any region, sub-second replication
- strong consistent reads only in the same region as writes (hence generally eventual consistency)

## DynamoDB Accelerator (DAX)

- it's an in-memory cache for DynamoDB (should be the default answer for any DynamoDB caching questions)
- tight integration with DynamoDB (as opposed to traditional caches, which are updated by the application that requests data from the DB) — less admin overhead
- it's an in-memory cache, hence much faster (< 400 milliseconds), reduced costs
- supports write-through (writing to the DB) — abstracts away the DB, the app talks to DAX

## DynamoDB TTL

- lets you define a timestamp for automatic deletion of items
- a stream of TTL deletions can be enabled (24 hr window) for any other actions on deleted items

## Amazon Athena

***todo: diagram — schema-on-read over S3***

- it's a serverless interactive query service
- ad-hoc queries on data — pay only for data consumed
- schema-on-read
    - original data never changes, remains on S3
    - schema translates data to relational-like on read
    - normally for other DBs, you have to perform ETL to load data into tables — not required with Athena

## ElastiCache

- in-memory database, high performance
- managed Redis or Memcached, as a service

    ***todo: diagram — ElastiCache in front of a DB***

- can be used to cache data for read-heavy workloads with low latency requirements — reduces DB workload, hence cost-effective
- used for storing session-state data too
- **requires changes in application code**

## Amazon Redshift Architecture

***todo: diagram — leader node / compute nodes***

- it's a column-based, petabyte-scale, data warehousing product
- OLAP based (column based), not OLTP (row/transaction)
- server based (not serverless)
- leader node and compute nodes
- one AZ in a VPC
- enhanced VPC routing

## Redshift DR and Resilience

***todo: diagram — backup/snapshot and cross-region copy flow***

- automatic incremental backups every 8 hrs, or 5GB of data, with a default 1 day retention (can extend up to 35 days)
- manual snapshots can be taken any time
- can be configured to copy across regions
