# HA & Scaling

## Regional and Global AWS Architecture

- considerations (when using a global app like netflix)
    - step 1: global:
        - global service location and discovery
        - content delivery (CDN) and optimisation
        - global health checks and failover
    - step 2: zoom in to regional
        - regional entry point
        - scaling and resilience
        - application service and components
- tiers/components:

***todo: diagram***

## Evolution of Elastic Load Balancers (ELB)

- 3 types, split between v1 and v2 — avoid v1, prefer v2
    - classic (CLB) - v1 - 2009 — not really layer 7, lacks features, 1 ssl per clb
    - application load balancer (ALB) - v2 - http/s/websocket
    - network load balancer (NLB) - v2 - tcp, tls, udp
- v2 is faster, cheaper, and supports target groups & rules

## ELB Architecture

- ELB can have multiple nodes
- 1 ELB can run only in 1 subnet in each AZ, 1+ nodes in each subnet
- dns name (A record) assoc with each ELB, resolving to 1+ nodes
- an internet facing node can point to both public and pvt resources
- /27 subnet addressing is ideal (/28 will be too less, because only 11 free addresses) — target 8+ free IPs per subnet
- nodes are configured with listeners (with port and protocol config)
- LBs can be used to provide **multi-tiering** in application (for e.g. one internet facing, and another pvt)

***todo: diagram***

- a tiered application would somewhat look like this

***todo: diagram***

- cross-zone LB — nodes can distribute to all available instances across zones, not just their own zones (it's a new feature, wasn't there historically)

***todo: diagram***

## ALB vs NLB

- ALB
    - layer 7 only — doesn't do smtp, ssh, tcp, udp, etc
    - looks into L7 content such as cookies, custom headers, user location, app data, etc
    - http/s (ssl/tls) terminated at ALB — no unbroken SSL to resources behind ALB
        - a new connection is made by ALB to application
        - ALB must have SSL certs if HTTPS is used
    - slower than NLBs because more levels of network stack to process
    - health checks evaluate application health also
    - ALB rules — to direct connections based on host header, path, source ip, etc

    ***todo: diagram***

- NLB
    - Layer 4 only — doesnt understand L7
        - no knowledge of headers, cookies, session stickiness
    - extremely fast (25% of ALB latency)
    - anything except http/s (smtp, ssh, game servers, fin apps that aren't http)
    - health checks aren't app aware, just do icmp/tcp handshake
    - can have static IPs — useful for whitelisting
    - forward tcp to instances — unbroken encryption
    - used with pvt link to provide services to other VPCs

## Launch Configs and Launch Templates

- they define what gets provisioned and provide that to ASGs
    - but can also be used directly from console/cli to launch instances
- define config for EC2 in advance — ami, type, storage, key pair, networking, userdata, iam etc
- not editable. LT has versions and has newer features (superset of LC)

## ASG

- min, desired, max instances
- can be done manually, or with policies
- integrated with ELB for elasticity
- ASGs are free, only the resources created are billed
    - use cool down to avoid rapid scaling
- ASG defines when and where, LT defines what
- scaling processes
    - Launch and Terminate — can be suspended and resumed
    - AddToLoadBalancer — add to LB on launch
    - Alarm notif — accept notifs from CW
    - AZRebalance — balances instances evenly across AZs
    - HealthChecks
    - ReplaceUnhealthy
    - ScheduledActions
    - StandBy

## ASG Scaling Policies

- it's not necessary, can be 0 policies assoc with ASG
- Manual scaling
- Dynamic scaling
    - Simple scaling — based on an alarm (e.g. CPU utilization)
        - but not flexible
    - Step scaling — increase/decrease desired capacity based on step adjustment, based on the size of alarm breach
        - e.g. if cpu util between 50-60%, do nothing
        - if 60-70 — add one instance
        - if 70-90 — add two
    - Target scaling — track an ideal target value, e.g. 40% CPU util, based on this add/remove instances
    - Scaling based on SQS — ApproximateNumberOfMessagesVisible

## ASG Lifecycle Hooks

- custom actions on instances during ASG actions, such as instance launch or instance terminate transitions
- instances remain in wait state until the lifecycle action is completed or timed out

## ASG Health Checks

- status checks provided by ec2
- health checks provided by ELB
- custom health checks
- health check grace period

## SSL Offload and Session Stickiness

***todo: diagram***

- Session/connection stickiness:
    - it's when the session state is stored in the instance and not externally
    - this means the instance is not stateless, and the user must be connected to the same instance to access their session data
    - this requires cookies, help with reconnecting the user to the same instance
- ideally, the instances should be stateless, and session data should be stored externally, like in dynamodb

## Advanced Demo — Architecture Evolution

[Instructions (GitHub)](https://github.com/acantril/learn-cantrill-io-labs/tree/master/aws-elastic-wordpress-evolution)

Evolution of a WordPress deployment from monolith to fully elastic:

- stage 1 - setup environment and build wordpress manually

***todo: diagram***

- stage 2 - automate the build using a launch template

***todo: diagram***

- stage 3 - split out DB into RDS and update the LT

***todo: diagram***

- stage 4 - split out WP filesystem into EFS and update the LT

***todo: diagram***

- stage 5 - enable elasticity via ASG & ALB and fix wordpress (hardcoded WPHOME)

***todo: diagram***

- stage 6 - cleanup

## Gateway Load Balancer (GWLB)

- helps you run and scale third party appliances
    - e.g. firewalls, intrusion detection, intrusion prevention, data analysis
    - because, different companies use various other 3rd party tools for various reasons, and need to integrate those with their aws setup
- two major components
    - GWLB endpoints — run from a VPC that is the source or dest of the traffic that you want to monitor
    - GWLB itself — balances packets across multiple backend appliances/instances
- traffic and metadata is tunnelled through using **GENEVE** protocol — between gwlb and backend instances
- there's flow stickiness, hence one flow of data always go to one security appliance instance

***todo: diagram***
