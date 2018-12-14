## Master Components: 
- makes global decision about the cluster (scheduling) and 
- detecting and responding to cluster events.

### kube-apiserver: 
- exposes k8s api
- front end of k8s control plane
- scales by deploying more instance

### etcd:
- distributed key- value store for all cluster data

### kube-scheduler:
- watches newly created pods having no node assigned

### kube-controller-manager:
- runs controller
- includes:
    - Node controller
    - Replication Controller
    - Endpoints Controller
    - Service Accounts & Token Controllers

### cloud-controller-manager:
- runs controllers that interact with the underlying cloud providers
- allows cloud vendors code and the Kubernetes core to evolve independent of each other
- controllers that have cloud provider dependencies:
    - Node Controller:
    - Route Controller:
    - Service Controller:
    - Volume Controller:

## Node Components:
- runs on every node

### kubelet:
- makes sure that containers are running in a pod
- takes set of PodSpecs 

### kube-proxy:
- enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding

### Container Runtime:
- software that is responsible for running container
- supports:
    - Docker
    - rkt
    - runc
    - any OCI runtime- spec implementation

## Addons:
- pods and services that implelment cluseter features.
- Namespaced addon objects are created in the kube- system namespace

### DNS:
- other addons are not strictly required but, all clusters should have cluster DNS
- serves DNS records for DNS records for k8s services

### Web UI:
- UI for k8s cluster
- allows users to manage and troubleshoot applications running in the cluster

### Container Resource Monitoring:
- records generic time- series metrics about containers in a central database

### Cluster- level Logging:
- saves container logs

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## Kubernetes Objects:
- persistent entities in kubernetes system
- represents the state of the cluster
- describes:
    - what container running, on which node
    - resources available
    - policies how they behave (restart policies, upgrades, fault- tolerance)
- object tells the kubernetes system what the cluster's workload will look like (desired state)
- kubectl makes API call to run k8s in the CLI
- to make k8s api call in our program we need a client (client- go)

### Object Spec and Status
- two nested object fields that govern the object's cofig
- These are:
    - object spec: (desired state)
        - the characteristics that a object is desired to have
    - status: (actual state)
        - describes actual state of the object
- Kubernetes control plane manages an object's actual state to match the desired state
- Example:
    >a Kubernetes Deployment is an object that can represent an application running on your cluster. When you create the Deployment, you might set the Deployment spec to specify that you want three 
    replicas of the application to be running. The Kubernetes system reads the Deployment spec and starts three instances of your desired application–updating the status to match your spec. If any 
    of those instances should fail (a status change), the Kubernetes system responds to the difference between spec and status by making a correction–in this case, starting a replacement instance.

#### Describing a object
- must provide desired state + some basic info (names etc)
- info are provided using .yaml file to kubectl
- kubectl converts the info to JSON
- Example Deployment object spec:


