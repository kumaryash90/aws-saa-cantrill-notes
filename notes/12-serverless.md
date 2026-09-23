# Serverless and Application Services

## Architecture Deep Dive

### Monolithic Architecture

- all tiers are together ⇒ fail together, scale together, bill together

***todo: diagram***

### Tiered Architecture

- logical separation/grouping
- each tier can scale independently (for e.g. can vertically scale processing instance to a bigger size without affecting upload and store tiers)
- no need to have direct connections — can use load balancers instead — and have multiple instances at each tier (horizontal scale)
- **problems**:
    - tiers are still coupled ⇒ e.g., upload tier expects and requires the processing tier to exist and respond ⇒ failure of processing tier impacts upload tier

### Evolving with Queues — Queue-Based Decoupled Architecture

- e.g., you upload a video → gets stored in s3 bucket → message added to the queue, containing data for processing job and video location in s3… no direct / synchronous communication with processing layer
- on the other end — the processing tier has an ASG, scaling up/down based on this queue length… this **worker fleet** (instances) in ASG take fifo ordered msgs from queue, fetch video, do the processing
- fully decoupled, each tier doing its own thing **async**, working with queues

***todo: diagram***

### Microservices

- if we continue to breakdown the monolith, we can arrive at self-sufficient services running independently — microservice
- but using queues here can get complicated very fast, because there can be so many such microservices and so many such queues

***todo: diagram***

- event driven arch solves this complexity

### Event-Driven Architecture

- instead of multiple queues between each producer and consumer, there's an event router — decides which consumers to deliver the events to
- the producers and consumers don't run constantly, instead they stop and save resources, and work only when there's a relevant event
- here, resource allocation and scale-up is done when an event is received — consuming resources as and when required — no constant polling, no wasting of resources
- an event can be any meaningful action — e.g. you click order on amazon, or there's a error msg in monitoring system, or video uploaded, etc
- this event driven use of resources is a key component for **serverless arch**

## AWS Lambda

***todo: diagram***

- function-as-a-service — a piece of code that runs in a runtime environment (e.g. python 3.8)
- billed for duration that a function runs
- a key part of serverless architecture
- docker is an anti-pattern for lambda
- it's stateless — everytime a function is invoked, it's brand new in a brand new env
    - there are some cases where state is possible, but in general it's stateless
- custom runtimes are possible using "layers"
- you don't control the cpu, rather you specify the memory needed, and then there's an indirect cpu allocation based on this
- function timeout at 900 sec (15 mins) — can't run lambda beyond that
    - you can create lambda workflows with step functions to work around it
- 512 MB storage available as /tmp

### Common Uses

- serverless applications (s3, API gateway, lambda)
- file processing (using s3, s3 events, lambda)
- database triggers (dynamo db, streams, lambda)
- serverless CRON (EventBridge/CWEvents + Lambda)
- realtime stream data processing (Kinesis + Lambda)

### Public and Private Networking for Lambda

***todo: diagram***

### Security

- uses execution role, and can have a resource policy

***todo: diagram***

### Logging

- cloudwatch stores metrics, cloudwatch logs
- X-Ray for distributed tracing
- cloudwatch logging requires permissions via execution role

### Invocation Methods

1. **synchronous**
    - invoked via cli/api, and client waits for response
    - errors and retries have to be handled at client side
    - used typically when it's a human invoking the lambda function
2. **asynchronous**
    - when aws services invoke lambda on your behalf
    - e.g. s3 upload → sends event to lambda
    - re-processing/retries can be configured within lambda
        - lambda function needs to be idempotent for this
        - events can be sent to dead letter queue (DLQ) after repeated failures
