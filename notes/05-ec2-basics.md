# EC2 Basics

## Virtualization 101

- privileged mode is the main thing to solve here
- history, from earliest to latest tech:
    - emulated (software/hypervisor sits between, and does binary translation)
    - paravirtualization — guest OS are modified
    - hardware assisted virtualization — but doesn't really solve IO related issues (network transfers and disk IO)
    - SR IOV — this is what EC2 uses, each hardware such as network card presents itself as a mini version of itself to the guest OS, etc

## Architecture

***todo: diagram***

- ***AZ***
- EC2 is good for traditional style compute with OS+app, or monolithic service, long running requirement, etc.

## EC2 Instance Types

- categories include: default (general purpose), memory optimized, storage optimized, accelerated compute, etc.
- [ec2instances.info](https://ec2instances.info)
- [aws.amazon.com/ec2/instance-types](https://aws.amazon.com/ec2/instance-types/)

## Storage Refresher

- direct attached — ephemeral, network attached (EBS) — persistent
- categories:
    - block storage
        - volume presented to OS as collection of blocks, raw, no other structure
        - bootable, mountable — OS creates file system on top of this
        - e.g. hard disk, SSDs
        - most EC2 use EBS volume as boot volume
    - file storage
        - provided by fileserver, readymade filesystem
        - mountable, but not bootable because OS doesn't have low level access
    - Object storage
        - abstract, no structure, flat collection of objects & attached metadata
        - not mountable, not bootable
        - super scalable
- storage performance
    - IO (block) size (size of wheels of racecar)
    - IOPS (rps, engine of racecar)
    - Throughput (speed of racecar)
    - IO x IOPS = Throughput
    - e.g.
        
        ```
        let 
            block size = 16 KB
            IO ops per sec = 100
            
        then
            througput = 16 KB x 100 = 1.6 MB/s
        ```

## EBS

- AZ resilient service
- typically attached to one instance
- persistent
- to migrate between AZ, do snapshot/backup in S3
- billed based on GB/month
- **it's not affected by ec2 lifecycle, since it's a separate service**

## EBS Volume Types — General Purpose

- GP2: general purpose SSD based storage, recommended for most work
    - 1 GB - 16 TB in size, created with an IO credit allocation
    - 1 credit (or IOPS) = 1 IO op of 16KB block/sec
    - IO credit bucket has max capacity of 5.4 million IO credits
    - fills at a rate of baseline performance of the volume type (min 100)
        - on top of that, add 3 credits per second per GB of volume size
    - also, GP2 can burst up to 3000 IOPS (depending if you have that in your bucket) — great for boots / initial workloads
    - for volumes larger than 1TB, credit system isn't used, and a baseline of above burst rate is always provided
        - max 16000 credits per second
- GP3: newer, SSD based
    - removes credit bucket architecture
    - every volume, regradless of size, starts with 3000 IOPS & 125 MiB/sec
    - 20% cheaper than GP2
    - higher max throughput

## EBS Volume Types — Provisioned IOPS SSD (io1/2)

- iops can be adjusted independently of the volume size
- low latency (sub-millisecond) and jitter
- max 64,000 iops per volume (4 times gp2/3)
- for block express it's 256,000
- volume size range 4GB - 16TB for gp2/3, and 4GB - 64TB for block express
- this means you can have large iops even for smaller volumes, but there's a max value for size-to-performance ratio
    - 50 iops/GB for io1
    - 500 iops/GB for io2
    - 1000 iops/GB for block express
- type/size of instance can limit this performance though (if small instance, then it won't matter even if you have block express)
    - there are instance maximums

## EBS Volume Types — HDD Based

- cheap, not great at random access
- ST1 — throughput optimized, good for streaming
    - good for sequential access
    - 125 GB to 16TB
    - max 500 IOPS, but block size is 1MB!!!
- SC1 — cold
    - cheaper
    - used when performance not an issue, store lots of data
    - max 250 iops, 1MB block
- can't be used for booting

## Instance Store Volumes

- attached directly to the instance (basically, present on the same host)
- ephemeral, lost when instance is moved, resized or if any hardware failure
- extremely high performance (throughput in GBs)
- added when launching instance, can't do later
- included in instance cost, can't opt out
- ephemeral [0-23]

## EBS vs Instance Store Volumes

- EBS benefits:
    - persistent, resilient, backup, add/remove later, isolated from instance lifecycle
- Instance store benefits
    - super fast, cost effective
- EBS has max iops of 256,000 (block express), can extend up to 260,000 by creating RAID0 using multiple EBS volumes
- If more iops required, instance store is the only option, but only if persistence is not a concern — good when replication is built in your app
- instance store is also used data is replicated across a fleet of instances, such as a load-balanced pool of servers

## Snapshots

- S3
- full backup initially, incremental after — you're only billed for used data (not allocated data), gigabyte-month
- backup/restore across AZs and regions
- backups can be lazy/slow, affecting performance
    - one way to force it is read all blocks immediately when restoring
    - or, use FSR (fast snapshot restore) — it's charged, and upto 50 snaps / region

## EBS Encryption

***todo: diagram***

## Network Interfaces, Instance IPs, DNS

- ENIs (elastic network interface)
    - each instance has 1 primary ENI and 0 or more secondary ENI
        - the IPs, SGs, etc are applied to these ENIs, and not the instance directly
- primary ENI
    - mac address
    - primary IPv4 private IP
        - there's a dns generated for it
            - e.g. ip-10-16-0-10.ec2.internal
            - within VPC, resolves to the private IP
    - 0 or more secondary IPs
    - 0 or 1 public IPv4
        - dns e.g. ec2-3-89-7-136.compute-1.aws.com
            - within VPC resolves to private IP, outside VPC, resolves to public IP
    - 0 or more IPv6
    - 1 elastic IP for each private IP
        - if this is set, then it removes the public IP, and replaces with this public elastic IP
    - security groups
    - source/destination check
- secondary ENI
    - same as above, but
    - can be detached and attached to different EC2s
        - helps with licensing (which are applied to mac)
- v imp: the instance OS doesn't see public IPv4

## AMI

- each AMI has unique ID, and is regional (diff AMI for diff regions)
- permissions: private, public, specific accounts
- can create EC2 instances
- can be created from EC2 instances (useful for creating bespoke config ec2 later)
    - (AMI Baking)
- AMI lifecyle
    - launch - configure - create image - launch
- ebs snapshot is also created when creating ami. it contains block device mapping.

## EC2 Purchase Options (Launch Types)

- On demand (default)
    - shared hardware, per-second billing (except other assoc resources)
    - ideal for short-term/unknown workloads
- Spot
    - cheapest (up to 90% discount)
    - aws sells any unused ec2 host capacity
    - if spot price goes above your max price, then your instances are terminated
    - never use for un-interruptible workloads
- Reserved
    - for long-term, consistent usage
    - reduced or no per second price, but there could be reservation fee
    - unused reservation is still billed
    - must be either 1 yr or 3 yr terms
- Dedicated host
    - entire ec2 host allocated to you (not just an instance)
    - you pay for the host, not for the ec2 instances running
        - can run any type of instance as long as host has capacity
    - host affinity — links instances to hosts (can't move)
    - it's mainly used for socket and core licensing requirements
- Dedicated instances
    - kinda similar to dedicated host, but you're not using the entire host, but just some instances with dedicated hardware (aws doesn't allocate to anyone else)
    - higher charges than above
    - useful when there are no strict licensing req

## Reserved Instances

- standard
- scheduled
    - ideal for long term usage, which doesn't run constantly
    - e.g. batch processing daily for 5 hrs, starting at 23:00
    - e.g. weekly data analysis on friday for 24 hrs
    - e.g. 100 hrs of ec2 per month
    - doesn't support all instance types or regions
    - minimum 1200 hrs per year & 1 year term
- capacity reservations
    - useful when you have un-interruptible compute requirement
    - but can't justify long term reservation
    - on demand capacity, at on-demand prices
        - but you pay regardless of if you consume it
    - no term limits
- ec2 savings plan (very powerful feature)
    - hourly commitment for 1 or 3 yr term, in exchange for less prices
    - beyond your commitment, regular on-demand pricing applies

## Instance Status Checks, Alarms, and Auto Recovery

## Shutdown, Termination, and Termination Protection

## Scaling

- Vertical:
    - basically, increasing the size of instance (but ultimately there will be an upper limit, can't do it forever)
    - there's disruption when scaling (because restarting, etc)
    - can get costly with the size of instance
    - no app rewrite/duplication required
    - data stays on the same machine
- Horizontal
    - adding new instances
    - requires a load balancer
    - need to maintain sessions (like login status, cart contents, etc.)
        - so, requires either app support, or off-host sessions
        - off-host: servers are stateless, session data stored externally
    - no disruption when scaling
    - theoretically no limit to scaling
    - less expensive
    - more granular control (adding smaller instances, instead of doubling instance size everytime like in vertical scaling)

***todo: diagram***

## Instance Metadata

- service provided by ec2 to instances — configure/manage the instance
- accessible inside all instances — **http://169.254.169.254/latest/meta-data**
- access to info about instance's environment, networking, auth, user-data, etc
    - for e.g. — while OS on ec2 can't see its public IPv4, the applications running on the instance can use this metadata to check this info
- **this service has no authentication, and is not encrypted**
