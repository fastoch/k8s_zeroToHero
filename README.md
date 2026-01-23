# Source material

- k8s zero to hero (2025 - 2h50): https://youtu.be/MTHGoGUFpvE?si=1H6K97HEiUpilqvt
- k8s secrets (2025 - 18min): https://www.youtube.com/watch?v=mSJXn6XdEr0&t=14s

# 1. Application Developer Context

We got some code plus the dependencies needed for our application to run.  
Our application is divided into multiple functional parts called **microservices**. 
We put the code and its dependencies inside of isolated environments called **containers**, one container per microservice.  

# 2. Containers offer many advantages

Containers are very easy to spin up, and equally easy to replace.  
Which means we can update and scale our application in a matter of seconds.  
And containerized apps run the same regardless of the host system, which is actually the reason why containerization was invented...  

We can also create multiple copies of a container to ensure our app (or a given functionality of our app) is always available.  

# 3. Why should we even use k8s? Containers are not enough

Containers run on top of an existing host environment (a node), usually a VM but it can also be a physical machine.  
And we can create as many containers as we want so long as the node's resources permit...  

Once the node has reached its maximum number of containers, we need to use another node to keep scaling up.  
And if we don't use a new node, our containers could die without us knowing...

If we only have one node and it dies, we lose all the containers that depend on it, which is another reason for using backup nodes.  

All those challenges are the reasons why Kubernetes (k8s) was invented.  
And k8s answers many other challenges that we haven't mentioned yet...  

# 4. k8s Pods, Nodes and Clusters

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

# 5. K8s components

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

# 6. Creating a Pod - workflow

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

# 7. YAML Manifests

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

# 8. Creating your own YAML manifest

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

# 9. Most common `kubectl` commands

- `kubectl get pods` returns the list of all pods in the current namespace
- `kubectl get namespaces` returns all namespaces in the current cluster
- `kubectl get pods -n <targeted_namespace>` returns all pods in a specific namespace
- `kubectl create ns <new_namespace>`
- `kubectl apply -f my-manifest.yml` creates a k8s resource based on the specified manifest file
- `kubectl describe <resource_type> <resource_name>` brings up detailed information about a resource, useful for troubleshooting
- `kubectl logs <pod_name>` to show the stdout and stderr generated by containers inside a pod
- `kubectl delete <resource_type> <resource_name>` or `kubectl delete -f my-manifest.yml`

# 10. Creating a Resource quota

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

# 11. Upgrading a k8s cluster is not an easy task

Migrating a k8s cluster to a newer version can introduce compatibility issues and make our previously working manifest files suddenly obsolete.  

It is recommended to:
- review release notes for breaking changes and deprecated APIs before starting
- test the upgrade in a staging environment mirroring production

https://www.linkedin.com/pulse/challenges-upgrading-kubernetes-clusters-suheb-ghare-d5zqf/

# 12. Monitoring Resource Consumption  

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

# 13. Balancing Resource Consumption

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

# 14. Probes

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

# 15. Port Forwarding

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

# 16. Opening an interactive terminal inside a Pod's container

To open an interactive terminal in our demopod (with the sh shell):  
`kubectl exec -it demopod -- sh`  

And to check the container's underlying OS:  
`cat /etc/os-release`  

# 17. Writing files inside a Pod's container

While having an interactive terminal running inside our container, we cannot use text editors such as vim or vi.  
Instead we can use tricks like `echo "some random text" > /var/www/index.html` to write files inside the container file system.  

# 18. Copying local files into a Pod's container

`kubectl cp <path_to_local_file> demopod:<destination_path_inside_the_container>`  

For example: `kubectl cp ~/Documents/nginx.conf demopod:etc/nginx/nginx.conf`  

# 19. Data Persistence with ConfigMaps

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

Then, we can create this pod via `kubectl apply -f pod_configMap.yml`.  
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

