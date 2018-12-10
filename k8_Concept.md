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
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
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