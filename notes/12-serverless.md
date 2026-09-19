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