# Secrets explained by Mischa van den Burg (nested course)

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
vi pod-secret-env-var.yaml
```

And deploy this pod:
```bash
kubectl apply -f pod-secret-env-var.yaml
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
This will read every key inside the data section of our Secret file and create one file per key.  
Each file will be named after a key and will contain the value for that key.  

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: consume-secret-volume-mount
spec: 
  containers:
    - name: myapp-container
      image: busybox # using busybox to easily check env vars
      command: 
        - "sh"
        - "-c" 
        - "echo 'Checking secrets:' && ls /etc/secrets/ && sleep 3600"
      volumeMounts:
        - name: secret-storage # refers to volume defined below
          mountPath: "/etc/secrets" # folder in which the volume will be mounted inside the container
          readOnly: true # Good practice
  volumes:
    - name: secret-storage 
      secret: 
        secretName: my-db-secret # name of the secret object
```  

We can create a file and paste the above YAML code inside it: `vi pod-secret-volume-mount`  
Then we can create that pod: `kubectl apply -f pod-secret-volume-mount`  

And if we check the logs of that pod: `kubectl logs consume-secret-volume-mount`  
We'll get the following output, which shows 2 files that each contain one of the values of our Secret file:  
```bash
Checking secrets:
DB_PWD
DB_USER
```

We can now enter the container and show the contents of the DB_PWD and DB_USER files:
```bash
kubectl exec -it consume-secret-volume-mount -- sh
cd /etc/secrets/
cat DB_PWD
cat DB_USER
```

## Secrets Security - Environment Variables vs Volume Mounts

**Volume mounts** provide the best way to consume Kubernetes secrets over environment variables.  
They offer superior security by avoiding exposure in process lists, logs, or subprocesses.  
Environment variables carry higher risks of leakage.  

**Prefer volume mounts** with `readOnly: true` and `defaultMode: 0400` for production workloads.  

For full security, combine with:
- **etcd encryption at rest**, 
- **RBAC least-privilege**, 
- and **external stores** like **Vault**   

## RBAC: Role-Based Access Control

**RBAC least-privilege** in Kubernetes means granting users, service accounts, or workloads only the minimal permissions needed for their specific tasks.  

Apply **least-privilege** by starting with no permissions and adding only what's required, such as read-only access to specific resources like pods or secrets in one namespace.  

Use granular rules with exact verbs (get, list, watch) and resources, avoiding broad cluster-admin roles.  

Regularly audit with `kubectl auth can-i` to verify and revoke excessive rights.

## Etcd encryption at rest

The **etcd** database is Kubernetes' key-value store for cluster state.  
**etcd encryption at rest** protects data stored in the etcd database by encrypting sensitive resources like **Secrets**.  

The `kube-apiserver` handles encryption transparently: 
- it encrypts data on write to etcd 
- and decrypts on read via the Kubernetes API.   

Multiple providers support this, ordered by priority in an **EncryptionConfiguration YAML file** passed to the `apiserver` 
via the `--encryption-provider-config` flag.  

Common providers include:
- **aescbc** (local AES-CBC keys), 
- **kms** (external Key Management Service like AWS KMS or GCP Cloud KMS), 
- and **secretbox**

## Secret Limitations

- Manual rotation needed: update the secret and restart deployments that depend on it
- The Base64-encoded value is visible in YAML and API responses (hence the need for strict RBAC) 

## Handling Secrets in a Production Environment - The GitOps Pattern using SOPS and Flux 

A solution to these limitations resides in using an external secrets operator.  

The GitOps pattern enables secure management of Kubernetes secrets by storing **encrypted** files in **Git** repositories.  
**Flux** automatically **decrypts** secrets and applies them to clusters.  

**Flux** acts as the GitOps operator to sync declarative configurations from Git.  
While **SOPS** (Secrets Operations) encrypts secrets using keys like GPG, age, or cloud KMS (Key Management Service) before committing them.  

**Flux** includes controllers like: 
- `source-controller` for Git syncing 
- and `kustomize-controller` for decryption and application. 

**SOPS** integrates natively via the `--decryption-provider=sops` flag in **Kustomizations**.  
Supported backends include OpenPGP, age, AWS KMS, GCP KMS, Azure Key Vault, and HashiCorp Vault.


--- END OF NESTED COURSE (K8s Secrets explained by Mischa van den Burg) ---

---

# 20. Secrets explained by Alta3 Research

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

Best practices are described in the "Secrets Security" chapter at line 623. 

## Creating a Secret via a manifest 

Here's an example YAML file (manifest) that we can use to create a secret:
```yaml
apiVersion: v1
kind: Secret
metadata: 
  name: mysql-secret
