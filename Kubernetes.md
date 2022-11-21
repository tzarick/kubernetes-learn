Parent: [[040 Knowledge Store MOC]]


# Basics
https://kubernetes.io/docs/tutorials/kubernetes-basics/

- Containerizized software
- Kubernetes helps you make sure your containerized applications run where and when you want, and helps them find the resources and tools they need to work
- Basic actions:
	- Create a Kubernetes (K8) **cluster**
	- Deploy an app
	- Explore your app
	- Expose your app publicly
	- Scale up your app
	- Update your app


## Create a Cluster
Objectives:
- Learn what a K8 cluser is
- Learn what Minikube is
- Start a Kubernetes cluster using an online terminal

- Kubernetes coordinates a highly available cluster of computers that are connected to work as a single unit
	- K8 allows you to deploy containerized apps to a cluster without tying them to individual machines
	- To make use of this^ applications need to be packaged ina way that decouples them from individual hosts: they need to be **containerized**
- Kubernetes automates the distribution and scheduling of application containers across a cluster in a more efficient way

A Kubernetes Cluster consists of 2 types of resources:
	- The **Control Plane** - coordinates the cluster
		- schedules apps
		- maintains app desired state
		- scales
		- rolls out updates
	- **Nodes** - the workrs that run applications
		- a VM or physical computer that serves as a worker machine
		- Each node has Kubelet - an agent for managing the node and communicating with K8 control plane
		- Also has tools for containerization (Docker, containerd)
		- A Kubernetes cluster that handles prod traffic should have at least 3 nodes (if 1 node goes down, redundancy is lost)
		- Nodes communicate with the control plane using the Kubernetes API


![[cluster-diagram.png]]

use `kubectl` to interact with kubernetes cluster

## Deploy an App
Objectives:
- learn about app deployments
- deploy your first app on Kubernetes with kubectl

- Create a Kubernetes Deployment configuration
	- this instructs Kubernetes how to create and update instances of your application

- Once the app instances are created, a Kubernetes Deployment Controller continuously monitors those instances
	- can replace downed instances etc
	- this provides a self-healing mechanism to address machine failure / maintenance

- Kubernetes creates app instances AND **keeps them running across Nodes**

**Pod** - the smallest deployable units of computing that you can create and manage in Kubernetes
	- a group of one or more containers with shared storage and network resources, and spec for how to run the containers
	- By default, the Pod is only accessible by its internal IP address within the Kubernetes cluster. To make the `hello-node` Container accessible from outside the Kubernetes virtual network, you have to expose the Pod as a Kubernetes [_Service_](https://kubernetes.io/docs/concepts/services-networking/service/).
	- `kubectl expose ...`

- `kubectl create deployment`

Things that happen when creating a deployment. kubectl:
-   searched for a suitable node where an instance of the application could be run
-   scheduled the application to run on that Node
-   configured the cluster to reschedule the instance on a new Node when needed

## Explore Your App
Objectives:
- Learn about Kubernetes Pods
- Learn about Kubernetes Nodes
- Troubleshoot deployed applications


Pod:
- a K8 abstraction that represents a group of one or more application containers (such as Docker), and some shared resources for those containers. Resources include:
	- shared storage (Volumes)
	- Networking, as a unique cluster of IP addresses
	- Info about how to run each container (container image version, specific ports to use)
- A pod models an app-specific "logical host"
	- A pod might include both the container with a Node.js app as well as a different container that feeds the data to be published by the Node.js webserver
	- The containers in a Pod share an IP Address and port space, are always co-located and co-scheduled, and run in a shared context on the same Node
- Pods are the atomic unit on the K8 platform
- When we create a Deployment on Kubernetes, that Deployment creates Pods with containers inside them (as opposed to creating containers directly). Each pod is tied to the Node where it is scheduled, and remains there until termination or deletion. For duplication, identical Pods are scheduled on other available Nodes in the cluster

 ![[Pods.png]]

- A Node can have multiple Pods
- Kubernetes control plane automatically handles scheduling the pods across the Nodes in the cluster
- Each K8 Node runs at least:
	- Kubelet - process responsible for comms between control plane and Node - manages the Pods and the containers running on the machine
	- Container runtime (i.e. Docker) - responsible for pulling the container image from a registry, unpacking the continer, and running the app

![[Node.png]]


## Expose Your App Publicly
Objectives:
- Learn about a Service in K8
- Understand how labels and LabelSelector objects relate to a Service
- Expose an app outside a K8 cluser using a Service

- Pods have a lifecycle - they will die. When a Node dies, the Pods running on it are lost
	- A ReplicaSet can then dynamically bring the cluster back to the desired state
	- However, each Pod has a unique IP, even Pods on a same Node - we need to be able to reconcile changes among the Pods so the app continues to function seamlessly

**Service** - abstraction which defines a logical set of Pods and a policy by which to access them
- enables a loose coupling between different Pods
- defined in YAML (or JSON)
- the set of Pods targeted by a Service is usually determined by a LabelSelector
- Each Pod has a unique IP - however, these IPs are not exposed outside the cluster without a Service
	- Services allow your app to receive external traffic
	- Can expose in different way depending on the type specified:
		- **ClusterIP** (default) - only exposed on an internal IP in the cluster. Not externally reachable
		- **NodePort** - Exposes the Service on the same port of each selected Node in the cluster using NAT. Externally addressable via `<NodeIP:NodePort>`
		- **LoadBalancer** - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service
		- **ExternalName** - Maps the service to the contents of the externalName field, by returning a CNAME record with its value

- A Service routes traffic across a set of Pods
- They allow Pods to die and replicate without affecting the running application
- Services match a set of Pods using Labels and Selectors

![[Service.png]]


## Scale Your App
Run multiple instances of the application

Objectives:
- Scale an upp using kubectl

- Scaling is accomplished by changing the number of replicas in a Deployment

![[Scaling.png]]

- Running multiple instances of an application will require a way to distribute traffic to all of them
- Services have an integrated load-balancer that will distribute network traffic to all Pods of an exposed Deployment
- When you have multiple instances of an application running, you would be able to do rolling updates without downtime

## Performing a Rolling Update
Objectives:
- Perform a rolling update using `kubectl`

Rolling updates allow for:
- Promotion of an application from one env to another (via container image updates)
- Rollback to prev versions
- CI/CD of applications with zero downtime

Important commands:
`kubectl describe <target>`
`kubectl get <target>`