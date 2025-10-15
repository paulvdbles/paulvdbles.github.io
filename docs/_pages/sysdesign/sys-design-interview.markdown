# A summary of Zhiyong Tan's system design
## 1. A walkthrough of system design concepts
Discuss approaches and trade-offs

Know requirements

Scalability: ability to easily / cost effectively change resources allocated to respond to changes in load

Example of things to consider:
- stateless backends
- geodns to direct to closest dc
- redis: serve cached request from consumer app
- put files on CDN
- idempotency

## 2. Typical system design interview flow
Keep principles in mind:
- Clarify functional and non-functional requirements
- Drive the interview
- Be aware of time
- Logging, monitoring, alerting, auditing
- Testing / maintainability
- Failure / graceful degradation
- Draw diagrams: flow chart, system diagram
- Design can always be improved

Possible order of interview:
1. Clarify requirements and discuss trade-offs
2. Draft API spec
3. Design data model / discuss analytics
4. Discuss topics like:
   - Failure design
   - Graceful degradation
   - Monitoring
   - Alerting
   - Bottle necks
   - Load balancing
   - Removing single point of failure
   - High availability
   - Disaster recovery
   - Caching
5. Discuss complexity and trade offs, maintenance, decommissioning process and costs

### Clarify requirements / discuss trade-offs
10 minutes to scribble down requirements
Tell interviewer:
- We want to ensure we capture all requirements
- But also mindful of time

Start:
- Overall purpose of system
- How does it fit into the big picture, business requirements?

1. Consider users / roles:
   - Who are the users? How do they use the system? 
     - Manual: real person via browser or app
     - Automated: other system calling endpoint
   - What are the roles of users?   
   - Clarify the numbers:
     - How many..
     - How often..
     - How much time..
   - Communication between users / ops needed?
   - Localization support: languages, addresses
2. Based on users, clarify scalability requirements:
   - Daily active users?
   - Daily / hourly requests?
   - Example: 1 billion users with each 10 searches means: 10 billion daily, 420 million hourly
3. What data is accessible to what user?
   - Discuss authentication + authorization, roles + mechanisms
   - Discuss content of response body API endpoint
   - How often is data retrieved?
4. What are use cases that involve search?
5. Discuss analytics requirements?
6. Scribble down pseudocode function signature

Always ask if there are other requirements and brainstorm them

Display awareness that requirements can change

### Draft API spec
Based on functional requirements, now draft API spec

Don't use a lot of time

Common endpoints:
- Health endpoints
- Sign-up (POST)
- Login (POST)
- Endpoints to CRUD user

### Draw the connections and processing between users and data
1. Phase 1: 
    - Draw users
    - Draw system that serves functional requirements (box)
    - Draw connections between users and systems
2. Phase 2:
   - Split request processing / storage
   - Based on non functional requirements make different designs. For example realtime consistency vs eventual consistency
3. Phase 3:
    - Break up into components, for example microservices
    - Draw connections
    - Consider logging, monitoring and alerting
    - Security
4. Phase 4:
   - Summary of our system design
   - New additional requirements
   - Analyze fault tolerance: what can go wrong?

### Design the data model
Problems with a shared database:
- Query's lock tables
- Schema migrations are more complex
- Various services using db restricted to using those specific db technologies

How do you handle users updating the same data concurrently?
- Lock data

### Logging, monitoring, Alerting
Monitoring: watching known problems
We monitor for bugs, degradations and unexpected events.

Observability: understand unknown problems, what is going on?

Google defines 4 golden signals of monitoring:
- latency
- traffic
- errors
- saturation

Instruments of monitoring:
1. Metrics
2. Dashboards
3. Alerts (with runbook)

Data quality jobs can check data anomalies

### Search bar
Multiple ways of implementing search:
- use SQL's LIKE
- search through data on client side
- ElasticSearch

Attach the search bar to GET endpoint (or POST?)

Elastic search definitions:
- Index: like db/collection where searchable data is stored
- Ingesting: adding to the index
- Mapping schema or blueprint of index, change to add field for example.

Create elastic index: ingest doc that should be searched when user submits query

Keep index up to date:
- Periodic indexing
- Indexing triggered by event
- Delete request with bulk API

Change mapping:
- Create news index, drop old
- Use elastic's reindex operation, but it's expensive

This table shows the terminology mapping between traditional SQL databases and Elasticsearch, which is helpful for understanding how concepts translate between the two systems.:
| SQL | Elasticsearch |
|-----|---------------|
| Database | Index |
| Partition | Shard |
| Table | Type (deprecated without replacement) |
| Column | Field |
| Row | Document |
| Schema | Mapping |
| Index | Everything is indexed |

