# Advanced VPC Networking

## VPC Flow Logs

- monitor flow of traffic, capture metadata (not contents)
    - version, account id, interface id
    - src & dest addrs and ports
    - protocol number (ICMP = 1, TCP = 6, UDP = 17)
    - packets, bytes, start and end
    - action (ACCEPT, REJECT)
    - log-status
- attached to:
    - VPC (all ENIs in VPC)
    - subnet (all ENIs in subnet)
    - ENIs directly
- not realtime
- destination - s3 or cw logs
- use athena querying engine for querying (from s3 using sql)
- certain types of traffic not recorded (DHCP, DNS server, Amazon Windows license)

## Egress-only Internet Gateway

- for IPv6 instances (because they're publicly routable by default, and NAT doesn't work with them, unlike IPv4)

## VPC Endpoints

- **Gateway**
    - private access to supported services (S3, DynamoDB)
        - without needing IG or NAT (hence no public internet access)
    - prefix list added to route table
    - endpoint policy
    - can't access cross-region services
- **Interface**
    - same function as above, but different method of achieving it
        - doesn't use prefix list
        - uses DNS, resolving to pvt ip of interface endpoint
            - regional DNS or zonal DNS
            - pvt dns overrides the default dns for services
    - for anything not dynamoDB (can access S3 now)
    - not HA by default, added to specific subnet/ENI
    - security group, endpoint policies
    - tcp and ipv4 only
    - uses **PrivateLink**

## VPC Peering

***todo: diagram***

- direct encrypted network link between two VPCs (only 2)
- connection is not transitive
- same region peers can reference each other (SGs)
- need to configure route tables on both vpcs, logical peer gateway
- there can't be overlap in CIDR for peering
