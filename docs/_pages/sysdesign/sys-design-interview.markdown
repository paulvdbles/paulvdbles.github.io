# System design zhiyong tan
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