Elastic can also be used as sql, but does not replace a db:
- No normalization
- No relations between tables (pk, fk)
- No ACID

### Other discussions
After creating design for requirements, discuss:
- Maintenance: features needed later, features we should deprecate
- Supporting other types of users
- Alternative architectural decisions
- Usability / feedback: metrics or NPS
- Edge cases and constraints
  - New requirements
  - How to support other formats
  - What if actions should be undone?
- Scalability and performance:
  - What if one users had one million followers?
  - Can we accept long P95 message delivery time?
- Latency throughput?
  - What if P99 message delivery needs to be within 500MS?
- Availability and fault tolerance
- Costs 

# Cloud native concepts
Address non functional requirements via:
- Micro services
- Service mesh
- Side car (istio)
- Containers
- Kubernetes
- Automation
- IaaC

# Reflection
Write one after the interview to improve

## 3. Non-functional requirements
Functional requirement: describe input and output of system. Can be represented by API endpoint

Non functional requirements: other than input and output, like
- Scalability: ability of system to adjust hardware usage, cost efficiently support load
- Availability: % of time system can accept requests and return desired response
- Performance/latency/P99: time for system to return response
- Throughput: current request rate processed by system
- Fault-tolerance: ability of system to continue operating if components fail, prevent permanent harm (data loss e.g.)
- Security
- Privacy: access control to PII data
- Accuracy: data may not need to be perfectly accurate, trade-off with cost/complexity
- Consistency: between nodes
- Cost
- Complexity, maintainability, debuggability, testability 

### Scalability
Scaling: increase CPU, RAM, storage capacity, network bandwith

Vertical scaling: upgrade resources (throw money)
Three disadvantages:
1. We will reach a point where cost increases faster than performance
2. Vertical scaling has technological limits
3. Vertical scaling may require downtime

Horizontal scaling: spread out processing and storage requirements across multiple hosts
Almost always discussed in interview

Questions to determine the customer’s scalability requirements:
- "How much data comes to the system and is retrieved from the system?"
- "How many read queries per second?"
- "How much data per request?"
- "How many video views per second?"
- "How big are sudden traffic spikes?"

HTTP is stateless: backend service using is is easy to scale horizontally

Starting point: stateless HTTP backend combined with horizontally scalable database read operations

Writes to shared storage are most difficult to scale.

Horizontally scaled services use load balancer. One of following:
- Hardware: distribute traffic across multiple hosts
- Shared load balancer service: LBaaS
- A server with software: HAProxy and NGINX

Tell load balancer is implied, don't draw

Level 4 vs 7: 
- level 4 operates at TCP layer: routes based on address info from first few packages in TCP stream
- level 7 operates at HTTP layer so has more capabilities:
  - load balancing/routing decisions based on packet content
  - authentication
  - TLS termination

Sticky session: load balancer sending requests from 1 client to 1 host for duration set by load balancer or app
Implemented with cookies.

Session replication: write to a host are copied to other hosts, so reads can be routed to any host in that session. Improves availability

Load balancer vs reverse proxy: lb is for scalability, reverse proxy is a technique to manage client-server communication

### Availability

99.9: 8.77 hours per year

Techniques for high availability are replication within dc's and across data centers in different continents

Other non-functional requirements may be traded off for high availability: strong consistency, low latency

If possible use async communication: for some services..
- requests don't need to be processed immediately 
- response doesn't need to be returned immediately

Use synchronous communication when an immediate response is needed
Immediate response can also be acknowledgement, with actual data returned 5 minutes later

Example where high availability may not be required: caching service (trade off for lower latency?)

### Fault tolerance
Self healing mechanisms

Closely related: failure design. Smooth error handling.

Replication: have multiple redundant instances of component, so they can go down without addecting overall optime

Forward error correction (FEC) is a method that protects data during transmission through noisy or unstable communication channels by adding redundancy to the message through techniques like error correction codes (ECC).

Circuit breaker: stop client from repeatedly attempting operation that is likely to fail
CB makes system more difficult to test
Can be implemented on server side by resilience 4j

Exponential backoff and retry: similar to circuit breaker. When client receives error, wait before retry

Caching responses of other services: if stale data is better than no data. Or leave out data if possible

Checkpointing: enables fault tolerance in data processing by saving progress after each batch of data is successfully processed. If a machine fails, its replacement can resume from the last checkpoint instead of restarting. This is commonly used in ETL pipelines with message brokers like Kafka to ensure reliable, resumable data processing.

Dead letter queue: 
- if write request to third party API fails, put request in DLQ and try later
- put requests that can't be processed

Queue locally or separate service?
- simplest: if acceptable to miss a request, drop it
- also simple: local try catch block, if host fails it's gone though
- complex: something like kafka

