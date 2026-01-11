# Source material

- k8s zero to hero (2025 - 2h50): https://youtu.be/MTHGoGUFpvE?si=1H6K97HEiUpilqvt
- k8s secrets (2025 - 18min): https://www.youtube.com/watch?v=mSJXn6XdEr0&t=14s

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

A k8s cluster is an aggregate of multiple nodes that are networked together in order to form a cohesive distributed system.  

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

# Creating a Pod - workflow

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
brand: Hershey's
expiration: October 30
delicious: true
```

Actual YAML manifests combine these 2 types: 
```yaml
icecream:
  flavor: yogurt
  brand: Hershey's
  expiration: October 30
  toppings:
    - sprinkles
    - fudge
    - whipped cream
  delicious: true
```

>[!important]
>Indentation is very important in YAML files.

# Creating your own YAML manifest

There are tons of templates available online, and you'll easily find the one that matches your need.  

Most manifests contain the following keys:
```yaml
apiVersion: 
kind: 
metadata:
  name: 
  namespace: 
  labels:
  annotations:
spec:
  containers:
  - name: container1
    image:
    ports:
  - name: container2
    image:
    port:
```
Note that the "Pod" value needs to be written with an uppercase P.  

# Most common `kubectl` commands

- `kubectl get pods` returns the list of all pods in the current namespace
- `kubectl get namespaces` returns all namespaces in the current cluster
- `kubectl get pods -n <targeted_namespace>` returns all pods in a specific namespace
- `kubectl create ns <new_namespace>`
- `kubectl apply -f my-manifest.yml` creates a k8s resource based on the specified manifest file
- `kubectl describe <resource_type> <resource_name>` brings up detailed information about a resource, useful for troubleshooting
- `kubectl logs <pod_name>` to show the stdout and stderr generated by containers inside a pod
- `kubectl delete <resource_type> <resource_name>` or `kubectl delete -f my-manifest.yml`

# Creating a Resource quota

A ResourceQuota in k8s defines a hard cap on cumulative resource usage and object counts within a single **namespace**.  

A YAML manifest for a basic resource quota would look something like that:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tiny-rq
spec:
  hard:
    cpu: "1"
    memory: 1Gi
```

We can then apply this resource quota to a specific namespace:
```bash
kubectl apply -f tiny-rq.yml -n <targeted_namespace>
```

We can create a demo namespace and apply our resource quota to it, then check if that's been applied: 
```bash
kubectl create ns demo
kubectl apply -f tiny-rq.yml -n demo
kubectl describe ns demo
```

# Upgrading a k8s cluster is not an easy task

Migrating a k8s cluster to a newer version can introduce compatibility issues and make our previously working manifest files suddenly obsolete.  

It is recommended to:
- review release notes for breaking changes and deprecated APIs before starting
- test the upgrade in a staging environment mirroring production

https://www.linkedin.com/pulse/challenges-upgrading-kubernetes-clusters-suheb-ghare-d5zqf/

# Monitoring Resource Consumption  

1. install the Kubernetes Metrics Server:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
2. Insecure TLS is often needed for local setups like minikube, kind, or k3s due to self-signed kubelet certificates:
```bash
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```
1. Wait 1-2 minutes, then check that the metrics server is up via `kubectl get apiservices | grep metrics` (status should be "True")
2. once this API service is running, we can run `kubectl top nodes` to monitor all nodes in our cluster
3. and to monitor all pods in all namespaces: `kubectl top pods -A`

# Balancing Resource Consumption

We'll do that by modifying our manifests, so that we don't starve our containers, but we don't let them consume more than they need.  

We have 2 parameters to set the right balance:
- requests: guarantees that your Pod's container is allocated a minimum amount of resources
- limits: sets the maximum amount of resources your Pod's container is allowed to consume

Example manifest for a Pod:
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: demo-pod
spec: 
  containers: 
  - name: nginx
    image: nginx:1.14.2
    resources: 
      requests: 
        cpu: 250m
        memory: 65M
      limits: 
        cpu: 500m
        memory: 130M
```

Notice that requests and limits are set on a container by container basis, not at the Pod level.  
And 250m for the cpu means "250 millicore".  

>[!important]
If there's a resource quota in place for the namespace in which the above Pod will live, we'll be able to create more Pods 
like this one, but only up until we reach the resource quota.  

## Recap about managing resource consumption

We set requests and limits at the container level when writing the manifest that will be used for Pod creation.  
Then we also write a manifest to define a resource quota at the namespace level, and all Pods that will live within 
that namespace can only consume as much as allowed by our resource quota.  

In other words, even when you've restricted your containers via requests/limits, at some point you're going to 
hit the maximum resources your namespace is allowed to use.  

# Probes

Probes are basically watch dogs that you can put on top of individual containers.  
There are 2 types of probes: **readiness** probes and **liveness** probes.  

## Liveness probe

The probe will keep poking the container to make sure it's still alive.  
If the container doesn't respond, the probe will count one strike.  
After a predetermined number of consecutive strikes, the probe will kill the non-responding container.  

## Readiness probe

This type of probe does the same but doesn't kill a non-responding container.  
Instead, it will prevent it from being added to the LoadBalancer, so that no traffic is routed to it.  

## Probe manifest example

Probes are extremely **granular**, meaning we have a lot of control over how we want them to act.   

Here's a simple example with a liveness probe: 
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: sise-lp
spec:
  containers:
  - name: sise
    image: mhausenblas/simpleservice:0.6.0
    ports:
    - containerPort: 9876
    livenessProbe: 
      initialDelaySeconds: 2
      periodSeconds: 5
      timeoutSeconds: 1
      failureThreshold: 3
      httpGet:
        path: /health
        port: 9876
```
As we said, the probe is added at the container level.  

