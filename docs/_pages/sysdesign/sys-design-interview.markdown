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