type: kubernetes.io/basic-auth
stringData: 
  password: alta3
```

We can then create the secret via `kubectl apply -f mysql-secret.yaml`  

If you run `kubectl describe secret my-secret`, notice that it won't show the value of our password.  

## How to read in the contents of our secret?

Here's a Pod manifest where the container will access our secret as an environment variable:
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: mysql-locked
spec: 
  containers:
  - image: mysql:8-debian
    name: mysql
    env: 
      - name: MYSQL_ROOT_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysql-secret
            key: password
```

Create this manifest via `vim pod_alta3_secret.yaml` and copy-paste the above contents into it.  
Then, create the pod via `kubectl apply -f pod_alta3_secret.yaml`  
Once created, we can enter the container: `kubectl exec -it mysql-locked -- bash`  
and if we run `echo $MYSQL_ROOT_PASSWORD`, it will show `alta3`  

# 21. Troubleshooting

## `kubectl logs`

Every container inside a pod generates its own logs.  
And the logs that are generated are always what the stdout (standard output) and the stderr (standard error) 
of a given container may be.  

- to show the logs of a mono-container pod: `kubectl logs <pod_name>`
- to show the logs of a specific container in a multi-container pod: `kubectl logs <pod_name> -c <container_name>`
- to show the logs of all containers inside a pod: `kubectl logs <pod_name> --all-containers`
- to show a live stream of the logs: `kubectl logs <pod_name> -c <container_name> -f`
- to show the logs generated during the last 10 seconds: `kubectl logs <pod_name> -c <container_name> --since 10s`

When a pod gets deleted, all of its logs are removed as well.  

# 22. K8s Labels

Labels are tags for objects, used for grouping and operating on multiple objects simultaneously.  
Labels are very important in Kubernetes.  

Labels are key-value pairs, as shown in this manifest:
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: testpod
  labels:
    runtime: webby
    ver: 2
    ssd: true
    env: dev
spec: 
  containers:
```

## Adding, updating or removing labels

To see if our pods have labels:  
`kubectl get pods --show-labels`  

First method to add|update|remove labels is to edit the pod manifest itself.  
Then we can run `kubectl apply -f podmanifest.yaml` to apply changes, and it works on running pods.  

Second method is by using specific `kubectl` commands:
- to add a label: `kubectl label pod demo-pod awesome=sauce` 
- to update a label: `kubectl label pod demo-pod awesome=training --overwrite`
- to remove a label: `kubectl label pod demo-pod awesome-`

There's no limit to the number of labels we can put on a given object.  

When we'll talk about deployments and services, you'll understand that many types of K8s objects 
depend on labels, and that's why it's important to be proficient at manipulating labels.  

One last command that is very useful with labels:  
`kubectl get pods -L alta3`  

The `-L` flag allows us to add a column for the specified label key.  
In our example, if the specified label is applied to one of our pods, it will show the corresponding value in the dedicated column.  

- To retrieve pods with a specific label: `kubectl get pods --selector=<label>`  
- To retrieve pods with a specific value for a given label: `kubectl get pods --selector=<label>=<value>`  

# 23. Deployments

**Definition**:  
A deployment is a declarative higher-level object that describes the desired state for a set of pods (via a ReplicaSet).  

Inside production environments, Pods are rarely launched on a cluster by themselves.  
Instead, they are made as part of deployments.  

Deployments allow us to create multiple clones of the same pod, called **replicas**.  

## Creating Deployments

This is usually done via manifests.  
But we can create a deployment manually: `kubectl create deployment demo-deploy --image=nginx`  

We can show our deployments: `kubectl get deployments`

## Pods vs Deployments

**Self-healing**:  
When you delete a pod outside deployment context, it doesn't get replaced.  
When you delete a pod that's part of a deployment, it gets replaced.  

**Scaling**:  
Deployments can be scaled, not pods:  
`kubectl scale deployment demo-deploy --replicas 3`  

## Deployment manifest

Deployments were invented after ReplicaSets, they do everything ReplicaSets can do plus:
- version control (rolling updates & rollbacks)
- zero down time
- accepts any changes made to the source YAML file

The hierarchy is Deployment > ReplicaSet > Pods > containers.  

In a deployment manifest, we don't need to name the pods, they'll be named after the replicaset that makes them.  

Here's an example manifest for a deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels: 
    app: nginx
spec: 
  replicas: 3
  selector:
    matchLabels: 
      app: nginx
  template:
    metadata: 
      labels: 
        app: nginx
    spec: 
      containers:
      - name: nginx
        image: nginx:1.14.2
      - name: redis
        image: redis
```