```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
- command to create from yaml: 

    `kubectl create -f <yaml link> --record`

-------------------------------------------------------------------------------------------

### Names
- client provided name of the resource

### UIDs
- k8s system generated unique id 
- every object in a cluster (in its lifetime) has a unique id

-------------------------------------------------------------------------------------------

## Namespace
- virtual clusters backed by the same physical cluster
- needs when many users spread across multiple teams or projects
- name must be unique across a namespace
- way to divide cluster resources between multiple users
- Kubernetes starts with three initial namespaces:
    - default : objects with no other namespace
    - kube-system : objects created by k8s system
    - kube-public : readable by not authenticated user

-------------------------------------------------------------------------------------------

## Pods
- Basic building block of k8s
- smallest & simplest 
- encapsulate:
    - container (or multiple containers)
    - storage
    - unique IP 
    - how the containers should run
- Pods itself does not run, it is actually an environment the container runs
- Pods don't self-heal
- Pods serve as unit of deployment, horizontal scaling, and replication. 
- Co-scheduling, shared fate (e.g. termination), coordinated replication, resource sharing, and dependency management are handled automatically for containers in a pod.
- Users don't need to create pod directly

### Durability of Pods:
- Pod is exposed as a primitive in order to facilitate:
    - scheduler and controller pluggability
    - support for pod-level operations without the need to “proxy” them via controller APIs
    - decoupling of pod lifetime from controller lifetime, such as for bootstrapping
    - decoupling of controllers and services — the endpoint controller just watches pods
    - clean composition of Kubelet-level functionality with cluster-level functionality — Kubelet is effectively the “pod controller”
    - high-availability applications, which will expect pods to be replaced in advance of their termination and certainly in advance of deletion, such as in the case of planned evictions or image prefetching.


### Usage of Pods
- One Container Pod: 
    - wrapper around a container
    - kubernetes manages the pods rather the container directly
- Multiple Container Pod:
    - encapsulate multiple container which need to share resources

### When Pods Deleted:
- If host node fails
- If the scheduling operation fails
- In the time of Node maintenance
- Lack of resources

### How pods manages multiple container:
- A pod may have a container that acts as a web server for files in a shared volume, and a separate “sidecar” container that updates those files from a remote source, as in the following diagram:
![How pods manages multiple container](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg "How pods manages multiple container")

_Fig: A multi-container pod that contains a file puller and a web server that uses a persistent volume for shared storage between the containers._
     

### Types of shared resource in pod
- _Networking:_ 
    - each pod has unique ip address
    - every container in the pod shares the network namespace including IP address and ports
    - Each container can communicate with each other using localhost
    - Applications in a pod must coordinate their usage of ports
    - Containers in different pods have distinct IP addresses and can not communicate by IPC (Inter Process Communication)
- _Storage:_
    - can specify a set of shared resources
    - all application containers in the pod can access shared volumes, allows the containers to share data

### Pod Templates
- Pod spec that includes other objects (Replication Controllers, Jobs, DaemonSets)
- Once a pod is created, pod template has no relationship with the created pod
- Changes in pod template has no effect on the pods already created 
- Example:
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

-------------------------------------------------------------------------------------------------------------------

### Uses of Pods
- content management systems, file and data loaders, local cache managers, etc.
- log and checkpoint backup, compression, rotation, snapshotting, etc.
- data change watchers, log tailers, logging and monitoring adapters, event publishers, etc.
- proxies, bridges, and adapters
- controllers, managers, configurators, and updaters

### Why not just run multiple programs in a single (Docker) container?

- Transparency: Making the containers within the pod visible to the infrastructure enables the infrastructure to provide services to those containers, such as process management and resource monitoring. This facilitates a number of conveniences for users.
- Decoupling software dependencies: The individual containers may be versioned, rebuilt and redeployed independently. Kubernetes may even support live updates of individual containers someday.
- Ease of use: Users don’t need to think about managing process.
- Efficiency: infrastructure -> responsibility, containers -> light-weight.

### Termination of pods:
- Pods are gracefully deleted. Default grace period is 30s
- Pods shows as TERMINATING 
- If preStop hooks run after the grace period, additional 2s grace period is added
- The processes are sent TERM signal
- After grace period expires, Pods are killed with SIGKILL
- Then the pod disappears from client
- To override grace periods 'kubectl delete <pod name> --grace-period=<second>'
- If grace-period is set to 0, then the pod is force deleted
- Force delete also gives a small grace period.

### Privileged mode
- to write network and volume plugins as separate pods 
- users without privilege will not be able to access these pods

------------------------------------------------------------------------------------------------------------------

## Pod Lifecycle
- Pod's status is defined by a PodStatus object, which a phase field
- the value of phase are:
    - _Pending_: The Pod has been accepted by the Kubernetes system, but one or more of the Container images has not been created. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.
    - _Running_: The Pod has been bound to a node, and all of the Containers have been created. At least one Container is still running, or is in the process of starting or restarting.
    - _Succeeded_: All Containers in the Pod have terminated in success, and will not be restarted.
    - _Failed_: All Containers in the Pod have terminated, and at least one Container has terminated in failure. That is, the Container either exited with non-zero status or was terminated by the system.
    - _Unknown_: For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.

### Pod Conditions:
- PodStatus has an array of PodConditions, through which the pod passed or not.
- Six fields:
    - lastProbeTime
    - lastTransitionTime
    - message
    - reason
    - status
    - type:
        - PodScheduled
        - Ready
        - Initialized
        - Unscheduleable
        - ContainersReady

### Container Probes:
- Probe is diagnostic performed by kubelet on a container
- kubelet calls a _Handler_ which performs the diagnostic
- 3 types of handlers:
    - _ExecAction_: Executes a specified command inside the Container. The diagnostic is considered successful if the command exits with a status code of 0.
    - _TCPSocketAction_: Performs a TCP check against the Container’s IP address on a specified port. The diagnostic is considered successful if the port is open.
    - _HTTPGetAction_: Performs an HTTP Get request against the Container’s IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.

- results of a probe
    - Success
    - Failure
    - Unknown 
- Two types of probes:
   - livenessProbe: 
       - indicates container running
       - if fails, kubelet kills the container
       - container is then subjected to its restart policy
       - if no liveness probe is provided, default is success
   - readinessProbe: 
       - indicates if the container is ready to service request
       - if fails, the endpoint controller removes pod's ip address from all services that match the pod
 
### When should you use liveness or readiness probes?
- if container crashes on its own whenever it encounters an issue or becomes unhealthy, no need for liveness probe
- if you want to send traffic to pod when a probe succeds, specify readiness probe
- the pod will start without recieving any traffic, only start receiving after probe starts succeeding
- if container needs to load large data, configuring files, specify readiness probe

### Pod readiness gate
- A field called ReadinessGate is specified in the PodSpec
- It specifies additional condition
- A pod is evaluated ready when:
    - all containers are ready
    - all conditions specified in the readiness gate are true

### Restart Policy
- PodSpec has a field called RestartPolicy. 
- Possible values are:
    - OnFailure
    - Always
    - Never
- Exited Containers that are restarted by the kubelet are restarted with an exponential back-off delay (10s, 20s, 40s …) capped at five minutes

### Pod Lifetime
- Pods don't disappear until someone destroys them (human or controller)
- But pods with Succeeded or Failed for more than a duration will expire and be auto destroyed after a specific time.
- States:
- Pod is running and has one Container. Container exits with success.
    - Log completion event.
    - If restartPolicy is:
        - Always: Restart Container; Pod phase stays Running.
        - OnFailure: Pod phase becomes Succeeded.
        - Never: Pod phase becomes Succeeded.
- Pod is running and has one Container. Container exits with failure.
    - Log failure event.
    - If restartPolicy is:
        - Always: Restart Container; Pod phase stays Running.
        - OnFailure: Restart Container; Pod phase stays Running.
        - Never: Pod phase becomes Failed.
- Pod is running and has two Containers. Container 1 exits with failure.
    - Log failure event.
    - If restartPolicy is:
        - Always: Restart Container; Pod phase stays Running.
        - OnFailure: Restart Container; Pod phase stays Running.
        - Never: Do not restart Container; Pod phase stays Running.
- If Container 1 is not running, and Container 2 exits:
    - Log failure event.
    - If restartPolicy is:
        - Always: Restart Container; Pod phase stays Running.
        - OnFailure: Restart Container; Pod phase stays Running.
        - Never: Pod phase becomes Failed.
- Pod is running and has one Container. Container runs out of memory.
    - Container terminates in failure.
    - Log OOM event.
    - If restartPolicy is:
        - Always: Restart Container; Pod phase stays Running.
        - OnFailure: Restart Container; Pod phase stays Running.
        - Never: Log failure event; Pod phase becomes Failed.
- Pod is running, and a disk dies.
    - Kill all Containers.
    - Log appropriate event.
    - Pod phase becomes Failed.
    - If running under a controller, Pod is recreated elsewhere.

- Pod is running, and its node is segmented out.
    - Node controller waits for timeout.
    - Node controller sets Pod phase to Failed.
    - If running under a controller, Pod is recreated elsewhere.

---------------------------------------------------------------------------------------------------------------------

## Init containers:
- which runs before app container starts
- like regular containers, except:
    - they always run to completion 
    - must complete successfully before another start
- If one of the init containers fails, Pod is restarted.
- Init Containers use RestartPolicy OnFailure.
- A Pod cannot be Ready until all Init Containers have succeeded
- If a pod is restarted, all init containers must restart
- Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: initonservice
  labels:
    app: initonservice
spec:
  containers:
    - name: initonservice
      image: busybox
      command:
        - sh
        - -c
        - while true; do echo app is running; sleep 1; done
      imagePullPolicy: IfNotPresent
  initContainers:
    - name: init-myservice
      image: ubuntu
      command:
        - sh
        - -c
        - "apt-get update; apt-get install dnsutils -y; until nslookup myservice; do echo waiting for myservice; sleep 2; done"
    - name: init-mydb
      image: ubuntu
      command:
        - sh
        - -c
        - "apt-get update; apt-get install dnsutils -y; until nslookup mydb; do echo waiting for mydb; sleep 2; done"
  restartPolicy: Always
```