- The `httpGet` section is where we determine the method the probe will be using to tap the container.  
  - Since the simple service application we're using here is an API, it can receive HTTP requests.  
  - `path` is the endpoint to which the probe will send GET requests.  
  - `port` is the specific port inside the container to which the requests must be sent.  
- `initialDelaySeconds`: how soon after creation the container will start being probed
- `periodSeconds`: at which frequency the container will be probed, every 5 seconds in our example
- `timeoutSeconds`: how long are we giving the container to respond
- `failureThreshold`: how many consecutive failures before killing the container

Writing a readinessProbe in a YAML manifest would be no different from this livenessProbe example.  
Except that a non-responding container would make the probe turn off traffic to the corresponding Pod until the container responds.  

Read the official documentation for more details about probes:  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/  

# Port Forwarding

```bash
kubectl run demopod --image=nginx
kubectl port-forward demopod 2224:80
```

Let's explain the above commands:
- `kubectl run` is a quick and dirty way to create pods without using a manifest
- `kubectl port-forward` maps a port available on the host system to the container's port on which the app is listening
  - in our example, we map localhost:2224 to the container's port 80, which is the default port for Nginx

This last command is going to hog up your command line, so you need to either split your terminal window or open a new one.  
In your new terminal window (our new pane), run `curl localhost:2224` to get the Nginx welcome page.  

Press ctrl+D to close the new terminal window, and ctrl+C to stop port forwarding in the first one.  

# Opening an interactive terminal inside a Pod's container

To open an interactive terminal in our demopod (with the sh shell):  
`kubectl exec -it demopod -- sh`  

And to check the container's underlying OS:  
`cat /etc/os-release`  

# Writing files inside a Pod's container

While having an interactive terminal running inside our container, we cannot use text editors such as vim or vi.  
Instead we can use tricks like `echo "some random text" > /var/www/index.html` to write files inside the container file system.  

# Copying local files into a Pod's container

`kubectl cp <path_to_local_file> demopod:<destination_path_inside_the_container>`  

For example: `kubectl cp ~/Documents/nginx.conf demopod:etc/nginx/nginx.conf`  

# Data Persistence with ConfigMaps

Pods and containers are stateless and ephemeral, when they die, all data inside of them is lost.  
In k8s, there's a resource called a **configMap** that allows us to store data such as config files, environment variables, etc.   

Making configMaps is very easy, attaching them to Pods/containers is a bit more complicated.  

## configMap & volumes

The configMap object will be mounted as a volume to the Pod itself.  
We then mount the volume to the container inside that Pod, using a specific mount point.  

This specific mount point will be some arbitrary directory inside the container.  

The same configMap can be mounted to as many Pods as we want.  
This is very convenient because this way, config changes only need to be made in one place.  

## ConfigMap manifest example

1. Let's create a heroes.txt file to which we'll add a bunch of DC heroes: `vim heroes.txt`
2. Let's create a configMap (cm) that contains that file: `kubectl create cm heroes-data --from-file=heroes.txt`
3. we can see this new ConfigMap and its contents via `kubectl get cm` and `kubectl describe cm heroes-data`

## Attaching a configMap to a Pod

2 steps:
1. add the configmap as a volume to the pod
2. mount that volume inside the container itself

Let's apply this to our previous Nginx pod manifest:
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: demo-pod
spec: 
  containers: 
  - name: nginx
    image: nginx:1.14.2
    # step 2: mount the volume inside the container
    volumeMounts:
    - name: dc-heroes
      mountPath: /heroes

  # step 1: add the configmap as a volume to this pod
  volumes:
  - name: dc-heroes
    configMap: 
      name: heroes-data
```

Then, we can create this pod via `kubectl apply -f podmanifest.yml`.  
And we can hop inside that pod via `kubectl exec -it demo-pod -- sh`  

Once inside our pod, we can run `ls` to notice that the /heroes directory (our mountPath) has been created.  
And if we run `ls heroes`, we'll see the contents of our configmap, which is the heroes.txt file.  

## About mountPath & subPath

In our previous example, we used `mountPath` to create a non-existing directory that will host our configMap data.  

**But what if we need to write data to an already existing directory that contains other files?**  
If we specify `mountPath: /etc/nginx/` in our manifest, this will overwrite the existing `nginx` directory.  
And if we specify `mountPath: /etc/nginx/heroes.txt`, that would just create a directory called `heroes.txt`, which is not what we want.  

The solution is to use a **subpath**, which can be done as follows:
```yaml
volumeMounts:
- name: dc-heroes
  mountPath: /etc/nginx/heroes.txt
  subPath: heroes.txt