The `selector: matchLabels:` section says "any pod that has that label on it belongs to that deployment".  

## Deployment Rollout Strategy: Rolling Update

Let's say you have one ReplicaSet that counts 3 pods.  
The rolling update would go that way:
- create a duplicate ReplicaSet = ReplicaSet2
- start a new pod inside ReplicaSet2
- delete one pod inside ReplicaSet1
- start a second pod inside ReplicaSet2
- delete one pod inside ReplicaSet1
- start a third pod inside ReplicaSet2
- delete the last pod inside ReplicaSet1
- make ReplicaSet2 the new production ReplicaSet

ReplicaSet1 does not get deleted, it is kept in case we need to roll back.  

## Rollout history

The following command shows all revisions that the specified deployment has gone through:  
`kubectl rollout history deployment demo-deploy`  

To roll back to the previous revision:  
`kubectl rollout undo deployment demo-deploy`  

# 24. Storage

K8s is storage-agnostic.  
But you need to teach your cluster how to handle the storage that you want to use.  
For that, we need to create a **StorageClass**.  

Kubernetes storage for Deployments is built around 3 main objects: 
- **StorageClass** (how to provision), 
- **PersistentVolume** (the actual storage resource), 
- and **PersistentVolumeClaim** (what the app asks for). 

Pods in a Deployment never talk directly to the PV; they mount the PVC, which is bound to a PV that may be dynamically 
created via a StorageClass.

## StorageClass, PersistentVolume and PersistentVolumeClaim

A StorageClass defines different types of storage that cluster administrators can offer to applications.  
It enables dynamic provisioning of **PersistentVolumes** (PVs) based on specified parameters like performance levels, 
backup policies, or reclaim behavior.  

StorageClasses abstract underlying storage systems, allowing pods to request storage via **PersistentVolumeClaims** (PVCs).  
Think of a PVC as the key to access a specific storage (a PersistentVolume).  

- You add a volume to the pod. That volume represents the PVC.
- You mount that volume to a directory inside the container (mountPath)
- And anything that gets saved inside that directory will be 

The PVC (volume) is connected to a specific PersistentVolume that lives on the chosen storage space (AWS for example).  
The storage space of your choice is recognized by K8s thanks to the StorageClass you've defined.  

You can tie the same storage to multiple containers, which makes sense for pods that are part of the same deployment.  

## High-Level Workflow

- Cluster admin defines one or more StorageClasses to describe storage “profiles”
- Storage can be:
  - Statically provisioned: admin creates PVs ahead of time.​
  - Dynamically provisioned: a PVC referencing a StorageClass triggers automatic PV creation.

The data outlives Pods: 
- deleting a Pod or a ReplicaSet does not delete the PV data
- what happens on a PVC deletion depends on the reclaim policy
  
## Example StorageClass (generic CSI)

