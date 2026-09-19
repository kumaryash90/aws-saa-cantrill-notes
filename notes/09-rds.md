# RDS

## Database Refresher and Models

## ACID vs BASE

## Databases on EC2

- considered a bad practice
- why?
    - lot of admin overhead — managing, backup, replication, monitoring, etc.
    - ec2 is AZ service — if AZ is down, database goes down too
    - performance can be bad
    - EC2 is on/off service, no serverless, no easy scaling
    - missing out on features — AWS does a lot of perf optimisation etc.
- cases where it can be considered
    - require OS access or root access to DB
    - more granular configuration
    - business/vendor want it (!)
    - certain DB versions or DB+OS combo are not there in AWS
    - architecture AWS doesn't provide (replication/resilience)

## RDS Architecture

***todo: diagram***

- RDS uses subnet groups (can span across AZs)
- deploys primary instance and standby instance in separate subnets/AZs
- there can be multiple databases within an instance
- dedicated EBS storage per instance
- data is replicated synchronously between primary and standby
- there can be read replica and s3 backups (these are outside RDS group)
    - AWS manages the s3 bucket
- cost components:
    - instance type and size
    - multi AZ costs (if applicable)
    - storage type and amount (ebs)
    - data transfer
    - backups
        - gb-month usage

## RDS MultiAZ

- Instance mode
    - primary and standby (only 1)
    - standby isn't used until a faliover. only used for backups.
    - one endpoiint, cname pointing to primary
- Cluster mode
    - 1 writer and 2 reader replicas
    - readers can be used along with writer, hence simple scaling
    - data is "committed" when 1+ readers have finished writing
    - two endpoints: cluster endpoint (writer) and reader endpoint
        - there's also instance endpoints, but used only for testing etc.
    - replication is via transaction logs… more efficient
    - much faster hardware
- synchronous replication in both modes

## RDS Backups, Snapshots, and Restore

- automated backups and manual snapshots
- aws manages s3 bucket for this
- snapshots aren't deleted when rds instance is deleted. you have to do it manually
- automated backups occur once per day
    - every 5 minutes, transaction logs are also stored
    - these are deleted automatically (0 to 35 day period) — data older than 35 day is deleted automatically
- RDS can replicate backups/tx-logs to another region, if explicitly enabled
- restores:
    - creates new rds instance, new address
    - snapshots ⇒ single point in time
    - automated ⇒ any 5 minute point in time
        - tx logs are replayed from that point onward (Good RPO)
    - restores are slow (think about RTO)

## Read Replicas

- asynchronous replication
- same region or cross-region
- 5 per primary instance
- applications need to be configured to use the addresses of read replicas, they don't know it by default
- near zero rpo, and near zero rto (unlike backups / snapshots that take time)
    - but only during failure
    - doesn't help with data corruption (because they have same corrupted data)

## RDS Security

- SSL/TLS in transit (can be mandatory)
- KMS for EBS volume encryption
    - handled by Host/EBS
- CMK and data keys
- encryption can't be removed once enabled
- RDS for MSSQL and Oracle use TDE (transparent data encryption)
    - it's handled by the DB itself, more secure

***todo: diagram***

- IAM authentication for db user

***todo: diagram***

## RDS Custom

## Aurora Architecture

- runs as a cluster
    - 1 primary, and 0+ replicas
    - both are active
- no local storage in instances
- shared data volumes
    - all SSD based, high IOPS, low latency
    - storage is billed on what's used
    - high water mark — billed for most used
- auto detects failure of disk, and repairs
- can have up to 15 replicas (min 6 across AZs)
- there's a cluster endpoints, and a reader endpoint (load balances across replicas)
- no free tier, doesn't support micro instances
- backups work the same way as RDS
    - restore creates a new cluster
- there's a backtrack feature — rollback to a previous point in time
- fast clones — copy on write — much faster

## Aurora Serverless

- (what fargate is to ECS)
- removes overhead of managing individual db instances
- scalable — ACU — Aurora Capacity Units
    - serverless cluster has a min and max acu range
    - cluster adjusts based on load
    - these are shared instances, in a pool, allocated as needed
    - no local storage
- consumption billing per second basis
- shared proxy fleet managed by AWS
    - interaction with cluster goes through these
- use cases
    - infrequently used apps, such as a low volume blog
    - new apps
    - variable workloads (thus auto-scaling helps)
    - unpredictable workloads
    - dev and test DBs
    - multi-tenant apps

## Aurora Global Database

- for aurora provisioned clusters
- allows data to be replicated globally
    - good for BC/DR planning
    - also performance improvements for customers

***todo: diagram***

## Multi-Master Writes

- multiple writer nodes in the aurora cluster
- application can connect to 1 or more at the same time
    - there's no single cluster endpoint
- write ops require quorum of all writer nodes
    - also replicated to cache of each node
- almost a fault tolerant system, quick failover, etc

## RDS Proxy

- why
    - opening/closing connections consumes resources
    - takes time ⇒ latency
    - handling failover is hard, doing it in app is risky
    - proxy does connection pooling, abstracts away complexity, reduces latency

***todo: diagram***

- use cases / when to use
    - too many connection errors
    - small/burst instances
    - aws lambda — time saved, connection reuse, IAM auth
    - long running Saas apps
    - when resilience to database failure is a priority
        - RDS proxy can reduce time for failover, make it transparent to the app
- key facts
    - fully managed by aws
    - for both RDS and Aurora
    - auto scaling, HA by default
    - only accessible from a VPC
    - can enforce ssl/tls
    - reduces failover time by 60%

## Database Migration Service (DMS)

- a managed db migration service, runs using a replication instance
- source and destination endpoints — one of them must be in AWS
    - both can't be on-premise databases

***todo: diagram***

- SCT used only when db engine changes
- DMS with Snowball for larger migrations (in multi-TB size)
    1. use SCT to extract data locally and move to a snowball device (ordered from aws)
    2. ship the device back to aws. they load it onto an s3 bucket.
    3. DMS migrates from S3 into target store
    4. CDC can capture changes, and via S3 intermediary they are also written to target database
