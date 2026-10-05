# Hybrid Environments and Migration

## Border Gateway Protocol (BGP) 101

***todo: diagram***

- is what Direct Connect and Dynamic VPN use
- AS — autonomous system - routers controlled by one entity — a network in BGP
    - ASNs are unique. allocated by IANA. 0 - 65535
- it's a path-vector protocol — best path to destination between peers
    - ASPATH
- operates over tcp/179
- *(it's like a collection of networks?)*
- iBGP (routing within AS) and eBGP (routing between AS's)

## IPSec VPN Fundamentals

- IPSec

    ***todo: diagram***

    - a group of protocols, sets up secure tunnels across insecure networks
    - between 2 peers (local and remote), provides auth and encryption
    - two main phases
        - 1: IKE Phase 1 — Internet Key Exchange — slow & heavy
            - auth
            - asymmetric encryption to agree on and create symmetric key (Diffie Hellman key exchange)
            - IKE SA (security association) created — phase 1 tunnel complete
        - 2: IKE Phase 2 (fast and agile)
            - uses keys agreed in phase 1
            - agree encryption method, and keys used for bulk data transfer
            - IPSec SA — phase 2 tunnel
- Policy based VPNs (rule sets match traffic. a pair of SAs) and Route based VPNs (target matching (prefix). matches a single pair of SAs)

## AWS Site-to-Site VPN

***todo: diagram***

- runs over public internet (*) (not Direct Connect)
- full HA if designed/implemented correctly
- quick to provision (< 1 hr)
- components involved:
    - VPC
    - VGW - virtual private gateway
    - CGW - customer gateway
    - VPN connection between VGW and CGW
- static vs dynamic vpn
    - static has static route configuration on both sides. no load balancing and multi-connection failover.
    - dynamic vpn uses **BGP** — networks exchange info on routes etc. multiple vpn connections provide HA and traffic distribution
- considerations:
    - speed limit on aws side — ~1.25Gbps
    - latency — public internet introduces variability, inconsistency
    - cost, data cap
- benefits
    - quick to setup (few hours)
    - can be used as a backup for physical connections such as Direct Connect
    - can be used with physical connections

## Direct Connect (DX) Concepts

- business premises ⇒ DX location ⇒ aws region
- aws allocates physical port at DX location, to connect a physical cable from business premises
- high speed, low latency, because no public internet (unlike vpn)
- 1, 10, or 100 Gbps
- create VIFs (virtual interfaces) to access aws private and public services, but not internet (needs additional config)

## DX Resilience and HA

## DX - Public VIF + VPN (Encryption)

***todo: diagram***

- running a vpn over dx — auth/encryption + low latency
- uses a public vif + vgw/tgw endpoints

## AWS Transit Gateway (TGW)

- network transit hub — to connect VPCs to each other or on-premise networks
- designed to reduce network complexity
- attachments to other network types
    - valid attachment types: VPC, site-2-site, dx gateway
- without tgw — complex network, scales badly
    - (because peering is not transitive)

        ***todo: diagram***

- with tgw — simpler, globally routable, scalable network

    ***todo: diagram***

- considerations:
    - supports transitive routing
    - can be used to create global networks
    - can be shared between accounts
    - peer with different regions, same or cross account
    - less complexity vs without TGW

## Storage Gateway - Volume

- storage gateway is used to integrate local infra with s3, ebs, etc.
- generally runs as virtual machine on premises
- presents storage as volume/raw-blocks using iSCSI, NFS, SMB
- volume stored mode
    - vm presents data over iscsi, similar to NAS on-premise
    - **volume is stored locally, on premise**
    - also has upload buffer (temp) — copied eventually to aws — ebs snapshots
    - benefits:
        - great for full-disk backups
        - disaster recovery
    - limitations: doesn't improve local datacenter capacity
- volume cached mode
    - same basic arch as above,
    - but:
        - primary location of data is in s3, not on-premise (you can still create ebs snapshots though)
        - there's a local cache for frequently accessed data
        - great for extension

## Storage Gateway - Tape (VTL - Virtual Tape Library)

- a library is 1+ drives, 1+ loaders, and slots
- storage gateway presents a media changer and tape drive over iscsi
- has local cache and upload buffer
- aws presents vtl backed by s3, and vts via glacier

## Storage Gateway - File

***todo: diagram***

- bridges local file storage over NFS and SMB with S3 storage
- files stored into a mount point are visible as s3 objects
- local read/write caching
- also supports s3 object lifecycle management

## Snowball / Edge / Snowmobile

- move large amounts of data in/out of aws
- physical storage units — suitcase/truck size
- ordered from aws, empty, loaded and returned
- encryption using kms
- 10 tb to 10 pb
- multiple devices to multiple premises
- snowball edge — storage + compute, larger capacity
    - storage optimized, compute optimized, compute with gpu
    - good for remote sites where you need storage with compute
- snowmobile — portable datacenter within a shipping container on a truck
    - up to 100 pb data

## Directory Service

- managed directory service — a store of users, groups, servers, file shares, etc
- centralised auth and management
- e.g. microsoft active directory
    - alternative is open source SAMBA
- private service, runs within vpc
- can be isolated, integrated on-premise, or as a proxy
- modes:
    - simple AD mode
    - aws managed microsoft AD
    - AD connector — proxies requests back to on-premises directory

## AWS DataSync

***todo: diagram***

- data transfer to/from aws
- huge scale
- keeps metadata
- built-in validation
- key features:
    - scalable - 10Gbps per agent
    - incremental and scheduled transfer options
    - compression and encryption
    - etc

## FSx for Windows Servers

- provides a native windows file system as a service
- key features and benefits:
    - VSS - user driven restores
    - accessible over SMB
    - supports DFS
    - managed - no file-server admin overhead
    - integrates with directory service and on-premise directory

## FSx for Lustre

- designed for HPC - Linux clients (POSIX)
- machine learning, big data, fin modeling etc
- highly scalable, sub-millisecond latency
- deployment types: persistent or scratch
- metadata targets (MSTs) and object storage targets (OSTs)
- designed for short term / temp workloads. no ha, no replication.

## AWS Transfer Family

- managed file transfer service, to/from S3 and EFS
- ftp, ftps, sftp, as2