CSI = Container Storage Interface

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"  # make it cluster default
provisioner: csi.example.com  # your CSI driver
parameters:
  type: ssd
  replication: "2"
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```
- Any PVC without `storageClassName` and with defaulting enabled will use this class.
- `WaitForFirstConsumer` ensures provisioning happens on a node where the consuming Pod is scheduled

## PersistentVolume (PV)

- A PV is a cluster‑level object representing an actual piece of storage.  
- A PV can be created manually (static) or by the provisioner when a PVC asks for storage (dynamic).​
- Its attributes are: capacity, access modes, storageClass, and reclaim policy.​
- It has its own lifecycle phases: Available, Bound, Released, Failed

### Static PV example

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-sc
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /export/data
    server: 10.0.0.50
```
`Retain` means that after the PVC is deleted, the underlying storage and data stay; admin must manually clean up or reuse.  

## PersistentVolumeClaim (PVC) and binding

A PVC is what apps use to request storage: “give me 5Gi, ReadWriteOnce, using this StorageClass.”  

### ReadWriteOnce

ReadWriteOnce (RWO) is a Kubernetes PersistentVolume access mode that allows a volume to be mounted as read-write by a single node in the cluster.  
Multiple pods on that same node can access the volume simultaneously, but pods on different nodes cannot mount it for writing.

### Binding logic

For dynamic provisioning:
- PVC specifies `storageClassName: fast-ssd`; 
- controller asks the StorageClass provisioner to create a matching PV and binds it.​

For static provisioning, controller looks for an existing PV whose:
-`storageClassName` matches,
- `accessModes` satisfy the claim,
- `capacity` ≥ requested.

### Dynamic PVC example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: my-app
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 20Gi
```

### Static binding to a specific PV

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data-static
  namespace: my-app
spec:
  storageClassName: ""     # avoid default StorageClass
  volumeName: nfs-pv       # exact PV name
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```
Setting `storageClassName: ""` disables default dynamic provisioning and allows binding to an explicitly named PV.  

## Using PVCs in a Deployment

Pods never reference PVs directly. They:
  1. Define a volume of type `persistentVolumeClaim` referencing the PVC.
  2. Mount that volume into containers via `volumeMounts`

### End‑to‑end example: StorageClass + PVC + Deployment

You can write multiple manifests in one YAML file using `---` to separate them:
```yaml
# 1) StorageClass (dynamic provisioning)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: csi.example.com
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
---
# 2) PVC requesting storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: my-app
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi
---
# 3) Deployment mounting the PVC
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web
          image: nginx:1.27
          volumeMounts:
            - name: app-storage
              mountPath: /usr/share/nginx/html
      volumes:
        - name: app-storage
          persistentVolumeClaim:
            claimName: app-data
```

- All Pods from this Deployment share the same PVC
- Deleting the Deployment leaves the PVC and PV intact
- Deleting the PVC will either delete or retain the PV based on reclaimPolicy

# 25. K8s Networking

## K8s Service

Every time a pod gets replaced, its IP address changes.  
In order to make our deployment available as a stable network endpoint, we need to make a service out of it.  

Let's delete our previous deployment and recreate it as follows:
```bash
kubectl delete deployment demo-deploy
kubectl create deployment demo-deploy --image=nginx --port=80 --replicas=3
```
The `--port=80` flag specifies the port that the container in each pod exposes for incoming traffic.  
It is required to then expose our deployment and turn it into a service.  

Now, we can expose our deployment and make it available as a service:
```bash
kubectl expose deployment demo-deploy
kubectl get services
```

Our newly created service has its own stable IP address.  
And we can run `kubectl describe svc demo-deploy` to see that: 
- our service has a static IP address 
- our 3 pods have dynamic IP addresses that change every time we scale our deployment up or down  

## Network Policies

Thanks to K8s services, our deployments and their pods can be reached by the outside world.  
But pods within a given deployment can also talk to each other, and we need to set some rules to restrict that.  

A NetworkPolicy, like any other K8s object, is defined through a manifest, such as this one:  
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: test-network-policy
  namespace: default
