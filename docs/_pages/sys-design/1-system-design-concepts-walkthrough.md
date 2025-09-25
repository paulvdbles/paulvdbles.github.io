# 1. A Walkthrough of System Design Concepts

System design interview: discuss approaches / trade off

Know the requirements

Scalability: to easily / cost effectively change resources allocated to respond to changes in load

## Example scaling the various services of a system
- Stateless, multiple back-ends
- GeoDNS to direct to closest DC
- Redis to serve cached requests
- Put files on CDN
- Idempotency
- Horizontal scaling
- IaaC
- Release: gradual rollout
- Logging, monitoring, alerting, search 
- Use span ids forlogging

- API Gateway: works but adds adds latency
- Alternative: service mesh (also called sidecar pattern). 
  - Use service mesh framework istio. 
  - Each pod contains 1 container service and 1 container sidecar
  - All service request/response go through sidecar
  - Benefit: no network latency
  - Downside: uses some system resources
- Sidecarless service mesh: logic in client host

- CQRS pattern: complex but lower latency and better scalability
- User auth
- Storage
- Async processing
- Notebook for analytics
- Search
- Privacy
- Fraud detection

Cloud vs DC:
- Cloud provides autoscaling and quality
- However also lock-in

Serverless: for fast startup use Graal VM