```

Instead of mounting the entire configMap/volume as a directory, `subPath` tells Kubernetes to mount only a single file 
from the configMap/volume = heroes.txt

---

# Secrets explained by Mischa van den Burg

source: https://www.youtube.com/watch?v=mSJXn6XdEr0&t=14s  

## What is a k8s secret?

It's a k8s object which kind is `Secret` that is designed to hold sensitive data.  
This data is stored as key-value pairs, where the values are base64-encoded (encoded, not encrypted).  

We can generate base64 codes from the CLI: `echo mypassword | base64`  
We can decode the generated string via `echo bXlwYXNzd29yZAo= | base64 -d`  

Real security relies on **etcd encryption at rest** (done in our cluster config) and **RBAC** (Role-Based Access Control).  

## Creating k8s Secrets 

- This can be done imperatively via `kubectl` commands (suitable for demos/testing): 
  - `kubectl create secret <secret_type> <secret_name> --from-literal=username=admin --from-literal=password='V3ryS3cr3t'`
  - `kubectl create secret <secret_type> <secret_name> --from-file=./config.json --from-file=./ssh-key`
- Or this can be done declaratively using YAML files (best practice):
```yaml
apiVersion: v1
kind: Secret
metadata: 
  name: my-db-secret
type: Opaque
data: 
  # values must be base64-encoded 
  DB_USER: YWRtaW4= # echo -n admin | base64
  DB_PWD: MWYyZDhjNDI= # echo -n 1f2d8c42 | base64
```

>[!TIP]
By default, the `echo` command appends a newline `\n` after printing its arguments.  
The `-n` option prevents from adding a trailing newline character to the output.  
`-n` ensures only the password string is piped to base64  

- To apply the above secret to our cluster: `kubectl apply -f secret.yaml`
- To see all our secrets: `kubectl get secrets`  
- To see the details of a specific secret: `kubectl get secret my-db-secret -o yaml`  

>[!NOTE]
>With k8s, you're usually not running `kubectl` commands directly, you'll be deploying things from code (GitOps approach).  

## Consuming K8s Secrets as Environment Variables

Here's a Pod manifest in which the container consumes the secret we've just created:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: consume-secret-env-var
spec: 
  containers:
    - name: myapp-container
      image: busybox # using busybox to easily check env vars
      command: ['sh', '-c', 'echo "My username is $APP_DB_USER" && sleep 3600'] # example command
      env: 
        - name: APP_DB_USER # env var name inside container
          valueFrom:
            secretKeyRef: 
              name: my-db-secret # our Secret's name
              key: DB_USER # key within the secret
        # --- OR --- mount all keys from the secret:
        # envFrom: 
        #   - secretRef:
        #     name: my-db-secret
```

>[!NOTE]
The Busybox Docker image simplifies checking environment variables by bundling a lightweight env command (along with hundreds of other Unix utilities) 
into a single tiny executable.  

As you can see, we have 2 options:
- mount a specific key inside our secret as one env var: `valueFrom: secretKeyRef:`
- mount all keys from the secret as environment variables: `envFrom: secretRef:`

Given you've already applied the secret to your k8s cluster, you can now create a file for the above manifest:
```bash
vi pod-secret.yaml
```

And deploy this pod:
```bash
kubectl apply -f pod-secret.yaml
```

And now, if you check the logs of this pod, you'll get "My username is admin":
```bash
kubectl logs consume-secret-env-var
```
Which means you've successfully injected your secret into the container and decoded it.  

To double-check, you can open an interactive terminal inside the container:
```bash
kubectl exec -it consume-secret-env-var -- sh
```
And once inside that container, you can run `env` to see the APP_DB_USER environment variable and its value of 'admin'.  

## Consuming Secrets via Volume Mounts

The second way to consume Secrets inside containers is by using Volume mounts.  



10/18

---

# Secrets explained by Alta3 Research

## Secrets vs configMaps

Secrets and configMaps are very similar.  
Both ConfigMaps and Secrets are Kubernetes API objects used to store configuration data for Pods.  

And both can be exposed to Pods as:
- Mounted volumes (files inside the container filesystem)
- Environment variables

The difference resides in that:
- ConfigMaps are for non-confidential configuration data 
- Secrets are for sensitive data such as passwords, OAuth tokens, and SSH keys.

There are many types of Secrets. K8s provides several built-in types for some commone usage scenarios.  

## Secrets are not encrypted

Secrets are, by default, stored **unencrypted** in the API server's underlying data store (etcd).  
Principles and practices for good Secret management for cluster administrators and application developers:  
https://kubernetes.io/docs/concepts/security/secrets-good-practices/  

## Creating a Secret via a manifest 

Here's an example manifest of how to create a secret:
```yaml

```

We can then create the secret via `kubectl apply -f my-secret.yaml`  

If you run `kubectl describe secrets my-secret`, notice that it won't show the value of our password.  

## How to read in the contents of our secret?




82/170 (48%)