### use of init container
- contain utilities that can't be added to app image for security reason
- provide an easy way to block or delay the startup of app Containers until some set of preconditions are met.

----------------------------------------------------------------------------------------------------------------
## Pod Preset:
- API resource for injecting additional runtime requirements into a pod at creation time
- use label selectors to specify the pods to which the preset applies
- allows not to have explicitly provide all information for every pod
- Pod presets can be disabled for a specific pod

### How it works:

- when pod creation request occures, system does
    - retrieve all PodPrestets
    - checks if label selectors of any PodPreset matches the labels of the pod
    - attempts to merge various resources defined by podpreset
    - on error, through event, and create the pod without injecting any resource from podpreset
    - annotate the the resulting modified pod spec to indicate that it has been modified by podpreset
- when podpreset applies to one or more pods
    - modifies the pod specs
    - for `Env`, `EnvFrom` & `VolumeMounts`, modifies the container spec for all containers
    - for Volume, modifies the podSpec

----------------------------------------------------------------------------------------------------------------
## Disruptions

----------------------------------------------------------------------------------------------------------------

# Controllers

## ReplicaSet
- ReplicaSet is the next-generation Replication Controller
- mostly used by Deployments as a mechanism to orchestrate pod creation, deletion and updates.
- Deployments own and manage ReplicaSet
- Example:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80
```

### Writing ReplicaSet:
- apiVersion
- kind
- metadata
- Pod template: same schema as pod
- Pod selector: 
- labels
- replicas

### Working with ReplicaSet:
- Delete:
    - kubectl delete
    - with rest api: 
    ```
    kubectl proxy --port=8080
    curl -X DELETE  'localhost:8080/apis/extensions/v1beta1/namespaces/default/replicasets/frontend' \
    > -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
    > -H "Content-Type: application/json"
    ```
- Delete Relicaset without pods
    - kubectl delete with --cascade=false
    - with rest api:
    ```
    kubectl proxy --port=8080
    curl -X DELETE  'localhost:8080/apis/extensions/v1beta1/namespaces/default/replicasets/frontend' \
    > -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
    > -H "Content-Type: application/json"
    ```
- Isolate pods from ReplicaSet:
    - for debugging or data recovery
- Scale
    - change the replica field
- can be used as target of HorizontalPodAutoscaler

### Alternatives:
- Deployment
- Bare pods
- Job
- DaemonSet

----------------------------------------------------------------------------------------------------------------

## ReplicationController (rc/rcs)
- ensures that a specified number of pod replicas are running at any one time
- makes sure a pod or set of pods are always up and available
- Example:
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
### how works:
- if too many pods, RC terminates extra pods
- if too few pods, RC starts more pods
- RC auto replaces the pod if fails, deleted or terminated

### uses:
- Rescheduling
- Scaling
- Rolling updates
- multiple release track
- with service

### Responsibility:
- RC simply ensures that the desired number of pods matches its label selector and are operational

-----------------------------------------------------------------------------------------------------------------

## Deployments:
- A Deployment controller provides declarative updates for Pods and ReplicaSets.
  
### Creating a Deployment:
```
kubectl create -f <file/url of yaml>
kubectl get deployments
kubectl rollout status <deployment name>
```

### Updating a Deployment
```
kubectl edit deployment/<name>
```

----------------------------------------------------------------------------------------------------------------

## StatefulSets
- manages stateful apps (app that saves client data from the activities of one session for use in the next session. The data that is saved is called the application’s state.)
- manages deployment and pods
- provides guarantee s about the ordering and uniqueness of the pods
- Unlike deployment it maintains a sticky identity for each of their pods.
- Each pod has a persistent identifier that it maintains across any rescheduling

### Using StatefulSets:
- requires for apps that need one of the followings:
    - Stable, unique network identifiers.
    - Stable, persistent storage.
    - Ordered, graceful deployment and scaling.
    - Ordered, automated rolling updates.
### Deployment and Scaling guarantees:
- For a StatefulSet with N replicas, when Pods are being deployed, they are created sequentially, in order from {0..N-1}.
- When Pods are being deleted, they are terminated in reverse order, from {N-1..0}.
- Before a scaling operation is applied to a Pod, all of its predecessors must be Running and Ready.
- Before a Pod is terminated, all of its successors must be completely shutdown.

### Pod management:
- OrderedReady Pod Management: waits for sequential launching of pods
- Parallel Pod Management: launch or terminate pods in parallel, not to wait for other pods.

### Update Strategies:
- OnDelete:
    - user must manually delete pods before updating
- RollingUpdate (default):
    - StatefulSet controller will auto delete and recreate each pod (largest to smallest)
    - a partition can be applied here, so that the pods greater than or equal to the partition will be updated
    

### DaemonSet:
- ensures that all (or some) nodes run a copy of a Pod.
- If node added to cluster > Pods added to it
- If node removed from cluster > Pods are garbage collected

### Typical uses of DaemonSet:
- running a cluster storage daemon, such as glusterd, ceph (Storage management soft) on each node.
- running a logs collection daemon on every node, such as fluentd or logstash.
- running a node monitoring daemon on every node, such as Prometheus Node Exporter, collectd, Dynatrace OneAgent, Datadog agent, New Relic agent, Ganglia gmond or Instana agent.

----------------------------------------------------------------------------------------------------------------

## Garbage Collection:
- The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.
- Some objects are owner of other objects
- Owned objects are called dependent
- for example, ReplicaSet is the owner of set of pods
- Owner of a object is defined in `ownerReference` field
- `ownerReference` is generally set automatically but it can be set manually too.

### Controlling deletion of dependents
- While deleting a object, whether the dependents will also be deleted can be specified
- Deleting the dependents is called `Cascading Deletion`
- Two types of cascading deletion:
    - Background: kubernetes deletes the object immediately and GC later deletes the dependent in background
    - Foreground: 
        - The object enters in `Deletion in progress mode`. In this mode:
            - visible via REST api
            - deletionTimeStamp is set
            - metadata.finalizers contains the value "ForegroundDeletion"
        - Once DPM is set then, GC deletes all the "blocking" dependents then the owner is deleted
- When Owner is deleted without the dependents, the dependents are known as `orphan`
- to Delete via REST api, set the `propagationPolicy` field on the `deleteOptions` argument when deleting an Object. Possible values include “Orphan”, “Foreground”, or “Background”.
    - BG:
    ```
    kubectl proxy --port=8080
    curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
    -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
    -H "Content-Type: application/json"
    ```
    - FG:
    ```
    kubectl proxy --port=8080
    curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
    -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
    -H "Content-Type: application/json"
    ```
    - Orphan:
    ```
    kubectl proxy --port=8080
    curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
    -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
    -H "Content-Type: application/json"
    ```
- Delete via kubectl:
    - `kubectl delete replicaset my-repset --cascade=<true/false>`
    
--------------------------------------------------------------------------------------------------------------

## Jobs
- creates one or more pods and ensures that a specified number of them successfully terminate 
- jobs track successfully completed pods
- when specified no of pods are completed, the job itself is completed
- job can run pods in parallel

### use case
- to run a pod successfully to completion. Because the job will start the pods again if its deleted for some reason

--------------------------------------------------------------------------------------------------------------

## CronJob
- creates job on a time based schedule
- times are denoted in UTC
- Limitation:
    - A cron job may not be run on its schedule at all
    - Or it may run twice in a schedule
    - For example, suppose a cron job is set to start at exactly 08:30:00 and its startingDeadlineSeconds is set to 10, if the CronJob controller happens to be down from 08:29:00 to 08:42:00, the job will not start. Set a longer startingDeadlineSeconds if starting later is better than not starting at all.
    
--------------------------------------------------------------------------------------------------------------

## Services
- Pods are born and die, they are not resurrected
- ReplicaSets create and destroy pods dynamically
- >A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service.
- the set of pods targeted by a service is determined by a label selector
- Example:
    ```
    kind: Service
    apiVersion: v1
    metadata:
      name: my-service
    spec:
      selector:
        app: MyApp
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
    ```
    - it’ll create a service named my-service
    - it targets 9376 on any pod with app=myapp label
    - service will have an IP (clusterIP)
    - maps incoming port to any targetPort
    - by default, targetPort is same as port
    - targetPort can be string, the name of a port in the pods
    - so you can change the port number that pod exposes without breaking service
    - kube-proxy watches the kubernets master for addition and removal of service
    - for each service, it opens a port(randomly chosen) on local node
    - any connecton to this(proxy) port will be proxied to the one of service’s backend pods
    - it installs iptable rules which caputre traffic to the service’s clsuterIP and port
    - then redicects that traffic to the proxy port which proxies the backend pods

### Service without selector
- cases for using service without cluster
    - You want to have an external database cluster in production, but in test you use your own databases.
    - You want to point your service to a service in another Namespace or on another cluster.
    - You are migrating your workload to Kubernetes and some of your backends run outside of Kubernetes.
- Example:
    - 
    ```
    kind: Service
    apiVersion: v1
    metadata:
      name: my-service
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
    ```
- This service has no selector
- so the corresponding endpoints will not be created
- manually map the endpoints as follows:
    ```
      kind: Endpoints
      apiVersion: v1
      metadata:
        name: my-service
      subsets:
        - addresses:
            - ip: 1.2.3.4
          ports:
            - port: 9376
      ```
### Discovering Service
- Environment variables
- DNS

### Round-Robin DNS
- a simple technique of load balancing various Internet services such as Web server, e-mail server by creating multiple DNS A records with the same name.
- For example, foo.dnsknowledge.com may be configured to return two IP address as follows:
```
  foo.dnsknowledge.com – 202.54.1.2
  foo.dnsknowledge.com – 202.54.1.3
