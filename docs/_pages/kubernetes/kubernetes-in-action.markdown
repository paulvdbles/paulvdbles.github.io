# Notes on Kubernetes in action

## Intro to docker
In the past we used VMs to run applications, but those have a big overhead (full OS running)

Containers perform system calls on exact same kernel running in host OS

How can we isolate containers?
- Linux namespaces
- Control groups: limited resources available to a process

Docker file is baked into docker image

Docker image: contains filesystem, OS, your app code, all application dependencies (libraries, runtimes, etc.)

Docker image is pushed to registry

Registry contains images. Others can pull image from registry

Image layers: Images are built in layers for efficiency and caching

Containers are created from Docker images: process running on host running docker, isolated, resource constrained

Constraint: containers must match host's CPU architecture

Docker uses containerd/runc under the hood

## Intro to kubernetes
Kubernetes helps to easily deploy/manage apps running in linux containers

Apps run in Pods (smallest deployable unit), not directly in containers.

Like OS for cluster. It provides:
- service discovery
- load balancing
- self healing

Kubernetes nodes have two types:
- Master: hosts control plane
- Worker: actual apps

Control plane:
- kube-apiserver: The core component server that exposes the Kubernetes HTTP API.
- etcd: Consistent and highly-available key value store for all API server data.
- kube-scheduler: Looks for Pods not yet bound to a node, and assigns each Pod to a suitable node.
- kube-controller-manager: Runs controllers to implement Kubernetes API behavior.
- cloud-controller-manager (optional): Integrates with underlying cloud provider(s)

Node Components:
- kubelet: Ensures that Pods are running, including their containers.
- kube-proxy (optional): Maintains network rules on nodes to implement Services.
- Container runtime: Software responsible for running containers. Read Container Runtimes to learn more.

Benefits of kubernetes:
- Simplify deployment
- Better hardware utilization
- Health check: reschedule unhealthy pods
- Auto scaling
- Simplify development

## First steps with Docker and Kubernetes
Docker images are built from stacked, reusable layers
Each layer represents a single instruction in the Dockerfile

Example:
Imagine you have three applications:
1. App A: needs Ubuntu + Python
2. App B: needs Ubuntu + Python + Flask
3. App C: needs Ubuntu + Python + Django

Instead of storing Ubuntu and Python three times:
- Ubuntu is stored as one base layer
- Python installation is another layer on top
- Apps A, B, and C all reference these same layers
- Only Flask (for B) and Django (for C) are stored as unique layers

Running a Hello world container with Docker:
> docker run busybox echo "Hello world"

Running other images:
> docker run <image>

Versioning container images:
> docker run <image>:<tag>
 
Build from Dockerfile:
> docker build -t kubia . 

Listing locally stored docker images
> docker images

Run image:
> docker run --name kubia-container -p 8080:8080 -d kubia

List all running containers:
> docker ps

Listing cluster nodes with kubectl:
> kubectl get nodes

Run image:
> kubectl run kubia --image=luksa/kubia --port=8080 --generator=run/v1

A pod contains one or more containers.

To make the pod accessible from the outside, we need to expose it through a Service object.
For example: LoadBalancer

## Pods
A single pod always runs on exactly one node.

Containers are designed to run only a single process per container (unless the process itself spawns child processes)

By default, the filesystem of each container is fully isolated from other containers.

