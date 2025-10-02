# A walkthrough of system design concepts
## A walkthrough of system design concepts
Discuss approaches and trade-offs

Know requirements

Scalability: ability to easily / cost effectively change resources allocated to respond to changes in load

Example of things to consider:
- stateless backends
- geodns to direct to closest dc
- redis: serve cached request from consumer app
- put files on CDN
- idempotency