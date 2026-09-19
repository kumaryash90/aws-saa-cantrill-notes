# VPC Basics

## VPC Sizing and Structure — Considerations

- avoid common IP ranges
- find out already used IP ranges across your company
- talk to vendors and find out any conflicts, avoid those ranges as well
- AWS min VPC size = /28 and max = /16
    - so if we choose 10.x.y.z ⇒ we can have 256 vpc networks (varying x from 0 to 255, and each network has 256*256 = 65536 ip addresses)
    - can also avoid 10.0.y.z upto 10.10.y.z, because these are commonly used
- also reserve 2+ networks per region per account
    - so if we need 3 regions in US, 1 in europe, and 1 in aus ⇒ 5 regions
        - 5 regions * 2 networks per region = 10
        - and if there are 4 accounts ⇒ 10 * 4 = 40 ranges (ideally)
    - also consider any remote/field-workers
    - "*basically, try to predict the future*"
- how many subnets you need? how many IPs per subnet?
    - decide based on AZs required and available per zone — can do 3 + 1 (for future), because → web tier, app tier, database tier, plus one spare

    ***todo: diagram***

- business could grow — accommodate for huge future growth

## Custom VPC

***todo: diagram***

- DNS in VPC provided by r53. its IP = baseIP + 2
    - you can also enableDnsHostnames and enabledDnsSupport (if needed)

## Subnets

- subnets within a vpc can communicate with each other by default
- there are 5 reserved IP addrs per subnet, we can't use those
    - so, for e.g., for our VPC subnet with CIDR 10.16.16.0/20, reserved addr are:
        - network address (10.16.16.0)
        - network + 1 (10.16.16.1) — VPC router
        - network + 2 (10.16.16.2) — Reserved (DNS*)
        - network + 3 (10.16.16.3) — future use
        - broadcast addr (10.16.31.255) — last IP in subnet
- other configs:
    - DHCP option set available per VPC
    - two imp IP allocation options
        - option 1 - auto assign public IPv4 — controls whether resources in the subnet are allocated a public addr in addition to their pvt addresses
        - option 2 - auto assign public IPv6

## VPC Routing, IG, Bastion Hosts

- VPC router
    - at addr network + 1
    - route table — either MAIN (from VPC config default), or custom table attached to the subnet — can only have one
    - router analyses destination addr in the packets — higher suffix takes priority (/32 is more specific and higher priority than /16, or 0.0.0.0/0 catchall)
    - local target ⇒ same subnet. is prioritised.
- internet gateway
    - region resilient, 1 per vpc
    - 1 vpc = 0 or 1 IG, 1 IG = 0 or 1 VPC
    - runs from aws public zone
- bastion host / jumpbox
    - an instance in a public subnet
    - incoming management connections
    - often the only way IN to a VPC
- demo — creating a public subnet connection. steps:
    - create IG. attach it to the vpc.
    - route table — associate web subnets. add default ipv4 and ipv6 routes to IG.
    - enable default public ipv4 settings in each web subnet
    - create ec2 bastion instance, select a web subnet.
    - now we can connect to this instance via ssh

## Stateful and Stateless Firewalls

## NACLs (Network Access Control Lists)

- associated with subnets (can't be attached to aws resources, unlike SGs), check inbound and outbound, but not within subnet
- are like stateless firewalls, and need to be configured explicitly for inbound and outbound
- can get complex very quickly with growing subnet number etc
- better used in conjunction with security groups, where NACL act as explicit deny mechanism for specific IPs
- 1 subnet can have 1 NACL, 1 NACL can be assoc with many subnets

## SGs (Security Groups)

- are like stateful firewalls (detect response traffic automatically)
- don't have explicit deny.. only explicit allow and implicit deny (thus can't block specific addresses/networks) — hence need NACL along with it
- can reference logical resources (and itself) unlike nacl
    - (technically, these are attached to the network interface (ENI) of the ec2 instance, not the instance itself)

***todo: diagram***

## NAT Gateways

***todo: diagram***

- must run from a public subnet
- adjust pvt ip to public ip in packets
- works for only outgoing traffic
- uses elastic IPs
- AZ resilient
- *(home routers use NAT)*
- v imp:
    - if you need to give an instance its own public ipv4 address, only IG is required
    - but, if you need to give pvt instances outgoing access to public internet and services such as S3, then you need both NAT gateway (to do this many to one translation) and IG (to translate from IP of NAT to a real public IP)
- it's a aws managed service
- billing based on hourly use per NAT, as well as data volume
- v. imp: NAT gateways can't be assoc with a Security Group
- NAT isn't required for IPv6, because all IPv6 in aws are publicly routable and work with IG directly
- (earlier, they used to run EC2 as NAT instance — run NAT process on EC2, and disable source and destination checks — because by default ec2 drops any network data for which it's neither the source or dest)
    - managed by customer, not scalable, less feature, many failure modes
