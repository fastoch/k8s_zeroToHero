# Source material

https://youtu.be/MTHGoGUFpvE?si=1H6K97HEiUpilqvt

# Application Developer Context

We got some code plus the dependencies needed for our application to run.  
Our application is divided into multiple functional parts called **microservices**. 
We put the code and its dependencies inside of isolated environments called **containers**, one container per microservice.  

# Containers offer many advantages

Containers are very easy to spin up, and equally easy to replace.  
Which means we can update and scale our application in a matter of seconds.  
And containerized apps run the same regardless of the host system, which is actually the reason why containerization was invented...  

We can also create multiple copies of a container to ensure our app (or a given functionality of our app) is always available.  

# Why should we even use k8s? Containers are not enough

Containers run on top of an existing host environment (a node), usually a VM but it can also be a physical machine.  
And we can create as many containers as we want so long as the node's resources permit...  

Once the node has reached its maximum number of containers, we need to use another node to keep scaling up.  
And if we don't use a new node, our containers could die without us knowing...

If we only have one node and it dies, we lose all the containers that depend on it, which is another reason for using backup nodes.  

All those challenges are the reasons why Kubernetes (k8s) was invented.  
And k8s answers many other challenges that we haven't mentioned yet...  

# k8s Pods, Nodes and Clusters

Containers work exactly the same in k8s as they do outside of it.  
The major difference resides in that k8s encapsulates containers inside objects called **Pods**.  

k8s manages Pods, not the containers inside of them.  
Pods are the simplest deployable unit in k8s world. And they can encapsulate one or more containers.  

## What's a cluster?

Pods and their containers still need a host machine, a node that will provide resources (storage, RAM and CPU).  
The combination of multiple nodes is called a **cluster**.  

## Control Plane & worker nodes

There a 2 kinds of nodes in k8s:
- **control plane**: contains all the tools required to keep our cluster running.
Sends instructions to the worker nodes.
- **worker node**: the vast majority of Pods are created inside of worker nodes. These nodes are the ones running our containerized services.

# k8s components

## kubectl

That is the CLI tool that is going to be installed inside of our personal workstation.  
kubectl enables us to access our k8s cluster. We do so by connecting to the control plane (controller node).  

## kubeconfig

In order for kubectl to be able to find and access a k8s cluster, we need to use a **kubeconfig** file.  
This is a YAML file that requires the use of `kubectl config` commands to be generated.  
And this file needs to be on the same machine where kubectl is installed.

## API Server

That's one of the control plane's components.  
All our kubectl requests are going to be received by the API server.  
All communication that occurs inside of a k8s cluster is done through the API Server.  

## etcd

Another one of the control plane's components. Was around long before k8s.  

It's a distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system or a cluster of machines.  

etcd stores the entire configuration of a cluster, its state. It's the cluster's memory.  
Losing etcd is like lobotomizing your cluster.  

## Scheduler

Also a control plane's component.  
Responsible for selecting the best node for placing a given Pod.  
Once the Pod has been placed on a node, the Scheduler's job is done and **kubelet** then takes over to deploy and observe the Pod.  

## Controller manager

A daemon that embeds controllers inside the control plane, such as:
- replication controller
- endpoints controller
- namespace controller
- service accounts controller
- etc.

## kubelet

This component runs on every node, including the control plane.  
It makes sure that containers are started, stopped or restarted appropriately.  

Any instruction that comes from the control plane is received by kubelet.  
kubelet is the sole connection of each node to the rest of the cluster.  

Needless to say that kubelet is one crucial component...  

## Container Runtime 

If you're using k8s, you are necessarily using a container runtime, because it is required to build the containers that will be running inside our k8s Pods.  

On every single node that's part of our k8s cluster, there's a container runtime engine.  
And the commands required for creating containers (`docker run`, `podman run`, etc.) are written for us by the **kubelet** component.  

To build containers that will live in a k8s Pod, we still need an image, a template with instructions about how the container should be built
and how the application inside of it should be set up.  

# Creating a Pod

1. We use `kubectl` commands to create a Pod
2. This is received by the API server inside the control plane (or controller node)
3. The API server tells etcd about our Pod creation request
4. if all checks pass, etcd sends a 200 HTTP response to the API server, which means "ok for creating a new pod"
5. the API server transmits this 200 response back to us (at the command line)
6. the API server then asks the Scheduler which node this new Pod will be created on 
7. the Scheduler makes a decision and informs the API server about the Pod's location
8. the API server informs etcd about the new Pod's location and tells the Scheduler to go ahead and place the new Pod
9. the API server reaches out to the kubelet component located on the worker node where our new Pod will be created
10. on the chosen worker node, kubelet runs all the necessary commands so the container runtime builds the container
11. And kubelet is also responsible for making the Pod in which the container will live

**Precision**:  
kubelet first creates the Pod, and only then it asks the container runtime to build the container inside of that Pod.  

Once the Pod has been created, kubelet will be constantly feeding status updates to the API server.  
And the API server will in turn make sure that these updates are being recorded to etcd.  

# YAML Manifests

When working with YAML, there are 2 types of data that you need to be aware of:
- lists (or sequences)
- dictionaries (or mappings)

In YAML, lists look like this:
```yaml
- milk
- eggs
- ice cream
- bread
```

Dictionaries are all about key-value pairs:
```yaml
flavor: yogurt
brand: 
expiration:
```


22/170
