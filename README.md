# Source material

https://youtu.be/MTHGoGUFpvE?si=1H6K97HEiUpilqvt

# Developer Context

We got some code plus the dependencies needed for our application to run.  

Our application is divided into multiple functional parts called **microservices**. 

We put the code and its dependencies inside of isolated environments called **containers**, one container per microservice.  

# Containers offer many advantages

Containers are very easy to spin up, and equally easy to replace.  

Which means we can update and scale our application in a matter of seconds.  

And containerized apps run the same regardless of the host system, which is actually the reason why containerization was invented...  

We can also create multiple copies of a container to ensure our app (or a given functionality of our app) is always available.  

# Why should we even use k8s?

Containers run on top of an existing host environment (a node), usually a VM but it can also be a physical machine.  

And we can create as many containers as we want so long as the node's resources permit...  

Once the node has reached its maximum number of containers, we need to use another node to keep scaling up.  

And if we don't use a new node, our containers could die without us knowing...

If we only one node and it dies, we lose all the containers that depend on it, which is another reason for using backup nodes.  

All those challenges are the reasons why Kubernetes (k8s) was invented.  

And k8s answers many other challenges that we haven't mentioned yet...  

# How does k8s work?

Containers work exactly the same in k8s as they do outside of it.  

The major difference resides in that k8s encapsulates containers inside objects called **Pods**.  

k8s manages Pods, not the containers inside of them.  

Pods are the simplest deployable unit in k8s world. And they can encapsulate one or more containers.  

## What's a cluster

Pods and their containers still need a host machine, a node that will provide resources (storage, RAM and CPU).  

The combination of multiple nodes is called a **cluster**.  

## Control Plane & worker nodes

There a 2 kinds of nodes in k8s:
- control plane: contains all the tools required to keep our cluster running.
- worker node: the vast majority of Pods are created inside worker nodes, also called **workers**. These nodes are the ones running our containerized services

7/170
