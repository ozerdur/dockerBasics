 # Kubernetes

<b>What</b>
: System for running many different containers over multiple different machines, a system to deploy containerized apps

<br>
<b>Why</b>: When you need to run many different containers with different images

- Expects all images to already be built
- One config file per object we want to create
- We have to manually set up all networking
- 
## Components

### api
the gw in master node which is responsible of the communication from inside and outside the cluster. It is the only component that all other components can communicate directly.

### etcd
A key-value database in master which contains the definition of work to be done.

### sched (kube scheculer)   
The component that decides how to work done (which nodes to be used etc)  and writes it to etcd via api.

### c-m(contoller manager)
The component that checks the desired state and actual state all the time and sends commands to worker nodes over api to achieve the desired state

### node
Physical or virtual machines to run the containers

### kubelet
the component on the nodes to make containers run in the desired state

### k-proxy
the component on the nodes to enable containers to communicate each other via api.


## Containerd
Kubernetes use **containerd** as a container runtime. Docker also uses containerd under the hood.

## Lens
Lens is one of the official GUIs for kubernetes


## Installation

### Kubeadm
To install k8s to your own nodes. **Multipass** can be used to  create ubuntu VMs easily.


## Config Context Commands
 To get contexts from config

    kubectl config get-contexts
 To get current context from config

    kubectl config current-context
 To switch another context

    kubectl config use-context <context name>

## Kubectl output result options
    -o wide
    -o yaml
    -o json

To learn about components
    
    kubectl explain pod

To see the details of an object

    kubectl details <object type> <object name>

To see the logs

    kubectl logs <podname>
    kubectl logs -f <podname> (to attach and monitor live data)

To exec cmd on pod

    kubectl exec <podname> -- <command>
    kubectl exec -it <podname> -- <command> (to attach input output)

## Cluster

Nodes + Master form a 'Cluster'

## Master