spec:
  podSelector: # what pods are we controlling?
    matchLabels:
      role: db
  policyTypes:
  - Ingress # incoming traffic
  - Egress  # outgoing traffic
  ingress:
  - from:
    - ipBlock: # must be from this IP range to be allowed
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector: # OR must be from this namespace with this particular label
        matchLabels: 
          project: myproject
    - podSelector: # OR must be from pods with this label to be allowed
        matchLabels: 
          role: frontend
    ports: # you can only receive TCP traffic on port 6379 
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock: # you can only send traffic to this IP range
        cidr: 10.0.0.0/24
    ports: # you can only send TCP traffic using this port 
    - protocol: TCP
      port: 5978
```
This network policy will only affect the pods that live inside the `default` namespace and have the label `role: db`.  
`podSelector: {}` would mean "apply this network policy to all pods".  

---

Kubernetes NetworkPolicies follow a default-allow model by default.  
Without any NetworkPolicy applied to a pod, all ingress and egress traffic is permitted across the cluster.  

Policies become restrictive only when at least one NetworkPolicy selects a pod.  
At that point, traffic to and from that pod is allowed if any matching policy explicitly permits it; unspecified traffic is then denied.  

To achieve a true **default-deny** (where anything not specified is prohibited):
- deploy an explicit "default deny" policy first, such as one with an empty `podSelector: {}` and no ingress/egress rules
- then add allow rules as needed

## Precision about K8s Services

We previously made a service by using the `kubectl expose` command.  
Now let's see how a service manifest looks like:
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: demo-svc
spec: 
  selector:
    demo: pods
  ports:
  - protocol: TCP
    port: 3423 # service's port (exposed to the outside world)
    targetPort: 80 # container's port (hidden from the outside world)
  - protocol: TCP
    port: 4444
    targetPort: 8888
```
There's a mapping between the service's port and the targeted container's port.  
Since it's possible for pods to have multiple containers, we can map multiple ports on the same service 
to route traffic to multiple containers/applications.  

## ClusterIP (internal traffic)

Let's say we have 2 pods on the same node, and pod1 wants to talk to pod2.  
Pod2 needs to be exposed by a service in order for pod1 to reach it.  

If pod2 is exposed, then pod1 can ask the kube-proxy to connect it to pod2.  
The kube-proxy will consult its IP table and look for the IP address of the service that exposes pod2.  
Once it finds that IP address, requests from pod 1 can be routed to pod2.  

Note that there is a kube-proxy on every node inside your cluster.  
And all kube-proxy share the same IP table.  

---

If pod1 wants to talk to a deployment that has 3 replicas (3 identical pods), 
then the kube-proxy will set up the rules for how this traffic gets routed.  
Since there are 3 pods in the targeted deployment, the kube-proxy will load balance requests from pod1 across those 3 pods.  
By default, load balancing will be of type "round-robin".  

Note that kube-proxy also makes sure a pod is healthy before routing any traffic to it.  

--- 

All of that **pod-to-pod communication**, be it within the same node or between different nodes, is possible 
thanks to a special service called a "**ClusterIP**".  

There are 3 types of K8s services:
- ClusterIP
- NodePort
- LoadBalancer

## NodePort (external traffic)

NodePort is the most primitive type of Service.  

It's a Kubernetes Service type that exposes an application running on a cluster to external traffic by opening 
a specific port on every node's IP address.  

Kubernetes assigns a static port (typically in the 30000-32767 range) on all nodes, forwarding traffic from `<node-ip>:<node-port>` 
to the Service's Pods via kube-proxy rules like iptables or IPVS.  

You access the Service using any node's IP and the NodePort, enabling load balancing across Pods even if they're on different nodes.  

NodePort suits dev/testing or custom load balancers but isn't ideal for production due to high port usage and firewall needs.  
For production, prefer **LoadBalancer** or **Ingress** over NodePort.

## LoadBalancer

K8s does not come with LoadBalancers, it's not a native feature.  

Kubernetes LoadBalancer services expose applications externally via **cloud load balancers**.  
They distribute incoming traffic across pods for high availability and performance.  
This service type automatically provisions an external IP in supported cloud environments.  

LoadBalancer services create a stable external IP address that routes traffic to matching pods based on selectors.  


164/170 (96%)