```
- Half of the time when a user make foo.dnsknowledge.com request will go to 202.54.1.2 and rest will go to 202.54.1.3. 
- In other words, all clients would receive service from two different server, thus distributing the overall load among servers.

### why virtual IP without RR DNS?
- There is a long history of DNS libraries not respecting DNS TTLs and caching the results of name lookups.
- Many apps do DNS lookups once and cache the results.
- Even if apps and libraries did proper re-resolution, the load of every client re-resolving DNS over and over would be difficult to manage.

### Headless Service
- none in the clusterIP filed creates a Headless service
- when loadBalancing and single ip service is not needed, headless service is used
- cluster IP is not allowed
- kube proxy doesn't handle these serviceszaq
- with selector:
    - creates endpoint record
- without selector:
    - doesn't create endpoint record

### Publishing services
- For some parts of your application (e.g. frontends) you may want to expose a Service onto an external (outside of your cluster) IP address.

#### ClusterIP:
- Exposes the service on a cluster internal IP.
- Choosing this value makes the service only reachable from the cluster
- this is default service type

#### NodePort:
- Exposes the service on each Node’s IP at a static port (the NodePort)
- master allocates a port from specified range 
- each node will proxy that port in the service
- specific ip can be set to proxy the port via --nodeport-addresses
- a clusterIP service, to which the nodeport service will route, is automatically created
- node port service can be accessed from the outside of the cluster. <NODE_PORT>:<NODE_IP>

#### LoadBalancer:
- Exposes the service externally using a cloud provider’s load balancer. NodePort and ClusterIP services, 
- setting the type field to LoadBalancer will provision a load balancer for the Service
- The actual creation of the load balancer happens asynchronously
- Traffic from the external load balancer will be directed at the backend Pods
- 

#### ExternalName:
- Maps the service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value
- Services of type ExternalName map a service to a DNS name rather than to a typical selector
- example:
    ```
    kind: Service
    apiVersion: v1
    metadata:
      name: my-service
      namespace: prod
    spec:
      type: ExternalName
      externalName: my.database.example.com
    ```
    - This Service definition, for example, would map the my-service Service in the prod namespace to my.database.example.com.

---------------------------------------------------------------------------------------------------------------

## DNS for services and pods:
- Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures the kubelets to tell individual containers to use the DNS Service’s IP to resolve DNS names.

### What gets DNS names?
- every service defined in the cluster
- Assume a Service named foo in the Kubernetes namespace bar. A Pod running in namespace bar can look up this service by simply doing a DNS query for foo. A Pod running in namespace quux can look up this service by doing a DNS query for foo.bar.

### Services
- A records:
    - Normal (not headless) services are assigned a DNS record of the form `my-svc.my-namespace.svc.cluster.local`
    - this resolves to the cluster IP of the service
    - Headless services are also assigned a DNS record for a name of the form `my-svc.my-namespace.svc.cluster.local`
    - Unlike normal Services, this resolves to the set of IPs of the pods selected by the Service
- SRV records:
    - SRV records are created for named ports that are part of normal/headless service
    - for named port, `_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`
    - this resolves to `my-svc.my-namespace.svc.cluster.local`
    - for headless service, this resolves to a set of IPs of the pod
    - contains the port number and the domain name of the pod auto-generated-name.my-svc.my-namespace.svc.cluster.local

### Pods
- A records:
    - pods are assigned a DNS of form `pod-ip-address.my-namespace.pod.cluster.local`

#### Pod's hostname and subdomain
- when pods are created, hostname is metadata.name
- optional hostname field is exist in PodSpec
- when specified, takes precedence over pod’s name to be the hostname
- also have subdomain field
- pod with hostname > foo and subdomain > bar in namespace my-namespace, DNS foo.bar.my-namespace.svc.cluser.local

#### Pod DNS policies
- _Default_: The Pod inherits the name resolution configuration from the node that the pods run on. See related discussion for more details.
- _ClusterFirst_: Any DNS query that does not match the configured cluster domain suffix, such as “www.kubernetes.io”, is forwarded to the upstream nameserver inherited from the node. Cluster administrators may have extra stub-domain and upstream DNS servers configured. See related discussion for details on how DNS queries are handled in those cases.
- _ClusterFirstWithHostNet_: For Pods running with hostNetwork, you should explicitly set its DNS policy “ClusterFirstWithHostNet”.
- _None_: A new option value introduced in Kubernetes v1.9 (Beta in v1.10). It allows a Pod to ignore DNS settings from the Kubernetes environment. All DNS settings are supposed to be provided using the dnsConfig field in the Pod Spec. See DNS config subsection below.

#### Pod’s DNS Config

-----------------------------------------------------------------------------------------------------------------

### Exposing pods to cluster

### Creating a service

### Accessing a service
- Two ways:
- *Environment Var*:
    - When a Pod runs on a Node, the kubelet adds a set of environment variables for each active Service.
    - If the service is created after the pods are created then environment variables are not created
    - In that case, first delete all the pods, while the service is running and the create pods again.
    - Then the env var will set auto by deployment
    - Commands:
        ```
        kubectl scale deployment <name> --replicas=0; //deletes all the pods 
        kubectl scale deployment <name> --replicas=2; //recreates again
    ```
- *DNS*
### Securing the service
- To secure a service before exposing it, we need to do:
    - Self signed certificates for https
    - An nginx server configured to use the certificates
    - A secret that makes the certificates accessible to pods
    - Example: https://github.com/kubernetes/examples/tree/master/staging/https-nginx/
### Exposing the service

-----------------------------------------------------------------------------------------------------------------

## Ingress
- An ingress is a set of rules that allows inbound connections to reach the kubernetes cluster services.
- It is like a door to the cluster
- Kubernetes clusters are firewalled from the internet.
- It has edge routers enforcing the firewall.
- svc, pods are not directly accessible outside the cluster 
- All traffic that ends up at an edge router is either dropped or forwarded elsewhere.
- Ingress defines some rules for external traffic to route them to kubernetes endpoints
- What ingress looks like:
![What ingress looks like](https://cdn-images-1.medium.com/max/1400/1*RX1ZjiDaXIChc2b_5OYIww.png "How ingress works")
- Here, S=Service, P=Pod, N=Node.
- Example:
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: ingress-tutorial
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      backend:
        serviceName: default-http-backend
        servicePort: 80
      rules:
      - host: myminikube.info
        http:
          paths:
          - path: /
            backend:
              serviceName: echoserver
              servicePort: 8080
      - host: cheeses.all
        http:
          paths:
          - path: /stilton
            backend:
              serviceName: stilton-cheese
              servicePort: 80
          - path: /cheddar
            backend:
              serviceName: cheddar-cheese
              servicePort: 80
    ```
    - All requests to myminikube.info is routed to the service echoserver
    - requests mapping to cheeses.all/stilton is routed to the stilton-cheese service
    - requests mapping to cheeses.all/cheddar is routed to the cheddar-cheese service.
    - backend tag implies all unmatched tag should be routed to default-http-backend
    - Annotation : 
        - `ingress.kubernetes.io/rewrite-target: /` this annotation is necessary if the target services expect requests from URL i.e cheeses.all and not cheeses.all/stilton
        - if the service doesn't accept request on that path a 403 error is given. 
        - rewrite-target is rewritten with the given path before the request get's forwarded to the target backend

