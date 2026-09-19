# AWS Fundamentals

## Public vs Private Services

- public vs private services ⇒ AWS networking: public internet, aws public, aws private
    - aws public gateways

## AWS Regions, AZs, and Resilience

- AWS regions, AZs
    - globally resilient
    - regionally resilient
    - AZ resilient

## VPC (Virtual Private Cloud)

- basically these are aws's private zones
    - region resilient service
        - used to create private networks inside aws, that other pvt services run from
        - also used to connect your aws private networks to your on-premise networks in a hybrid setup
        - also used to connect other cloud platforms when creating a multi cloud deployment
    - 1 default vpc, many custom vpc
    - default vpc: 1 subnet per AZ, has IGW, SG, NACL preconfigured
        - constant CIDR 172.31.0.0/16, and subnets have /20
    - each vpc is private and isolated from others unless configured otherwise

## EC2 (Elastic Compute Cloud)

- AWS's impl of IAAS (you have to setup OS and above, it only provides networking, hardware, storage etc)
    - EC2 service provides access to virtual machines (instances)
    - AMI
- AZ resilient
- ssh for mac and linux (port 22); RDP for windows (port 3389)
- local memory, external storage (EBS)
- 3 states - running, stopped (keeps storage alive), terminated (deletes everything incl storage)

## S3 (Simple Storage Service)

- objects and buckets
- 5tb object size, unlimited objects, unlimited users
- globally unique name, 63 chars, can't look like an ip addr
- no folders/file-system, but can emulate folder by using prefixes (like /abc/def.jpg)
- cheap, should be the first consideration when looking to store data
- runs in public zone of aws

## CFN (Cloud Formation)

- template to spin up instances, services etc
- can be yaml or json
- contains specifications like description, resources, parameters, output, etc
- can be used multiple times, efficient way to create stuff

## CW (Cloud Watch)

- monitoring
- metrics, logs, events
- has alarms - trigger when a metric crosses a defined threshold etc.
- demo
    - spin up an ec2 instance
    - go to cloud watch service, select metric - ec2, per instance metric, select cpu utilization, ≥ 15
    - connect to ec2 instance via browser
        
        ```bash
        ## install stress
        sudo yum install stress -y
        
        ## run stress, for 1 cpu and 3600 seconds
        stress -c 1 -t 3600
        ```

## Shared Responsibility Model

***todo: diagram***

## HA vs FT vs DR

## Route53 — DNS as a Service

- globally resilient service
- it registers domains, and hosts zones (or zonefiles)
    - it has relationships with all major domain registries (.com, .io, etc)
        - these domain registries have been tasked with this by IANA
        - IANA manages the root zone for DNS
        - the registries each manage one zone — e.g. .com zone
    - what happens when try registering a domain with r53?
        1. r53 checks with top-level domain registry, if the reqstd domain is avl
        2. it creates zonefile for the domain being registered, e.g. animals4life.org
            1. zonefile is just a db, containing all dns info for that domain
        3. allocates nameservers for this zone
            1. distributed globally, generally 4 for 1 individual zone
        4. puts zonefile on these 4 managed nameservers
        5. communicates with top-level reg, to add these nameserver records into the zonefile for .org top-level domain 
            1. by doing this, r53 indicates to tld reg that our nameservers are authoritative for the domain
- hosted zone
    - basically a zonefile hosted on 4 managed nameservers
    - can be public (in aws public zone) or private (linked to vpc)
    - hosted zones host dns records
        - they are referred to as recordsets in r53 (tiny difference)

## DNS Record Types

- nameserver records (NS) — in tld registry, point to nameservers
- A & AAAA records
    - map hostname to IP address
    - A-record is for IPv4, and AAAA-record is for IPv6
- CNAME (canonical name)
    - host to host records, kinda like dns shortcuts
        - doesn't point to ip-addrs, only names
        - e.g. if there's an A record called "server", that does multiple things, then many names can be created pointing to it, for convenience
    - reduces admin overhead
- MX records
    - hugely important for how email works on internet
    - it's how a server can find the mailserver for a specific domain
- TXT (text record)
    - add arbitrary text to a domain
    - e.g. to prove domain ownership
        - add random text data in the record, external party can match with that data, proving ownership