3. **event source mapping**
    - used on streams that don't generate events (kinesis, dynamodb streams, sqs)
    - instead these use polling queues, that look for new data and get new batches from source
        - then creates/sends event batches for lambda
    - v imp: event-source-mapping uses execution role from lambda to access the source and retrieve data (for other cases, lambda doesn't generally need permissions because generally no access to the source is needed)

### Lambda Versions

- can define versions of the function
- version = code + configuration
- version is immutable once published
- $Latest points at the latest version
- can also create aliases pointing at different versions (dev, stage, prod) — can be changed

### Lambda Start-Up Time

- lambda runs inside a runtime env, also known as execution context — it's kinda like a small container that's allocated resources to run your code
- cold start and warm start

***todo: diagram***

## Cloudwatch Events and EventBridge

***todo: diagram***

- a way to implement event-driven architecture in aws
- EventBridge is superset, newer service, aws promoting this and perhaps deprecating cw events
- both have same underlying architecture and similar functions
- what they do:
    - monitor event bus → pattern match / schedule match events → deliver to targets
- 1 default event bus per account (with EventBridge, you can create more)

## SNS

***todo: diagram***

- it's a public zone service, ha and scalable
- pub-sub architecture
- publishers publish messages to TOPICS, and subscribers receive those
- subscribers can be — https(s), SQS queues, lambda functions, etc.
- services can be publishers and subscribers at the same time, e.g. API
- can apply filter for subscribers
- msg size ≤ 256kb
- topics have topic policy — cross-account access
- has server side encryption
- delivery status, delivery retries

## Step Functions

- to address some limitations (design-decisions) of lambda
- lets you create serverless workflows (state-machines)
    - e.g. long running orderflow on amazon etc.
- max execution duration is 1 year
- workflow types: standard (1 yr) and express (highly transactional stuff, 5 mins)
- started via API gateway, event bridge, lambda, manually etc
- ASL (amazon states language) - json template
- type of states:
    - succeed & fail
    - wait (pauses the workflow for a given duration)
    - choice
    - parallel
    - map
    - task (single unit of work/action) — invokes services like lambda, etc.

## API Gateway

***todo: diagram***

- service to create and manage APIs
- sits between applications and integrations
- ha and scalable, handles auth, throttling, caching, cors, transformations, openAPI spec, direct integration with aws services, etc.
- public zone service
- http, rest, websocket apis
- req and res phases
- also integrates with cloudwatch
- types of auth methods supported:
    - cognito
    - lambda based auth (custom) — used for external id providers
- endpoint types
    - edge optimized — routed to nearest cloudfront pop
    - regional
    - private — within vpc
- stages: apis are deployed to stages. each stage has one deployment.
    - dev, prod
    - can have versions
    - stages can be enabled for canary deployments
- errors
    - 4xx — client errors
        - 400 - bad request
        - 403 - access denied
        - 429 - throttled
    - 5xx — server / backend errors
        - 502 - bad gateway
        - 503 - service unavailable
        - 504 - integration failure / timeout (29 seconds)
- caching — configured per stage
    - ttl default 300 seconds, min 0 and max 3600 seconds
    - can be encrypted
    - cache size 500 mb to 237 gb

## SQS

- public zone
- fully managed, ha
- two types: standard (best effort) or fifo (order)
    - standard ⇒ multi-lane highway ⇒ guarantees at-least-once delivery, but not order
        - very high perf, crazy scalable (nearly unlimited TPS)
    - fifo ⇒ single lane road ⇒ guarantees order and exactly-once delivery
        - low performance, 300 TPS without batching, 3000 with batching
        - must have ".fifo" suffix
- msg size ≤ 256kb, can link to large data
- received msgs are hidden (visibility timeout)
    - will get deleted by client or reappear in the queue
- dead-letter queues for problem msgs
- asg and lambda can work based on queue length
- **v v imp** — fanout architecture (e.g. multiple sqs subscribed to sns topic for master video upload, msg added to each queue, allowing multiple processing jobs to be started per object upload)
- billing based on requests — 1 request = 1-10 msgs upto 64kb total
    - long polling vs short polling
- encryption at rest (kms) and in transit
- identity policies and queue policies

## SQS Delay Queues

- initial period of invisibility for messages — no processing
    - 0 to 15 min delay
- not supported on fifo queues

## SQS Dead Letter Queues

- for problematic messages (processing failing repeatedly in main queue)
- different processing, diagnostic, logging methods can be used to identify faults
- retention period is generally longer than source queues, takes into account the enqueue timestamp isn't changed
