# Security, Deployment, & Operations

## AWS Secrets Manager

- shares functionality with parameter store
- but, designed mainly for secrets (API keys, passwords, etc.)
- supports automatic rotation, via lambda
- secrets are encrypted using KMS

## Application Layer Firewall (Layer 7)

- normal firewalls (layer 3/4/5) don't understand anything above these layers
- visibility of data (e.g. http headers, content, dns names etc)

## Web App Firewall

- it's AWS's implementation of layer-7 firewall
- uses WebACLs for config
    - rule groups and rules
- logs → S3, CW, firehose → lambda processing → update WebACL
- rule groups
    - can be referenced by multiple WEBACLs
    - max 1500 capacity (WCU)
- WAF rules
    - type, statement, action
    - types: regular, rate-based
    - statement: What to match, or Count all, or What & Count
        - e.g. can match origin, ip, header, body (frrst 8192 bytes only), etc
    - action: allow, block, count, captcha

## AWS Shield

- managed service for DDoS protection
- network volumetric attacks (L3) - saturate capacity
- network protocol attacks (L4) - TCP SYN flood
- app layer attacks (L7) - web request floods
- aws shield - standard
    - free
    - protection at the perimeter
    - common l3 or l4 attacks
- aws shield - advanced
    - usd 3k per month per org
    - protects more services
    - protection not automatic - must be enabled explicitly
    - proactive engagement, and aws shield response team
    - integration with WAF, app layer (l7)
    - realtime visibility of ddos events and attacks
    - health based detection

## Cloud HSM

- an aws provided hardware security module product
- similar to kms
- fips 140-2 (L3)
- behind the scenes, kms uses HSM
- so, cloudHSM is basically you running your own HSM — true single tenant HSM
- aws provisioned, but managed by customer
- you access it with industry standard APIs (not AWS's) — PKCS#11, microsoft CryptoNG, Java Cryptography Extensions (JCE)
- not deployed in your VPC, but in aws managed vpc for CloudHSM
    - injected into your VPC via ENIs
    - aws has no access to the secure area though
- no native integration with other aws stuff
- can be used to offload ssl/tls processing for webservers

## AWS Config

***todo: diagram***

- records config of resources over time into config history
- stored in s3 bucket
- useful for auditing of changes, compliance with standards (doesn't prevent changes though)
- regional service, supports cross-region/account aggregation
- changes generate SNS notifs and near-realtime events
- config rules — evaluate resources against a defined standard

## Amazon Macie

- data security and data privacy service (it's an ML service)
    - discover, monitor and protect data, stored in S3
    - automated discovery of data such as finance, health, id, etc
- uses data identifier (kinda like rules)
    - managed — built-in (ML, pattern matching etc)
    - custom — proprietary (regex based)
- integrates with security hub and eventbridge
- multi-account (via orgs)
- runs discovery jobs based on schedules and identifiers, outputs findings
    - policy findings (checks when certain bucket policies etc. are changed)
    - sensitive data findings (credentials, personal, financial, etc)

## Amazon Inspector

- automated security assessment service
- scans EC2 instances, OS, container, for vulnerabilities
- provides findings report
- two types of assessment:
    - network assessment (no agent required)
    - host assessment (OS level, agent required) — cve, cis rules pacakges
- rules package determine what is checked

## Amazon GuardDuty

- automated threat detection, continuous monitoring
- analyses supported data sources
- also uses AI/ML
- identifies unexpected / unauthorised activity
- findings can be sent to eventbridge