Log silent errors and perform periodic auditing

Bulkhead: fault tolerance mechanism where system is divided into isolated pools, one faulty pool will not affect the entire system

- Example 1: various endpoints have their own thread pool and will not share a thread pool.  If one endpoint's pool is exhausted it will not affect the ability of other endpoints to serve requests
- Example 2: if one feature has a bug that causes the enitre system to crash, dividing service into bulkheads prevents crashing all instances
- Example 3: one requestor has high request rate and prevents service from serving other requests

Fallback pattern: detect problem and execute alternative code path:
- Cached response
- Call alternative service that is similar to the service client is trying to get information from]

### Performance/latency and throughput
Performance/latency: time taken for system to return response to a user's request 

Typical request on consumer facing app: desired latency of 10 ms - several seconds
High frequency trading application: several ms

Design decisions to achieve low latency:
- deploy close to users
- if users are scattered: deploy in multiple dcs that are chosen to minimize distance to clusters of users
- if hosts across dcs need to share data: service must be horizontally scalable
- CDN
- Caching
- decrease data size with RPC instead of REST
- design own prototcol to use TCP and UDP instead of HTTP
- Use batch en streaming techniques

### Consistency
ACID:
- focuses on data relationship like foreign keys and uniqueness

CAP
- "consistency is actually linearizability, defined as all nodes containing the same data at a moment in time, and changes in data must be linear; that is, nodes must start serving the changes at the same time."

Consistency is a trade-off for improvements in:
- availability
- scalability
- latency

Favor linearizability: HBase, MongoDB, Redis
Favor availability: Cassandra, CouchDB, Dynamo, Hadoop, Riak

During discussion:
- ACID vs CAP consistency
- Tradeoffs between linearizability vs. eventual consistency

Techniques for linearizability and eventual consistency:
- Full mesh
- Quorum

"Techniques for eventual consistency that involve writing to a single location, which propagates this write to the other relevant locations:"
- Event sourcing (section 5.2), a technique to handle traffic spikes.
- Coordination service.
- Distributed cache.

"Techniques for eventual consistency that trade off consistency and accuracy for lower cost":
- Gossip protocol.
- Random leader selection.

Disadvantages of linearizability include the following:
- "Lower availability, since most or all nodes must be sure of consensus before they can serve requests. This becomes more difficult with a larger number of nodes."
- More complex and expensive.

Full mesh: every host in cluster has address of other host, send messages to all of them. easy to implement, not scalable

Coordination service: 
- third party component chooses leader node. Having leader decreases nr of messages
- All other nodes send message to leader
- Leader does processing and send back final result
- each node communicates with its leader, leader manages nr of nodes

Distributed cache:
- like redis or memcached
- our service's nodes can make periodic requests to the origin to fetch new data, then make requests to the cache to update its data
- simple, low latency and distributed cache can be scaled independently of service
- has more requests than other solutions, except full mesh

Gossip protocol: 
- like how epidemics spread. node randomly selects other node and shares data
- used by cassandra and dynamodb (vector clocks)

Random leader selection:
- simple algorithm to elect leader, no guarantee for 1 leader
- leader can share data with all hosts
- disadvantage: duplicate requests possible
- used by kafka and yarn

### Accuracy
- Correct and not approximations 
- Relevant in systems with complex data processing or high rate of writes
- Estimation algorithm is less accurate but also less complex: HyperLogLog, Count-Min Sketch

Stale cache: data in underlying db already modified

Short refresh policy is expensive

Alternative to keep data up to date: update or delete an associated cache key when data is modified. More complex

Accuracy can be traded off. Eventually consistent systems do a trade off with:
- availability
- complexity
- cost

When distributed systems don't immediately reflect recent writes across all replicas: issue of "consistency" rather than "accuracy" (even though reads may return outdated/incorrect data)

In eventually consistent systems, there's a temporary period after a write where different replicas have different values, causing reads to potentially miss recent changes. 
- Consistency: the state of synchronization between replicas
- Accuracy: would imply the data itself is wrong rather than just temporarily out-of-sync.

### Complexity and maintainability
- First step: clarify requirements so we don't design for unneeded requirements
- Which components can be separated into independent systems?
- Use common services like:
  - Load balancing
  - Rate limiting
  - Authentication & authorization
  - Logging, monitoring, alerting
  - TLS termination
  - Caching
  - DevOps and CI/CD

System is unavoidable complex: trade-off complexity for lower availability and fault-tolerance

Discuss are other tradeoffs in requirements we can do to combat complexity
For example: ETL pipeline to delay data processing operation that is not needed in real time

Trade off complexity for better latency and performance: minimize message size (RPC)
When going for RPC suggest serialization framework: AVRO, THRIFT, ProtoBuf

