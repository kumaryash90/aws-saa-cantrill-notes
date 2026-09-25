# Global Content Delivery and Optimization

## Cloudfront Architecture

- a content delivery network — improve delivery of content from its original location to the users — uses caching and an efficient global network to achieve this
- terms:
    - origin — source of content — anything that runs a webserver and has a public ipv4 address
    - distribution — configuration unit of cloudfront
    - edge locations — local cache
    - regional edge cache — larger version of edge location, provides another layer of caching
- 1 or more **behaviors** sit between origin and distribution

***todo: diagram***

## Cloudfront Behaviors

- controls much of ttl, protocol and privacy settings

## TTL and Invalidations

- default ttl 24hrs. can define max/min ttl to restrict per-object ttls.
- when an object is invalid/expired at edge location
    - edge location checks with origin — gets either 302 (not modified), or the new version of the object with 200 status code.
- cache invalidation
    - performed on distribution
    - applies to all edge location, takes time
    - can invalidate individual files, or directories/groups, or everything
    - versioned file-names are ideal, not affected by user cache either
        - e.g. abc_v1.jpg, abc_v2.jpg, etc

## AWS Certificate Manager (ACM)

- https — connection secured by host verification (with cert) and in-transit encryption — ssl/tls handshake
- ACM allows creating / importing certificates
- regional service — certificate must be present in the same region as the service it's associated with
- works only for specific aws services, such as ALB or Cloudfront (not EC2)
- for cloudfront, even though it's global service, the region for creating acm certificate is always us-east-1
    - this certification is then passed-on/applied to the edge locations (which may be in different regions)
- certificate required at each step — from viewer to cloudfront, from cloudfront to origin, and any other intermediate step
- public and private — for cloudfront, only public certs can be applied to cf distribution (can't use self-signed)
    - viewer protocol and origin protocol
    - s3 handles certs natively
    - for ALB, need public cert
    - custom origins such as EC2 or on-premise, need public certs (but ACM doesn't support it, so you need to supply manually)
- acm issued certificates are auto renewed

## Cloudfront and SSL/TLS

- before 2003, every ssl enabled site needed its own ip — it wasn't possible for a server to host more than 1 website
    - host headers (layer 7) were used to check the host name etc., and ssl is applied at transport layer — hence this problem
- post 2003, SNI (server name indication) extension was added to TLS — allows client to tell which domain name it wants to access — this occurs within tls handshake, before http is involved
- SNI mode in cloudfront — free and default
    - but to support old browsers, you may need dedicated IP, which is charged extra

## Origin Types and Architecture

- types
    - s3 buckets
        - simplest to integrate, because designed to work with cf
        - can be restricted to be only accessed via cf
    - aws media package channel endpoints
    - aws media store container endpoints
    - custom (webservers)
        - viewer protocol and origin protocol can differ
        - more granular config (custom ports etc., protocol etc.)

***todo: diagram***

## CF Security — OAI and Custom Origins

- origin access control
    - enables auth for s3, lambda, and other origins
    - attached to an origin within a distribution
    - cloudfront signs every origin request (SigV4)
    - cf distribution added to resource policy
- origin access identity (OAI) — legacy way to do the above
    - it's an actual identity, unlike oac
    - cf becomes that oai

***todo: diagram***