We should split multi-tier apps into multiple pods (don't put frontend and backend in same pod)
- Splitting a single pod into two separate pods allows Kubernetes to distribute them across different nodes, enabling better utilization of all available cluster resources instead of leaving nodes idle.
- Also consider scaling: Kubernetes scales at the pod level, not container level, so scaling a multi-container pod creates duplicate sets of ALL its containers (e.g., scaling a pod with frontend + backend containers to 2 instances gives you 2 frontends AND 2 backends).

We can create pods from YAML descriptors: 

1. metadata:
    - name: unique pod identifier
    - namespace: where pod lives
    - labels: key-value pairs for organization/selection
    - annotations: non-identifying metadata (notes, timestamps)

2. spec (desired state - what you define):
    - containers: list of container images, ports, commands
    - volumes: storage definitions
    - restartPolicy: Always/OnFailure/Never
    - nodeSelector: node placement constraints
    - resources: CPU/memory requests and limits
    - imagePullSecrets: for private registries

3. status (current state - Kubernetes populates):
    - phase: Pending/Running/Succeeded/Failed/Unknown
    - conditions: PodScheduled, Ready, Initialized, ContainersReady
    - containerStatuses: running/waiting/terminated state per container
    - podIP: assigned IP address
    - startTime: when pod started
    - reason/message: error or status information

Note: You only write metadata and spec; Kubernetes manages status automatically.

## Liveness probe
If the container’s main process crashes, the Kubelet will restart the container

Bug that causes main process to crash? Kubelet will restart

What if app stops working without crash: Java app with a memory leak will start throwing OutOfMemoryErrors, but the JVM process will keep running

We should check the health of an app from the outside: liveness probe

3 mechanisms:
- HTTP GET
- TCP Socket
- Exec probe

Spring Boot offers Actuator module. Exposes three endpoints:
- Liveness probe endpoint: The application maintains a LivenessState that can be either CORRECT or BROKEN
- Readiness probe endpoint: Whether the application is ready to handle traffic. The application maintains a ReadinessState that can be either ACCEPTING_TRAFFIC or REFUSING_TRAFFIC.
- Overall health endpoint: Health Indicators: Spring Boot aggregates various health indicators (database, disk space, etc.)

## Replication controller
Ensures pod is always running: creates/deletes pods to match desires count
- Label selector
- Replica count
- Pod template

# ReplicaSets
Next-generation ReplicationController with better pod selection
Same purpose as ReplicationController: ensures desired number of pods are always running

Key components:
- Label selector (supports set-based selectors, more flexible than ReplicationController)
- Replica count
- Pod template

Continuously monitors pods matching label selector:
- Too few pods → creates new ones from template
- Too many pods → deletes excess

Rarely create ReplicaSets directly - Deployments manage them:

- Each ReplicaSet represents one version of pod template
- Deployment creates new ReplicaSet when updating (e.g., new image)
- Old ReplicaSets kept for rollback history

## Job 
"allows you to run a pod whose container isn’t restarted when the process running inside finishes successfully."

## Services:
Service enables clients to discover and talk to pods

It's the single point of entry to group of pods

Has ClusterIP (internal IP) + port that remains stable for the service's lifetime

Front-end needs service so people can find it

Back-end also needs service so front-end can easily find it

Connections to pods are load balanced

Services select pods using label selectors

Specify available port and target port:
- port: 80 (service will be available on port 80)
- targetPort: 8080 (routes each connection to port 8080 of pod)

Service Discovery:

**Environment Variables:**
- Automatically injected for services that exist BEFORE a pod starts
- Format: `<SERVICE_NAME>_SERVICE_HOST` and `<SERVICE_NAME>_SERVICE_PORT`
- Services created after a pod starts won't have env vars in that pod

**DNS:**
- FQDN format: `<service-name>.<namespace>.svc.cluster.local`
- Short name `<service-name>` works within same namespace

Service Types:

**ClusterIP (default):**
- Internal cluster access only
- Assigns a cluster-internal IP

**NodePort:**
- Exposes service on each node's IP at a static port
- Accessible from outside cluster via `<NodeIP>:<NodePort>`

**LoadBalancer:**
- Provisions external load balancer from cloud provider
- Extends NodePort and ClusterIP

**ExternalName:**
- Maps service to external DNS name (returns CNAME record)
- No selectors, no Endpoints, no proxying

Additional Concepts:

**Endpoints:**
- Resource that sits between Service and Pods
- Maintains list of IP addresses of pods that match the service selector
- Automatically managed by Kubernetes

**Headless Services:**
- Set `clusterIP: None`
- No load balancing or single IP
- Used for direct pod-to-pod discovery via DNS

**Services without Selectors:**
- Manually manage Endpoints
- Use case: connect to external databases or services in another namespace

**Ingress (separate from Services):**
- NOT a Service type
- HTTP/HTTPS routing layer that works with Services
- Provides URL paths, SSL termination, name-based virtual hosting

## Volumes
Basics:
- Each container has its own isolated filesystem from its image
- Filesystem is ephemeral - data is lost when container restarts
- New containers start fresh with only files from the image build
- Containers in the same pod can't see each other's filesystem by default

Why volumes?
- Preserve data beyond container lifecycle
- Share data between containers in a pod
- Provide configuration or secrets to containers

Volume lifecycle:
- Defined as part of pod specification
- Lives as long as the pod exists (unless using persistent volumes)
- All containers in pod can mount the same volume
- Data preserved across container restarts (within same pod)

Types:
Ephemeral Volumes (pod lifecycle):
- `emptyDir`: Empty directory, deleted when pod removed
- `configMap`: Inject configuration data
- `secret`: Inject sensitive data
- `downwardAPI`: Expose pod/container metadata

Persistent Volumes:
- `persistentVolumeClaim` (PVC): Request for storage
- `hostPath`: Node's filesystem (avoid in production)
- Cloud provider volumes: AWS EBS, Azure Disk, GCE PD
- Network storage: NFS, iSCSI, Ceph, GlusterFS

Each container in pod has own isolated filesystem: comes from container image

New containers have exactly the same files added at build time

New container never sees anything written by previous container

Use volumes if you want to preserve data

Defined as part of pod

Sidecar: augments operations of main container of pod

Persistent storage: NAS

## Sidecar Pattern (separate concept):
- Helper container that augments main container
- Common uses: logging, monitoring, proxies, data sync
- Shares pod resources and can share volumes
- Example: Log collector sidecar reading main container's log volume

## Container Entrypoint & CMD (Docker/Container concepts)
ENTRYPOINT:
- Defines the executable invoked when container starts
- Not easily overridden (requires --entrypoint flag)
- Example: ENTRYPOINT ["nginx"]

CMD:
- Specifies default arguments passed to ENTRYPOINT
- Easily overridden by providing arguments to docker run
- If no ENTRYPOINT, CMD is the complete command
- Example: CMD ["-g", "daemon off;"]

In Kubernetes:
- command: overrides ENTRYPOINT
- args: overrides CMD

## Config maps and secrets
ConfigMaps:
- Store non-sensitive configuration data in key-value pairs
- Decouple configuration from container images
- Can be updated without rebuilding images
- Note: Updating ConfigMap does NOT automatically update running pods (unless using mounted volumes with certain configurations)

Secrets:
- Store sensitive data (passwords, tokens, keys)
- Base64 encoded (not encrypted by default!)
- More restricted access controls than ConfigMaps
- Types: Opaque, docker-registry, TLS, service-account-token

Ways to Use ConfigMaps/Secrets:
Environment Variables:
- Set individual env vars from specific keys
- Set all key-value pairs as env vars
- Static - won't update if ConfigMap/Secret changes

Volume Mounts:
- Mount as files in a directory
- Each key becomes a filename
- Can auto-update when ConfigMap/Secret updates (with some delay)

Command Arguments:
- Pass as arguments to container commands

## Deployments

Manages ReplicaSets and provides declarative updates to pods

Update Strategies:

Recreate:
- Delete all existing pods, then create new ones
- Downtime between old and new version
- Use case: When app can't handle multiple versions running

RollingUpdate (default):
- Gradually replace old pods with new ones
- Zero downtime deployment
- Configurable parameters:
   - `maxSurge`: How many pods above desired replica count (default: 25%)
   - `maxUnavailable`: How many pods can be unavailable (default: 25%)

How Deployments Work:

Architecture:
- Deployment manages ReplicaSets
- Each ReplicaSet represents a version of the pod template
- ReplicaSets manage the actual pods
- Old ReplicaSets kept for rollback (controlled by `revisionHistoryLimit`)

Updating a Deployment:
- Trigger update by modifying pod template (image, env, etc.)
- Creates new ReplicaSet
- Scales up new ReplicaSet while scaling down old one
- Config changes outside pod template don't trigger rollout

Block bad rollout: check readiness probe, if it fails rollout will not continue

New pods must pass readiness checks before receiving traffic
Failed readiness = rollout automatically blocked
Deployment stays partially updated until fixed