Discuss how to do outage

CD: allow easy deployment and rollback
Fast feedback cycle improves maintainability

SonarQube

### Cost
Trade off other non functional requirements with costs:
- Higher cost for lower complexity: vertical scaling instead of horizontal scaling
- Lower availability for improved cost: decrease redundancy of system
- Higher latency for improved costs: using data center in a cheap location, further from users

Implementation, monitoring, each non-fun requirement like high availability can impact cost.

Mindful of problem severity when you alert engineers on call to save costs

Cost of replacing dependencies

### Security
- TLS termination vs keeping data encrypted between services or hosts in a dc
- Which data can be stored unencrypted? Encrypted at rest vs storing hashed data

OAuth 2.0 and OpenID connect

Discuss rate limiting to prevent DDoS attack

### Privacy
PII: data that can be used to identify a user

Use access control on PII data, like LDAP

We can encrypt data both in transit (SSL) and at rest

Technique for append only db or filesystem like HDFS:
- Each customer gets encryption key
- Encryption key can be stored in a mutable storage system like SQL
- Encrypt this customers data using their key before its stored
- Customer needs to be deleted? Delete key so all data becomes inaccessible

External vs internal service
- External: definitely design for security and privacy
- Internal: 
  - may rely on security mechanisms of our user agents 
  - may decide internal users will not attempt malicious actions
  - so for designing rate limiter service you can choose to not discuss sec/privacy measures
  - better: engineering culture of implementing sec/privacy by default

Have policy for storing user data. Databases storing user data should have tight security / strict access control policies

### Cloud native
Approach to address non functional requirements like scalability, fault-tolerance and maintainability

"Cloud native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach.

These techniques enable loosely coupled systems that are resilient, manageable, and observable. Combined with robust automation, they allow engineers to make high-impact changes frequently and predictably with minimal toil."

Utilize cloud native techniques:
- containers
- service meshes
- microservices
- serverless functions
- immutable infrastructure or Infrastructure as Code
- declarative APIs
- automation
For benefits: resilient, manageable, observable, allow frequent and predictable changes

## 4. Scaling databases
Storage services are stateful service: mechanisms to ensure consistency nd require redundancy to avoid data loss

Types of storage:
- Database
  - SQL: tables and relationships. SQL must have ACID properties
  - NoSQL: a db that has not all SQL properties
  - Column oriented: organize data into columns, like hbase
  - Key-value: Data is stored as colleciton fo key value pairs, Key corresponds to disk location via hashing algorithm.
    Read performance is good, keys are hashable
- Document: can be interpreted as key-value database where values have no size limits. Value can be text, json or yaml for example
- Graph: efficiently store relationships between entities. Example is Neo4J
- File storage: data stored in files, can be organized into directories/folders
- Block storage: stores data in evenly sized chunks with unique identifiers. Not used a lot for web applications, more for low level components of other storage systems like dbs
- Object storage: Flatter hierarchy than file storage. Objects are accessed with simple HTTP API. Writing object is slow, and object annot be modified so only for static data. For example aws S3

Database or other way to store data?
Could also use file, block, object storage. 
During you can have preference but should be able to discuss relevant factor and consider others

Decision is based on discretion and heuristics
- Discretion: There's no single "correct" answer that applies universally. The developer uses their judgment based on the specific situation, requirements, and context of the project. It's a subjective decision that different experienced developers might make differently.
- Heuristics: These are practical rules of thumb or guidelines that developers have learned through experience, rather than strict formulas. For example:
  - "If you need complex queries and relationships between data, use a database"
  - "If you're storing large binary files like images or videos, use the filesystem"
  - "If multiple users need concurrent access with transactions, use a database"
  - "If the data structure is simple and won't change much, files might be sufficient"

### Replication
Scaling a database: implement a distributed db onto multiple nodes
Scaling happens via replication, partitioning and sharding
- Replication: Making copies of data, storing them on different nodes
  - Purpose: redundancy, high availability, read performance 
- Partitioning: is dividing a large dataset into subsets called "partitions" based on certain criteria. Partitions can all exist on the same node.
  - Purpose: Organization, query performance, maintenance
- Sharding: Divide data in subset + distribute across nodes
  - Purpose: horizontal scaling, handling data too large for one server


Single host has limitations and cannot fulfill our requirements:
- Fault tolerance: nodes back up data on other nodes
- High storage capacity: single node can be vertically scaled but that's expensive and nodes throughput can become problem
- Higher throughput: database needs to process reads and writes for multiple processes / users
- Lower latency: we can geographically distribute replicas to be closer to users

