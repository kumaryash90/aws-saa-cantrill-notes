# Route 53 — Global DNS

## R53 Public Hosted Zones

- R53 is globally resilient service
- hosted zones ⇒ authoritative for the domain
- public zone ⇒ accessible from public internet and VPCs
    - VPCs have DNS resolvers at +2 address
- r53 can be also used to host externally registered domains

## R53 Private Hosted Zones

- only accessible within their associated VPCs

***todo: diagram***

- split view ⇒ same zone name for public and private, but different set of records available to each

***todo: diagram***

## CNAME vs Alias

## Simple Routing

- 1 record per name
- each record can have multiple values
- doesn't support health checks for the resource that the record points to

## Health Checks

- separate from, but used by records
- not only for aws stuff, but can check anything, just need ip address
- tcp, http/https checks with string matching (first 5120 bytes)
- can check endpoints, cloudwatch alarms, check other health-checks

## Failover Routing

- e.g. using a failover for "out of band" failure/maintenance page (e.g. EC2-S3)

***todo: diagram***

## Multi Value Routing

- multiple records with same name, pointing to diff resources
- returned at random, up to 8, healthy records

## Weighted Routing

- weight associated with the records of same name and diff resources
- if x is weight, then that record/resource is returned (x/total-weight)% of the time
    - if weight is 0, it's not considered
    - if all weights are 0, then all records are considered
- useful for simple load balancing, or when testing new software versions, etc
- if selected record is unhealthy, the process repeats until a healthy record is chosen

## Latency Routing

- optimizing performance and user exp
- multiple records with same hostname but different regions
    - aws maintains latency data for user and region, and returns a healthy record with lowest latency

## Geolocation Routing

- records are tagged with location (country, continent, etc)
- returns relevant records (not closest)
- starts checking with state, then country, then continent, etc, ultimately default record, or none (if no default)
    - you will not get a canadian record returned, if you're based in UK and no closer records exist
- can be used for language specific content, or load balancing across regional endpoints

## Geoproximity

- aims to provide records as close to customer as possible (in terms of physical distance, not latency)
- you can define +/- bias — expands/shrinks a geographic region

## Route 53 Interoperability

- (when either hosting or domain-registration is done with a diff provider)
    - NOTE: r53 can do both jobs — domain registrar and domain hosting, or one of those, but **these are separate functions of route-53 service**
- (registrar — a company that registers the domain on your behalf, here aws' r53)

## DNSSEC

- records/data returned from DNS can be corrupted (accidentally due to old cache etc., or deliberately by a malicious actor)

***todo: diagram***

- DNSSEC strengthens authentication in DNS using digital signatures based on public key cryptography — now a dnssec capable resolver would be able to identify the poisoned data
    
    ```bash
    # just dns query
    dig www.icann.org
    
    # dns query with dnssec -- returns signatures too
    dig www.icann.org +dnssec
    ```

## Implementing DNSSEC Using Route53

- KMS
    - KSK
- ZSK (zone signing key, created/handled within route53, not KMS)
- then route53 adds KSK and ZSK public parts into a DNSKEY record within the hosted zone
- Private KSK is used to sign those DNSKEY records and create the RRSIG DNSKEY record ⇒ these sigs mean any dns resolver can verify
- then route53 establishes a chain of trust with the parent zone — the parent zone needs to add a DS record (delegated signer record) which is a hash of the public part of the KSK for this zone
- DNSSEC validation can be enabled for VPCs — invalid results won't be returned
