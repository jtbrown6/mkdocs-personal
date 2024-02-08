# Kubernetes Therapy

## Purpose
A good way to find out how to deal with the intracacies of configuring and running Kubernetes On-Prem.

## CKAD Commands

```bash
# General Commands
kubectl <action> --help
kubectl run <podName> --image=nginx --restart=Never --dry-run -o yaml > output.yaml
kubectl exec -it nginx /bin/bash 
kubectl exec -it nginx ls /path/found/when/running/describe/pod
kubectl get pods --all-namespaces
kubectl get <item> <itemName> --export -o yaml > exported.yaml

# Deployment Commands
kubectl create deployment nginx --image=nginx
kubectl run nginx --image=nginx  --replicas=3
kubectl get deploy busybox --export -o yaml > exported.yaml

kubectl create job nginx --image=nginx
kubectl create cronjob nginx --image=nginx --schedule="* * * * *"

# Force Update
kubectl set image deploy/nginx nginx=nginx:1.9.1

# Rollback Update
kubectl rollout undo deploy/nginx
kubectl rollout status deploy/nginx

# Observability (Needs Heapster)
kubectl top pod -n my-namespace
kubectl top node -n my-namespace
kubectl logs -f podName containerName    <- stream logs live, container can be left out

# Service
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml

# Taint 
kubectl taint nodes NodeA app=blue:NoSchedule

# Secrets
kubectl create secret generic my-secret --from-literal=foo=bar -o yaml --dry-run > my-secret.yaml

kubectl get secret SecretName -o yaml    <- view secret
```