Scale reads (query's with SELECT): increase number of replicas
Scale writes: more difficult

Typical design: 
- one backup onto a host on the same rack
- one backup on a host on a different rack or DC or both

Sharding data gives benefits:
- Scale storage: db too big for one node? sharding across nodes helps
- Scale memory: db stored in memory needs to be sharded because vertical scaling of memory on single node is expensive
- Scale processing: process in parallel
- Locality: we can shard so that data needed by particular cluster node is stored closer than on another shard
Trade off is increased complexity (need to track shards locations)

Single leader replication: 
- all write operations occur on a single node (leader)
- Scaling reads, not writes
- SQL service loses its ACID consistency
- Simplest to implement

Multi leader replication:
- scale writes and db storage size
- Require handling race conditions, which are not present in single leader replication
- Each leader must replicate its writes to all other nodes

Replication introduces consistency and race conditions for operations where sequence matters

Consistency: 
- ensure a db transaction brings the db from one valid state to another
- any data written to the db must be valid according to rules

Look for ways to relax consistency requirements.

Leaderless replication: all nodes equal
Reads and writes can occur on any node

HDFS replication: does not fit into any of these three approaches

### Scaling storage capacity with sharded databases
If the db size grows to exceed the capacity of a single host we need to delete old rows
Retain old data: store in sharded storage: HDFS or Cassandra
Or sharded RDBMS

### Aggregating events
Database writes are difficult and expensive to scale, try to reduce the rate of db writes wherever possible
Techniques:
- Sampling: consider only certain data points, ignore others
- Aggregation: combine multiple events into single. Can be implemented using streaming pipeline

Reduce reads:
- Caching
- Approximation

Single-tier aggregation
Multi-tier aggregation

The main tradeoffs of aggregation are eventual consistency and increased complexity. 

Fault tolerance: host goes down? loses all of its aggregated events

### Batch and streaming ETL
ETL: extract transfer load, copy data from 1 to more sources
Batch: 
- process in batches, periodically or manual
- use batch tools like airflow and luigi 

Streaming:
- continuous flow, processed in real time
- use streaming tools Kafka and Flink

ETL pipeline consists of DAG of tasks

Messaging terminology:
- messaging system: transfer data from one app to another
- message queue: message goes through queue, wait to be processed
- producer/consumer: aka publisher/subscriber aka pub/sub is async messaging sys
- message broker: 
  - program that translates a message from formal protocol of sender to formal protocol of receiver
  - Kafka and RabbbitMQ are message brokers
  - AMQP is protocol implemented by RabbitMQ
- event streaming: continuous flow of events, processed in real time
- pull vs push
  - in general pull is better: consumer controls the rate of message consumption (will not be overloaded)
  - load/stress test to determine throughput and performance. monitor throughput and queue size of messages
  - situations where push is better: crawlers or UDP audio video streaming (failed messages are not resent)

Kafka vs RabbitMQ:
- companies use shared kafka service
- both:
  - smooth out uneven traffic
  - prevent overload by spike
  - no need for many hosts to handle periods of high traffic: cost efficient
- different:
  - kafka is more complex but more capabilities
  - rabbitmq is not scalable by default
  - kafka has replication
    - kafka events are not removed after consumption: same event can be consumed repeatedly. Set retention period to decide when its deleted
    - rabbitmq messages are removed upon dequeueing
    - Kafka has no concept of priority, rabbitmq has

if rabbitmq is sufficient we can use it

Lambda architecture:
- processing big data running batch and streaming pipelines in parallel
- parallel fast and slow pipelines
  - fast pipeline achieves speed by:
    - approximation algo
    - in memory db (redis?)
    - no replication
  - slow pipeline:
    - mapreduce db (hive, spark with hdfs)
- update same destination
- trade offs:
  - fast pipeline trade off consistency and accuracy for lower latency
  - slow pipeline vice versa

Use lambda architecture for big data systems that need consistency and accuracy

Database solutions:
- Common: SQL, Hadoop, HDFS, Kafka, Redis, Elasticsearch
- Less common: Mongo, neo4j, aws dynamo, google firebase

Kappa architecture as alternative to Lambda architecture:
- architectural pattern 
- focuses on handling streaming data and combines both real-time stream processing and batch processing 
- within a unified technology stack.
  - Example of kappa: use apache kafka + apache flink to process live events while they arrive + historical data by replaying topic from beginning
  - Example lambda: apache storm for real time + apache hadoop/spark for batch

### Denormalization

Normalization: breaking down tables into smaller, related tables to reduce redundancy and improve data integrity.
Benefits:
- consistent
- no duplicate data
- insert and update faster: only one table has to be queried (denormalized schema insert/update may query multiple tables)
- smaller db size
- fewer columns -> fewer indexes -> faster index rebuilds
- queries can join only tables that are needed

Disadvantage of normalization:
- slow joins
- tables contain codes rather than data -> queries contain joins -> joins are more verbose to write and maintain

### Caching 
Cache frequent or recent queries in memory

Caching should improve:
- performance
- availability
- scalability

Caching can be done at many levels: client, api gateway, each service

Read strategies: optimized for fast reads

- Cache aside: cache sitting aside of the db. Application first makes request to cache:
  - data hit: return data
  - cache miss: application makes read request to db. then writes data to cache (lazy load)
  - is best for read heavy loads
  - advantages:
    - minimizes the nr of read requests and resource consumption
    - only requested data will get into the cache
    - simple
  - disadvantage:
    - the cache can become stale. we set a ttl or use write-through
    - a reuqest with cache miss is slower than directly to db (db read + cache write)
- Read through: application does not contact db: implementation of db requests goes from app to cache
  - best for read-heavy loads
  - cache miss: cache makes request to db and stores data in cache, then returns to app
  - trade off: multiple db requests can't be grouped as a single cache value (cache aside can)

Write strategies: optimized to minimize stale cache, but higher latency + complexity
- Write through: every write goes through cache, then to db
- advantage:
  - consistent. cache can't go stale
- disadvantage:
  - slow write: every write is done on both cache and db
  - cold start with new cache node , use cache-aside to resolve
  - most data is never read: use ttl to lower cost and reduce space
  - In a write-through cache, every write operation updates both the cache AND the database simultaneously. This creates a specific challenge when your cache has limited space 
- write-back/write-behind: app writes data to cache, but cache not immediately to db
  - cache flushes updated data to db in intervals
  - advantage:
    - faster write than write through
  - disadvantage:
    - same as write-through, other than slow writes
    - more complex: cache must be highly available
- Write-around: 
  - app only writes to db
  - updates cache on cache miss

### Cache as a separate service
Why is it a separate service and not in memory of a service's host?
- service is stateless -> request randomly assigned to host -> each host would have different cache data
- caching is useful when request patterns lead to hot shards
- cache would be wiped on deployment
- scale cache independently
- coalescing: many clients make same request at the same time -> our db service will execute same query. cache deduplicates request and sends a single request to service

Cache on client

Consider cdn

### Data to cache
Cache either:
- http response: cache body, retrieve using cache key (HTTP method and URI of request)
- db queries: use cache-aside to cache relational db queries

pub vs private:
- private: on a client, for personalized content
- public: on a proxy like CDN or our services

Don't cache: private info, real time info, copyright

Info that can cache may but cache, should be validated against server

### Cache invalidation

Cache invalidation: replace/remove entries
Cache busting: cache invalidation for files

Browser: set max age

fingerprinting

On client: use policies like random replacement, LRO, FIFO, LIFO

### Cache warming
Already fill cache before cache miss can occur
Applies to: CDN, front end, backend. not to browser

Advantage: first request will have same low latency as next ones
Disadvantage: complex, more traffic from querying service to fill cache, if we have billion users only 1 will have slow experience, cache items could expire before they are used

## 5. Distributed transactions
System writes data to other service. 
Failure could lead to data inconsistency across services.

Transaction:
- group several reads/write into a logical unit to maintain data consistency across services
- single operation, executed atomically
- commit: transaction succeeds
- abort/rollback: transaction fails
- ACID properties

If possible, we should use event streaming (like kafka) for writes with services pulling them
Otherwise use distributed transaction

Distributed transaction: combine separate write requests as a single distributed (atomic) transaction

Consensus: all systems agree that write happened or did not happen

Patterns for maintaining consistency in distributed transactions:
- event sourcing, change data capture and event driven architecture
- checkpoint and DLQ
- SAGA
- two phase commit

Two-phased commit and saga: achieve consensus
Others: make one db source of truth when inconsistency happens from failed write

### Event driven architecture

Event driven architecture:
- announce events that already happened (instead of work to be done)
- async and non blocking
- loose coupling, scalability and responsiveness / low latency
- eventual consistency

Alternative to EDA: service makes request to another service. Expensive, complex, error-prone and less scalable
- unavailability or slow performance of either service means that the overall system is unavailable
- consumes thread in each service
- traffic spike could overwhelm service
- requestor needs to keep thread until request completed
- traffic spikes can be solve with auto scaling / rate limiting

However this alternative is:
- more consistent (operation completes before returning response)
- EDA has lower latency for the initial response, Direct calls have lower latency for the complete operation

Less resource intensive: event log

We can choose to not follow non-blocking philosophy of EDA (skip request validation)

### Event sourcing
Event sourcing: pattern for storing data as events in append-only log

1. first make write to event log
2. write succeeds
3. event handlers consume new event and write to other db

Can be implemented in multiple ways: kafka topic, row in db, document in mongo, or even in memory db

Complex: manage event stores, replay, versioning and schema updates

### Change Data Capture (CDC)
Change Data Capture (CDC):
- log data change events to change log event stream
- provide event stream via API
- consistency and lower latency than event sourcing (request processed near rt)

Transaction log tailing pattern: another pattern against inconsistency:
- process needs to write to db
- and produce to kafka
one of those can fail -> inconsistency

### Comparison of event sourcing and CDC
Both used in distributed systems to propagate data changes
Decouple services by using async communication

Differences between event source and CDC:

| Aspect | Event Sourcing | Change Data Capture (CDC) |
|--------|---------------|--------------------------|
| **Purpose** | Events are the primary record | Sync data changes between services |
| **Source of Truth** | Event log itself | Publisher's database (events are secondary) |
| **Granularity** | Business-level actions/state changes | Database-level operations (insert/update/delete) |

**Quick Summary:**
- **Event Sourcing**: The events ARE your data - rebuild state by replaying events
- **CDC**: The database is your data - events just notify others about changes

### Transaction supervisor
Ensures a transaction is successfully completed or compensated
Periodic batch job or serverless function

First implementation: interface for manual review
Automating risky, only after extensive testing

### Saga
SAGA:
- Pattern to manage failure
- Long-lived transaction 
- Can be written as sequence of transactions
- All must complete successful or roll back

Typical implementation: services communicate through message broker (kafka or rabbbitmq)

Only do distributed transaction if certain services satisfy certain requirements

Saga Example: Travel Package Booking

| Component | Description |
|-----------|-------------|
| **Scenario** | Book complete tour package (flight + hotel + payment) |
| **Services Involved** | 1. Airline Ticket Service<br>2. Hotel Room Service<br>3. Payment Service |
| **Success Condition** | ALL services must succeed (flight available AND room available AND payment processed) |
| **Failure Handling** | Any failure triggers rollback in reverse order using compensating transactions |

### Why Separate Payment Service?

1. **Timing Control**: Don't charge customer until flight + hotel confirmed
2. **Privacy/Security**: Don't share customer payment info with third-party services
3. **Business Model**: Our company handles customer payment, then pays vendors

### Saga Flow

```
Success: Airline ✓ → Hotel ✓ → Payment ✓ → Complete
Failure: Airline ✓ → Hotel ✗ → Rollback Airline → Abort
         Airline ✓ → Hotel ✓ → Payment ✗ → Rollback Hotel → Rollback Airline → Abort
```

Key Takeaway: Sagas enable distributed transactions across multiple services with automatic rollback capability when any step fails.

Structure coordination:
- choreography (parallel) 
- orchestration (linear)

| Aspect | **Choreography** | **Orchestration** |
|--------|-----------------|-------------------|
| **Design Pattern** | Observer pattern | Controller pattern |
| **Execution Model** | Parallel requests | Linear/sequential requests |
| **Coordinator** | Initiating service (minimal role) | Orchestrator service (central control) |
| **Communication** | Services talk directly to each other | All services talk only through orchestrator |
| **Initiator Topics** | 2 topics:<br>• Produces to start saga<br>• Consumes final result | 2 topics per service:<br>• Produces requests<br>• Consumes results |
| **Service Subscriptions** | May subscribe to multiple topics<br>(must track consumed events in DB) | Each service subscribes to only 1 topic<br>(simpler, fewer DB writes) |
| **Code Understanding** | Must read ALL services to understand saga flow | Read orchestrator only to understand flow |
| **Development Independence** | Less independent (changes affect multiple services) | More independent (changes only affect orchestrator) |
| **Performance** | • Lower latency (parallel)<br>• Less network traffic<br>• Fewer events | • Higher latency (sequential)<br>• More network traffic<br>• 2x events (through orchestrator) |
| **Single Point of Failure** | None (distributed) | Orchestrator (must be highly available) |
| **Rollback Trigger** | Individual services trigger compensations | Orchestrator triggers all compensations |

Choose Choreography when:
- Performance/latency is critical
- Services can handle complexity
- Team can maintain distributed logic

Choose Orchestration when:
- Centralized visibility needed
- Service independence important
- Simpler service logic preferred

Key Difference: Choreography = distributed control, Orchestration = centralized control

# 6 Common services for functional partitioning
Functional partitioning: scalability technique to partition out backend functions to run on their own clusters

## Common functionalities of various services
Services can share non functional requirements
Implement for each service: duplicate work
So we can use libraries
Better solution: centralize in API gateway

API Gateway: 
- lightweight, service, stateless machines, across DCs
- Functionality:
  - Security: authentication, authorization, SSL termination, server side data encryption
  - Error checking: request validation, request deduplication
  - Performance / availability: caching, rate limiting, request dispatching
  - Logging / analytics

## Service mesh/sidecar pattern
Service mesh can combat the disadvantage of API gateway:
- additional latency in each request
- large cluster of hosts, requires scaling to control costs
Disadvantage:
- service unavailable if sidecar is unavailable


## Metadata service
Stores info used by multiple components within system
Components pass ids to each other, then request metadata service for data based on id
less duplicate data (like sql normalization)
Example: ETL

## Service discovery
Typically:
- each internal service is assigned port number to access
- each external service/ui serivce is assigned assigned url


## Functional partitioning and various frameworks
Purpose of web server app, example:
- browser downloads browser app from node.js
- when browser makes url request with specific path node.js handles routing and serves right page
- url may include certain path / query params. node.js app makes appropriate backend requests
- certain user actions may require multiple backend requests: node.js app exposes own api to browser app.
- user action -> browser app -> api request to node.js app/server -> one or more backend requests

why doesn't browser directly call backend? 
- data returned by api endpoint may not be exactly what is required
- multiple calls over internet between device and dc is inefficient

graphql: allow user to request just what is needed, but harder to secure than rest

Browser app frameworks: react, vue.js, angular

Server side frameworks: express, deno, goji, rocket, vapor, vertx, php

Mobile app dev: 
- native android: kotlin / java
- native ios: swift / objective c

cross platform: react native, flutter, ionic, xamarin, electron, PWA

backend development frameworks: gRPC, thrift, protocol buffers, dropwizard, flask, django

## Library vs. service
Determine components and discuss pros and cons of implementing these on client or server side

| Aspect | Library | Service |
|--------|---------|---------|
| **Version Control** | Users choose version/build and control upgrades. Risk: may use outdated versions with bugs/security issues | Developers control build and upgrades centrally |
| **Data Sharing** | Limited - no communication between devices. Users must implement host-to-host communication themselves | Built-in data synchronization via requests or database. Transparent to users |
| **Language** | Language-specific | Technology-agnostic |
| **Latency** | Predictable | Less predictable (network-dependent) |
| **Behavior** | Predictable and reproducible | Less predictable due to network issues |
| **Scalability** | Must scale entire application. User bears costs | Independently scalable. Service provider bears costs |
| **IP Protection** | Code can be decompiled | Code not exposed to users |

Key takeaway: Libraries offer more user control and predictable performance but require more implementation work and have language constraints. Services provide centralized management and technology flexibility but depend on network reliability.

## Common API paradigms

OSI Model Layers

| Layer | Name | Description | Examples |
|-------|------|-------------|----------|
| 7 | Application | User interface | FTP, HTTP, Telnet |
| 6 | Presentation | Presents data. Encryption occurs here | UTF, ASCII, JPEG, MPEG, TIFF |
| 5 | Session | Distinction between data of separate applications. Maintains connections. Controls ports and sessions | RPC, SQL, NFX, X Windows |
| 4 | Transport | End-to-end connections. Defines reliable or unreliable delivery and flow control | TCP, UDP |
| 3 | Network | Logical addressing. Defines the physical path the data uses. Routers work at this layer | IP, ICMP |
| 2 | Data link | Network format. May correct errors at the physical layer | Ethernet, wi-fi |
| 1 | Physical | Raw bits over physical medium | Fiber, coax, repeater, modem, network adapter, USB |

Key notes: Actor, GraphQL, REST, and WebSocket are built on HTTP (Layer 7). RPC operates at Layer 5 as it directly handles connections, ports, and sessions. Each layer's protocols are implemented using lower-level protocols.

RPC
- remote procedure call, make procedure execute
- encoding: CSV, XML, JSON, Thrift, protobuf, avro
- advantage over rest:
  - resource optimization. choose for low power devices or scaled large web service
  - protobuf is efficient, json repetitive and verbose
  - devs define schemas

graphql:
- declarative data fetching
  - client decides what it wants / what format
  - server can deliver efficiently what is requested
- tradeoffs are: complexity, learning curve, smaller user community, encodes json only, analytics more complicates, cautious with security

Websocket:
- communication over persistent TCP connection 
- new connection every time
- websocket is a protocol, like http.
- unlike rest, rpc, graphql: these are design patterns / philosophies
- stateful (open connection)
process:
1. client sends request to server
2. http handshake + request server to use websocket
3. next messages use websocket over tcp
4. open connection, same host needs to handle request (in rest any host should be able to handle)

Compared:
- REST: startup use REST because simple
- RPC: large companies can benefit from efficiency and compatibility
- GraphQL: new
- Websocket: bidirectional communication