### Ingress Controller
- in order to ingress resources to work, cluster must have an ingress controller running
- when a user requests an ingress by POSTing an ingress resource to the API server, the ingress server is responsible for fulfilling the ingress requests, usually with a loadbalancer
- It may also configure the edge routers or additional front ends to help handle traffic.
- without IC there is no use of creating ingress resources
- IC can be written by own
- But no need as third party controllers are available


### Setting up ingress in minkube
- To run the above example of ingress on minikube, follow the below commands:
    ```
    //to enable ingress in minikube
    minikube addons enable ingress
    
    //run echoserver service
    kubectl run echoserver --image=gcr.io/google_containers/echoserver:1.4 --port=8080
    kubectl expose deployment echoserver --type=NodePort
    
    //run stilton-cheese
    kubectl run stilton-cheese --image=errm/cheese:stilton --port=80
    kubectl expose deployment stilton-cheese --type=NodePort
    
    //run cheddar-cheese service
    kubectl run cheddar-cheese --image=errm/cheese:cheddar --port=80
    kubectl expose deployment cheddar-cheese --type=NodePort
    
    //create the ingress resource
    kubectl create -f ingress-tutorial.yaml
    
    //add the hostname in /etc/hosts to access the hostname from pc via minikube
    echo "$(minikube ip) myminikube.info cheeses.all" | sudo tee -a /etc/hosts
    ```