## Table of Contents
1. [Flow of Actions](#Flow)
2. [Deep Dive: Kube Proxy](#kube-proxy)
3. [Deep Dive: Security 4C's](#4Cs)
4. [Deep Dive: Network Providers](#network-plugins)

**Table of Contents**
- Control Plane Components
- Work Node Components
- Flow of Actions
- Extending w/ Add-Ons
- Additional Components
- Deep Dive - Operators
- Deep Dive - Kube Proxy
- Deep Dive - Network Plugins
- Deep Dive - Security (4C's)
- Deep Dive - StatefulSets (CassandraDB)

## Control Plane Components (C.A.S.E.)

Contains 4 Total Components
1. API Server
2. etcd
3. Scheduler
4. Controller Manager

### Control Plane Details
- **API Server:** communicates with etcd to read and write the desired state. Desired state is provided from user performing kubectl tasks like sending a pod definition in the form of yaml. Responsible for processing requests for creating and updating resources

- **etcd:** key-value store used as primary database; stores the desired state of cluster 

- **Scheduler:** looks for pods in the pending state that have not been assigned to node yet selects them based on factors like resources, afinity rules and which nodes have capacity. once selected, must talk to the `kubelet` which will then take on the responisibility of the pod

- **Controller Manager:** focuses on the controllers that handle specific types of resources and their desired state for `reconciliation` and `corrective` purposes. so the replication controller kicks off and ensure the desired pod replicas are maintained

## Worker Node Components (P.K.R.)

Contains a total of 3 Components
1. Kubelet
2. Kube Proxy
3. Container Runtime

### Worker Node Details
- **Kubelet:** agent that resides on the node and recieves pod assignment from the scheduler which updates that pod information on itself. From there it communicates with the `container runtime` to create and manage the containers for the pods. it also monitors the nodes health and status of the containers and restarts/stops/deletes if need be while also reporting node status and resouce usage back to control plane

- **Kube Proxy:** maintains network rules on nodes to enable and manage network connectivity to and from pods. so on the nodes, there are network rules typically IPTables (linux based firewall rules for NAT) to define network traffic routing to pods based on service selectors and IP addresses; this also handles when new services are created and handling the forwarding of traffic to the backend pods correctly and even uses the services DNS name alongside the ClusterIP. So the NodePort, ClusterIP, LoadBalancer etc supports the traffic of these. 

- **Container Runtime:** software responsible for doing the `docker run/build` commands via Containerd



## Flow of Actions <a name="Flow"></a> 
Goal is always understanding desired state
1. User runs `kubectl create pod` command to `API Server`
2. `API Server` stores this desired state into the `etcd` store
3. The `Scheduling Phases` kicks in and watches for `pending` state pods and evaluates for various factors and selects appropriate node and assigns
4. The `Kubelet Phase` activates and accepts assignment and interacts with `Container Runtime` to create the containers for that pod
5. The `Kubelet` updates status of the containers to `etcd` in Control Plane
6. The `KubeProxy` begins to handle the networking 


## Extending Cluster Functionality through Common Add-Ons 

1. **CoreDNS:** handles DNS resolution for pods and services using the following structure `serviceName.namespace.svc.cluster.local` 

2. **Dashboard:** WebUI for monitoring cluster

3. **Ingress Controllers:** enables external access to services along with load-balancing

4. **Network Policy:** not to be confused with network plugin, but this handles traffic flow between pods based on rules

5. **Service Mesh:** categorized as `infrastructure` add-ons which enhance networking by adding features like traffic-management, observability and security for microservices

6. **Flux/Flagger:** categories as `tools` not add-ons which handle automation of app deployments, manage releases and progressive delivery like canary deployments


## Additional Components

1. **ConfigMaps:** allows to `decouple` configuration data like `API Endpoints` and `Connection Strings` from the pod definitions so you can update pod configurations without actually changing the pod specs

2. **Labels and Selectors:** Labels are `key-value` pairs and Selectors `filter` based on these labels; So you can have a pod with the `prod` label and apply policies to the selector for that environment

3. **Annotations:** adds additional metadata so you can track `buildIDs` or `ReleaseNotes` 

4. **StatefulSets:** helps with consistency for deploying `databases` or `queues` when you need guarantees for consistent network hostnames and stable storage after pod restarts

5. **DaemonSets:** ensure pod is running on every node which is good for `monitoring agents` or log collectors; Kured is a DaemonSet

6. **Custom Resource Definitions (CRDs):** define custom resources and controllers which is good for a Machine Learning Model so Kubernetes can understand and manage it like it was a built in resource


# Kubernetes Deep Dives

## Kubernetes Operators

**What:** special type of controller that leverage the use of `Custom Resource Definitions` to handle complex application configurations

**Benefits:** users can take high level configurations and settings within a `CRD` and the `Operator` will watch for that CRD and implement into `low-level` actions within Cluster. Allows you to focus on defining desired state while the operator will handle the operational task to make it happen!

**Helm-Chart Comparison:** although they have somewhat of the same functionality, Operators focus on complex application deployments due to their integration directly with Kubernetes since they are basically controllers which will handle current/desired state


## Kube-Proxy <a name="kube-proxy"></a> 

**Overview:** manages network connectivity for pods && ensures services are accessible in/outside cluster and ensures they go to healthy pods via health-checks

### Key Aspects of Kube Proxy
1. **Network Rules Management:** since this sits on `NODES` the network rules are applied via IPTables and define pod traffic routing using their `IP Address` or `Service Selector`

2. **Service Exposure:** when a service is created, backend pods need to have that traffic forwards to them. Pods also need to be accessible via DNS name or ClusterIP address

3. **Service Types:** handles the big 4, ClusterIP, NodePort, LoadBalancer, and ExternalDNS 

## Network Plugins <a name="network-plugins"></a> 

1. **Overlay Networks:** virtual networks built on-top of physical infrastructure; biggest value is they allow `pods on different nodes` to talk to each other as if they were on `the same local network` even if they are in different physical locations

2. **Pod to Node Commuication:** typically this is done using `encapsulation` techniques. If a pod on NodeA wants to talk to pod on NodeB, data packets are encapsulated with additional information like `source and destination IP addresses` and are sent over network and `decapsulated` thereafter

3. **Scalability:** as you add more clusters, nodes, or pods to cluster, the network plugins dynamically manage IP address allocation and routing keeping everything isolated 

Network Provider
- Flannel
- Calico

Flannel
- uses an overlay network which makes things simple
- less advanced features
- not ideal for organizations that have strict security requirements BUT you can add in other projects to improve security
- incurs extra encapsulation which slows down communications due to increased size of packets because it has information in the packet that is necessary for reaching the destination. furthermore, increased CPU resources due to encapsulation/decapsulation so latency is increased which is not good for high network traffic environments

Calico
- does not use a overlay network, which improves performance and scalability
- uses BGP to route packets between host so they dont need an extra layer wrapped with encapsulation to move between hosts
- directs packets natively without an extra step of wrapping traffic in an additional layer
- 


# Understanding Overlay Networks

**Purpose:** allows pods and nodes to communicate regardless if they are hosted on different machines and subnets in a cluster

**Components:**
- Encapsulation: traffic is wrapped with the sourceNode and decapsulated at the `destinationNode`
- Routing/Forwarding: decisions are done with the overlay network NOT with the physical network. Overlay will have its own routeTables and protocols to direct traffic



# Understanding Networking Traffic

## Flannel - Traffic Flow Example

**Steps:**
1. ContainerA tries to talk to ContainerB on a different Host
2. ContainerA traffic goes to HostA's bridge
3. HostA Bridge tries to get the MAC address of ContainerB via ARP
4. Since ContainerB is on HostB, flannel daemon wraps into Layer3 over UDP over physical network


## Calico - Traffic Flow Example

**Steps:**
1. Already works at Layer3
2. There is a routing rule for a default gateway of 169.254.1.1
3. ContainerA talks to default gateway FIRST
4. Because of the BGP client running on the Node, it is synced and knows how to route to the destination

### Differences
- Calico makes use of routing principals like how the internet works 
- Flannel makes use of L2 broadcast domains

## Security - 4C's <a name="4Cs"></a> 

1. Code: Secure apps and libraries
2. Container: Vulnerability Scanning, Control and Protect Runtime
3. Cluster: restrict access to Control Plane and Nodes
4. Cloud: Secure Access to API's and Infrastructure (baremetal)


## StatefulSets and CassandraDB

When creating a StatefulSet, there are multiple components that are deployed or needed as well.
- Pods get stable hostnames and ordinal index ie pod01, pod02
- A headless service (no ClusterIP) is deployed to discover individual pods via DNS
- StorageClass and PVC's are deployed which bind persistent storage to pods even if they are replaced

**CassandraDB:** because of its distributed nature, it uses sharding so data is distributed amongst each other. **Benefit:** although using a DB hosted elsewhere is useful, when you need local data or to keep it closer to the apps, integration with K8 ecosystem like rolling updates or scaling and of course the scaling and availability of K8s, this is uesful.


# Storage in K8s Important Concepts

**Options**
1. Leverage PV and PVC's via HostPath
2. StatefulSets
3. StorageClass w/ Dynamic Provisioning


Important Notes
- Storage Classes allow for pods to claim a PV even if there is no match
- PVC's consume PV resources like Pods do Nodes


Storage Classes
- helps to define different types of storage that can be leveraged (SSD,HDD)
- allows for dynamic provisioning if PVC's are leveraged
- defines provisioners for dynamic provisioning Azure Disk, Nutanix or distribed storage like Ceph

Considerations for Storage Classes
- on-demand volumes to be created without admins via dynamic provisoning
- if you need different types of storage for faster or slower apps
- small environments this is NOT needed

# StatefulSet Blog

**Interesting Links:**
- Stateful Best Practices [Blog](https://loft.sh/blog/kubernetes-statefulset-examples-and-best-practices/)
- Stateful Redis Guestbook App [Link](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/)
- Stateful WordPress and mySQL Example [Link](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)


## Research Topics
- Primary-replica architecture + fixed pod-name being expected for stateful Applications


## Understanding Stateful Applications

**What:**
- Stateful Applications: apps that need to store data and keep track of it
- Examples of StatefuApps: databases like mySQL, Oracle are statefulApps
- Stateless Apps: these are apps that during each new request, they get `new` data and process it like Nginx and NodeJS

**Modern Architecture:**
Stateless Application `NodeJS` **--connects to -->** StatefulApplication `mySQL`

### Understanding StatefulSets

**StatefulSet:** controller that is used to run the stateful app as a `pod`
- assigns a `sticky identity or ordinal number` like `pod[0]` to each replicaPod vs `deployments` which get randomIDs
- new pods are created by cloning the previous pod's data 
- deleting pods will be done in `reverse order` so scaling from 4 to 3 will delete the most recent created pod vs randomly

### StatefulSet - Read/Write Process
**Reading Data:** request is forwarded to any of the 3 pods
**Writing Data:** request will be forwarded to `pod[0]` which is the primary and then synced to other pods



## Storage Class Example
```yaml
#Example Storage Class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Persistent Volumes
- Storage Classes tend to create PVs
- an additional resource as a volume plugin provisioned by the Cluster Admin

Persistent Volume Claims
- requested by the user via the deploy.yaml
- PVC's consume PV resources like a Pod consumes Node resources
- Access modes like RWO or RWM determine capabilities 
- K8 Control Plane will look to match the PVC to the PV. BUT if notExists + Dynamic Provisioning is enabled via StorageClass it will be created regardless





# Options

To ensure data persistence in Kubernetes, you would typically use a Persistent Volume (PV) and a Persistent Volume Claim (PVC). A Persistent Volume is a piece of storage that has been provisioned by an administrator, while a Persistent Volume Claim is a request for storage by a user. Additionally, you might consider using StatefulSets if your application requires stable, unique network identifiers, stable, persistent storage, and orderly, graceful deployment and scaling.

Here are the different options you could use to ensure data persistence for your Python Docker container within a Kubernetes cluster:

1. **Persistent Volumes and Persistent Volume Claims**:
   - First, you create a Persistent Volume (PV) that provides the storage resource.
   - Next, you create a Persistent Volume Claim (PVC) which claims usage of the PV for your application.
   - Finally, you modify your application's deployment configuration to use the PVC.

Here's how you could do it:

```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: countit-pv
spec:
  capacity:
    storage: 1Gi  # adjust the size as needed
  accessModes:
    - ReadWriteOnce
  hostPath:  # for demonstration purposes, not suitable for production
    path: "/mnt/data"
---
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: countit-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fastapi-counter
  template:
    metadata:
      labels:
        app: fastapi-counter
    spec:
      containers:
      - name: fastapi-counter-container
        image: <your-image-name>
        ports:
        - containerPort: 80
        volumeMounts:
        - name: countit-storage
          mountPath: /usr/src/app
      volumes:
      - name: countit-storage
        persistentVolumeClaim:
          claimName: countit-pvc
```

2. **StatefulSets with Persistent Volume Claims**:
   - StatefulSets are used when you have stateful applications that require stable identifiers and storage.

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: fastapi-counter
spec:
  serviceName: "fastapi-counter"
  replicas: 1
  selector:
    matchLabels:
      app: fastapi-counter
  template:
    metadata:
      labels:
        app: fastapi-counter
    spec:
      containers:
      - name: fastapi-counter-container
        image: <your-image-name>
        ports:
        - containerPort: 80
        volumeMounts:
        - name: countit-storage
          mountPath: /usr/src/app
  volumeClaimTemplates:
  - metadata:
      name: countit-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

3. **Using a Storage Class**:
   - Storage Classes provide a way to describe different “classes” of storage offered in the cluster.

```yaml
# storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fastapi-counter-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
# Then you can reference this Storage Class in your Persistent Volume and Persistent Volume Claim configurations.
```

Replace `<your-image-name>` with the name of your Docker image. Each of these options has its own set of considerations and trade-offs, so you'll need to choose based on your application's requirements and your environment.




# Nutanix Notes for Storage (Not Finished)

- RWO (Nutanix Volumes) vs RWM (Nutanix Files) or others
- relationship between pods and PVC or PV's, what gets x and why?

Simple Example LI5
- Storage Classes: different types of boxes
- PV: the actual box being granted
- PVCs: tickets asking for the box
- StatefulSet: already reserved box regular users get everytime 


----


Persisting Data on Nutanix Kubernetes
0. Create the Nutanix StorageClass (not 100% reqired since spec.storageClass if empty will default to pvc without `dynamic provisioning` being used)
1. Create PV that provides storage
2. Create PVC that uses the PV created already
3. Pod or Deploy yaml will mount the volume referencing the PVC

## Create PV
**Note:** Need to use the correct Nutanix specs for this. This will be the spec.storageClassName which will reference the provider
```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: countit-pv
spec:
  capacity:
    storage: 1Gi  # adjust the size as needed
  accessModes:
    - ReadWriteOnce
  hostPath:  # for demonstration purposes, not suitable for production
    path: "/mnt/data"
```

## Create PVC
**Note:** 

```yaml
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: countit-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```


## Sample Deployment.yaml
Note: spec.volumes will point to the pvc.claimName. Since the app source data lives in the /usr/src/app folder, mount it there to hold the required files

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fastapi-counter
  template:
    metadata:
      labels:
        app: fastapi-counter
    spec:
      containers:
      - name: fastapi-counter-container
        image: <your-image-name>
        ports:
        - containerPort: 80
        volumeMounts:
        - name: countit-storage
          mountPath: /usr/src/app
      volumes:
      - name: countit-storage
        persistentVolumeClaim:
          claimName: countit-pvc
```


# Persistent Storage with Nginx

Mounting Volumes vs Attaching Local Directory
- **Mounting:** this is helpful when you have an application that will perform some CRUD actions
- **Attaching:** more useful when you are developing real-time and need to test. Think of this as `dev-containers`

# Table of Contents
- Simple with Custom Starting Page
- Bi-Directional Development with Volume Mount to Local
- Attaching Volumes for CRUD - Sample FastAPI
- Transitioning to Kubernetes

# **Option 1:** Create a Simple Nginx Custom Starting Page
**Purpose:**
- You require static files to be ready when container loads
- Regardless of pod restarts, data will persists

Steps
1. Create a simple `custom.html` page
2. Build a simple Dockerfile and store html into defaut directory
3. Test

## 1. Create Simple HTML File
```html
<!DOCTYPE html>
<html>
<head>
    <title>Custom Nginx Page</title>
</head>
<body>
    <h1>Welcome to my custom Nginx page!</h1>
    <h3>Version v1</h3>
</body>
</html>
```

## 2. Build Dockerfile and Move

Create `Dockerfile`
```bash
FROM nginx

COPY custom.html /usr/share/nginx/html/index.html

CMD ["nginx", "-g", "daemon off;"]
```

## 3. Testing
**Build Container:** `docker build -t custom-nginx:v1`

**Run Container:** `docker run -d -p 80:80 custom-nginx:v1`

Navigate to `localhost:80`





# **Option 2:** Mounting and Exposing Volumes in Containers
**Purpose:**
- When you require access to local storage
- Perfect for Real-Time Development


Steps
1. Know the path of the local directory you would like to mount
2. Create a temp file called `index.html`
3. Pass in the location in the docker run command

### 2. Pass in Location in Docker Command
Run Container: `docker run -d -p 80:80 --name tempcustom-nginx -v /home/devops/nginx:/usr/share/nginx/html nginx`


Example `Dockerfile`
```bash
FROM nginx

COPY ./custom-nginx-html /usr/share/nginx/html

EXPOSE 80

# Note: you can change the /etc/nginx/conf.d file to change the index file name. Mount this before because it requires a restart
```


# **Option 3:** Volumes Attaching w/ FastAPI

Steps
1. Create simple .py app that will have an /increment endpoint and add to a file we can /get anytime
2. Build the Dockerfile
3. Create and Ensure Volume is attached to running container


## 1. Create `main.py`

```python
from fastapi import FastAPI
from pathlib import Path

app = FastAPI()

COUNTER_FILE = Path("/usr/src/app/counter.txt")

def read_counter():
    if COUNTER_FILE.exists():
        return int(COUNTER_FILE.read_text())
    return 0

def write_counter(count):
    COUNTER_FILE.write_text(str(count))

@app.get("/increment/")
def increment_counter():
    count = read_counter()
    count += 1
    write_counter(count)
    return {"count": count}

@app.get("/get/")
def get_counter():
    return {"count": read_counter()}
```

## 2. Create Dockerfile

```bash
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory
WORKDIR /usr/src/app

# Copy the FastAPI application
COPY ./main.py .

# Install FastAPI and Uvicorn
RUN pip install fastapi uvicorn

# Expose the port the app runs on
EXPOSE 80

# Command to run the application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

## 3. Configure Docker

**What Happens When You Dont Attach Volume?**
- Run command: `docker run -d -p 80:80 counterapp:v1`
- Output: Application works `HOWEVER` when you restart the container, the counter /get does not persist the data


Create Volume: `docker volume create countit`
Attach Volume: `docker run -d -p 80:80 --name fastapi-counter-container --volume countit:/usr/src/app fastapi-counter` *Note: you can also use the VOLUME command in the Dockerfile*

# **Option 4:** Moving to Kubernetes

**Options**
1. Leverage PVs and PVCs via HostPath
2. Leverage StatefulSets
3. Leverage StorageClass Dynamic Provisioning















