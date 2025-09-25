# Kubernetes in action

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


## Job 
"allows you to run a pod whose container isn’t restarted when the process running inside finishes successfully."