- now test the ingress out by visiting `myminikube.info` , `cheeses.all/stilton` , `cheeses.all/cheddar` from your browser.

### Updating ingress
- `kubectl edit ingress <name>`


### Types of ingress
- Single Service Ingress:
- Simple fanout

-------------------------------------------------------------------------------------------------------------
## NetworkPolicies
- specificaiton about how groups of pods are allowed to communicate with each other and other network endpoints
- use labels to select pods and define reules which specify what traffic is allowd to the selected pods
- by default pods are non isolated, accept traffic from any source
- becomes isolated by having a network policy that selects them
- then the pod will reject nay connection that are not allowed by the policy

### NetworkPolicy Resource
- 
- Example:
    ```
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: test-network-policy
      namespace: default
    spec:
      podSelector:
        matchLabels:
          role: db
      policyTypes:
      - Ingress
      - Egress
      ingress:
      - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
            - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
        ports:
        - protocol: TCP
          port: 6379
      egress:
      - to:
        - ipBlock:
            cidr: 10.0.0.0/24
        ports:
        - protocol: TCP
          port: 5978
    ```
- Mandatory Fields: As with all other Kubernetes config, a NetworkPolicy needs apiVersion, kind, and metadata fields. For general information about working with config files, see Configure Containers Using a ConfigMap, and Object Management.
  