- are machines(or vm's) with a set of programs to manages nodes
- works according to given yaml files
- Controls what each <b>Node</b> does

## Node

Virtual Machine or Physical Computer that run containers

- Can Contain multi containers

### Deployment

Controls Pods

### Pod

Smallest item we can deploy
Runs one or more closely related containers
Contains one or more container(grouping containers which are tightly coupled, like postgres,
logger, backup-manager containers)
Containers running on the same pod can communicate each other with localhost and they exist on the same node.
Every pod has a unique IP

### Service

Sets up networking in a Kubernetes Cluster
Creates Endpoint object under the hood

- ClusterIP

  - Exposes a set of pods to other object in the cluster

- NodePort
  - Exposes a container to the outside world(only good for dev purposes)
    - port: cluster port
    - targetPort: container port
    - nodePort: Outside port (if not set random 30000-32767)
- LoadBalancer
  - Legacy way of getting network traffic into a cluster (OUTDATED)
  - Makes sense only on cloud services
- Ingress
  - Exposes a set of services to the outside world

## Liveness Probe
To check container health in a custom way


## Readiness Probe
To add the container to load balancer once it is ready.

## Minikube

A program that enables us to create a single node kubernetes cluster in development environment

## Managed Solution in Production

AWS has EKS(Amazon Elastic Container Service for Kubernetes)

Google has GKE(Google Cloud Kubernetes Engine) for production

# Commands

## kubectl get pods

Lists pods

## kubectl get deployments

Lists deployments

## kubectl get services

Lists services

## kubectl get namespaces

Lists namespaces

## kubectl apply -f \<yaml file>

- Applies yaml config.
- Create objects.
- If created objects manually deleted, they are automatically recreated by master

## kubectl delete -f \<yaml file>

Removes objects created by yaml file. if these objects manually deleted, they are automatically recreated by master

## kubectl port-forward \<pod name> \<clusterPort></clusterPort>:\<containerPort>

Opens a nodePort with cmd

## kubectl describe \<object type> \<objectname>

Lists details of the object

Ex: kubectl describe pod client-pod

## kubectl set image \<object_type>/\<object_name> \<container_name>=\<new image to use>

Imperative cmd to update image

## Pod Config

- Runs a single set of containers
- Good for one-off dev purposes
- Rarely used directly in production

- Updates

  Can update:

  - image

  Can not update fields:

  - name
  - port
  - containers

## Deployment Config

- Runs a set of identical pods(one or more)
- Monitors the state of each pod, updating as necessary
- Good for dev and production

  Can update:

  - image
  - name
  - port
  - containers
## Init Containers
Used to create containers for startup tasks in pods. Actual containers will run after it.

## Label & Selectors
Labels are used as reference for selectors


## Annotations
Used for insensitive key-value unlike labels

## ReplicaSet
Automatically created by Deployment and can be used to rollback. It does not  update the config automatically if used directly. 

## Rollout Strategies
- Recreate:  deletes all pods before creating new ones
- RollingUpdate(default): does a transient update

## Volume in generic container terminology

Some type of mechanism that allows a container to access a filesystem outside itself

## Volume in Kubernetes

An <b>object</b> that allows a container to store data at the pod level

persist data after <b>container</b> delete/create(deleted when pod is deleted so not appropriate)

 - EmptyDir : Stored in the pod
 - HostPath : Stored in the worker node

## Persistent Volume

Used to persist data after <b>pod</b> delete/create

## Persistent Volume Claim (PVC)

Used to persist data after <b>pod</b> delete/create. Can use storage classes

- just an item in billboard(claim)

  Billboard (pod config)->computer store (statically provisioned PVs->factory(dynamically provisioned PVs)

  Config

  - Access Modes
    - ReadWriteOnce
      - Can be used by a single node
    - ReadOnlyMany
      - Multiple nodes can read from this
    - ReadWriteMany
      - Multiple nodes can read and write

## kubectl get pv

Lists persistent volume (allocated)

## kubectl get pvc

Lists persistent volume claims (just the menu)

## kubectl get storageclass

list storage classes(in local default but many options exist in cloud(Google Cloud Persistent Disk, Azure File, Azure Disk, AWS Block Store etc.))

## Secret
To store sensitive data. stringData is used to enter in plain text format. data is used to enter in base64 format. Creates files within the container. 

docker-registry type is used to generate docker registry image pull secrets


    kubectl create secret generic <secret_name> --from-literal=<secret_key>=<secret>
    kubectl create secret generic <secret_name> --from-file=<secret_key>=<file path> (more secure)
    kubectl create secret generic <secret_name> --from-file=<json file path> (json file name is the key) 

## Config Map
To store insensitive data. Entered in plain text format. Creates files within the container.

    kubectl create configmap  <config name> --from-literal=<config key>=<value> --from-file=<key>=<file path>


## Node Affinity
Similar to NodeSelector but has more options( operator In, NotIn, Exists, DoesNotExist)

## Pod Affinity
To select node according to the pods that it runs. 

## Taint Toleration
To restrict nodes to run only some defined nodes by taint.

## DaemonSet
To run a pod automatically on nodes.

## StatefulSet
Every pod has a pv and they are created and deleted in order. Useful to create clusters with master and workers

## Job
Used to create jobs. Pods are not deleted after completion so that the logs can be seen

## CronJob
Used to create scheduled jobs. Pods are not deleted after completion so that the logs can be seen

## Role
To create roles in namespaces
  rules:
    - apiGroups: [""]
      resources: ["secrets"]
      verbs: ["get", "watch", "list"]

## RoleBinding
To bind roles to users, groups and service accounts

## ClusterRoleBinding
To bind cluster roles to user, groups and service accounts

## ServiceAccount
Used to create a service account. Certificates and tokens are generated under /var/run/secrets/kubernetes.io/serviceaccount in pod. We can connect to kubernetes from pod

## Ingress
Prevents using multiple load balancers with a reverse proxy. Usually path based routing is  used

## Dashboard & GUIs
  - kubernetes/dashboard
  - Lens (https://k8slens.dev)

## Static Pod
Kubelet creates pods automatically according to a director(by default /etc/kubernetes/kubelet)

## NetworkPolicy
To restrict namespaces, pods and IP/Ports for ingress and egress 

## Service Mesh
- To encrypt communication between services
- To observe service mesh
- To change traffic flow by rules

Ex: Istio, Linkerd

## Custom Resource Definition
To generate custom resources. Operator needs to be created to take the necessary actions for the CRD