- spec: NetworkPolicy spec has all the information needed to define a particular network policy in the given namespace.
  
- podSelector: Each NetworkPolicy includes a podSelector which selects the grouping of pods to which the policy applies. The example policy selects pods with the label “role=db”. An empty podSelector selects all pods in the namespace.
  
- policyTypes: Each NetworkPolicy includes a policyTypes list which may include either Ingress, Egress, or both. The policyTypes field indicates whether or not the given policy applies to ingress traffic to selected pod, egress traffic from selected pods, or both. If no policyTypes are specified on a NetworkPolicy then by default Ingress will always be set and Egress will be set if the NetworkPolicy has any egress rules.
  
- ingress: Each NetworkPolicy may include a list of whitelist ingress rules. Each rule allows traffic which matches both the from and ports sections. The example policy contains a single rule, which matches traffic on a single port, from one of three sources, the first specified via an ipBlock, the second via a namespaceSelector and the third via a podSelector.
  
- egress: Each NetworkPolicy may include a list of whitelist egress rules. Each rule allows traffic which matches both the to and ports sections. The example policy contains a single rule, which matches traffic on a single port to any destination in 10.0.0.0/24.
  
- So, the example NetworkPolicy:
    - isolates “role=db” pods in the “default” namespace for both ingress and egress traffic (if they weren’t already isolated)
    - allows connections to TCP port 6379 of “role=db” pods in the “default” namespace from:
        - any pod in the “default” namespace with the label “role=frontend”
        - any pod in a namespace with the label “project=myproject”
        - IP addresses in the ranges 172.17.0.0–172.17.0.255 and 172.17.2.0–172.17.255.255 (ie, all of 172.17.0.0/16 except 172.17.1.0/24)
    - allows connections from any pod in the “default” namespace with the label “role=db” to CIDR 10.0.0.0/24 on TCP port 5978


------------------------------------------------------------------------------------------------------------------------

## Persistent Volume
- a subsystem that provides an API to the users and admin that abstracts how storage is provided from how it is consumed
- PersistentVolume:
    - is a piece of storage in the cluster that has been provisioned by an admin
    - is a resource in the cluster just like a node is a resource
    - have a lifecycle independent of any individual pod that uses the PV
- PersistentVolumeClaim:
    - is a request for storage by a user
    - similar to a pod
    - pods consumes node resource and PVC consumes PV resources
    - Pods can request specific CPU or memory and PVC can request specific size and access modes

### Lifecycle of volumes and claim
- Provisioning
    - Static provision
    - dynamic provision
- Binding
    - a control loop in the master watches for new PVCs and finds a matching PV if possible and bind them together
    - if a PV was dynamically provisioned then the control loop will always bind that PV to that PVC.
    - PVC and PV binding are one to one mapping
- Using
    - pods use claims as volumes
    - once a user claim and that claim is bound, the PV belongs to the user as long as they need
- Reclaiming:
    - when user is done with their volume, they can delete the PVC object which allows the reclaimation of the resource
    - volumes can be:
        - Retain:
            -  PVC is deleted but PV still exists and considered released
            -  The PV can be reclaimed
        - Delete
            - removes both PV and associated storage asset
 ### PV
 - Example:
    ```yaml
    apiVersion: v1
       kind: PersistentVolume
       metadata:
         name: pv0003
       spec:
         capacity:
           storage: 5Gi
         volumeMode: Filesystem
         accessModes:
           - ReadWriteOnce
         persistentVolumeReclaimPolicy: Recycle
         storageClassName: slow
         mountOptions:
           - hard
           - nfsvers=4.1
         nfs:
           path: /tmp
           server: 172.17.0.2
    ```
    - volumeMode : 
        - `raw` to use raw block
        - `filesystem` to use filesystem, default
    - Access mode:
        - ReadWriteOnce – the volume can be mounted as read-write by a single node
        - ReadOnlyMany – the volume can be mounted read-only by many nodes
        - ReadWriteMany – the volume can be mounted as read-write by many nodes
    - Class:
        - storage class
    - Reclaim policy:
        - Retain/ Delete
    - node affinity
        - which nodes this volume can be accessed from
    - Phase
        - Available - not bound to claim
        - bound - bound to clai,
        - released - deleted but not reclaimed by the cluster
        - failed - failed auto reclaimation
    
### PVC
- Example:
    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: myclaim
    spec:
      accessModes:
        - ReadWriteOnce
      volumeMode: Filesystem
      resources:
        requests:
          storage: 8Gi
      storageClassName: slow
      selector:
        matchLabels:
          release: "stable"
        matchExpressions:
          - {key: environment, operator: In, values: [dev]}
    ```
    - Resource
        - size of disk
    - selector:
       - matchLabels - the volume must have a label with this value
       - matchExpressions - a list of requirements made by specifying key, list of values, and operator that relates the key and values. Valid operators include In, NotIn, Exists, and DoesNotExist.

### Claims as volume
- Example:
    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: mypod
    spec:
      containers:
        - name: myfrontend
          image: nginx
          volumeMounts:
          - mountPath: "/var/www/html"
            name: mypd
      volumes:
        - name: mypd
          persistentVolumeClaim:
            claimName: myclaim
    ```    
    - Pods access storage by using claim as voume
    - claims must exist in the same namespace
    
-----------------------------------------------------------------------------------------------------------------------

## Volume Snap Shot
- A VolumeSnapshotContent is a snapshot taken from a volume in the cluster that has been provisioned by an administrator. It is a resource in the cluster just like a PersistentVolume is a cluster resource.
- A VolumeSnapshot is a request for snapshot of a volume by a user. It is similar to a PersistentVolumeClaim.