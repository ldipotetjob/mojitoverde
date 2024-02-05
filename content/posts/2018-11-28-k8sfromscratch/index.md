---
title: Kubernetes from scratch for developers
date: 2018-11-28 09:00:00
tags:
    - Kubernetes
categories:
    - container orchestration 
keywords:
    - Kubernetes
    - Akkak8s
    - orchestration
    - conatiner
---
![k8s_img](/mojitoverde/images/k8s.png#floatleft)
<br></br>
Trying to explain Kubernetes(k8s), I bring you here a guide about how to make a very fruitful study of it.

An advice before beginning the study of k8s: 
* You should study in-depth about any **application container engine**, I suggest [docker](https://docs.docker.com/) but there are many others like [rkt](https://coreos.com/rkt/), [runc](https://github.com/opencontainers/runc) or any other that follows [OCI](https://www.opencontainers.org/)  specifications.

You can find some resources that help in your journey to understand the most important aspects related to theb container engine: 
* [The documentation site](https://docs.docker.com/get-started/#containers-and-virtual-machines) can guide you with several examples and practical tasks.  
* [Docker free lab](https://training.play-with-docker.com/) is very useful too, easy and intuitive and mainly it encompasses the whole learning process in Docker. 

At this point should be obvious for everyone concepts like Cluster or Node.

![ClusterConcep_img](/mojitoverde/images/ClusterConcep.png#floatcenter)

First things to know:
1. What [k8s](https://kubernetes.io/docs/concepts/overview/) really is? It is an open source system for managing containerized applications across multiple hosts, providing basic mechanisms for deployment, maintenance and scaling of applications. 
2. The [components of k8s](https://kubernetes.io/docs/concepts/overview/components/) 

### Kubernetes basic infrastructure 


![masternode_img](/mojitoverde/images/masternode.png#floatcenter)
![kubernetesKluster_img](/mojitoverde/images/kubernetesKluster.png#floatcenter)

Our target is to deploy and finally run applications that are **living in our containers**. Our containers are living into the **k8s PODs**. These PODs are the basic building block in **k8s** and the most important aspects in this process are **our applications** hosted in our containers and living in those PODs at the same time.

Because our target is to deploy and finally run applications this is the whole picture at this point:

**A Cluster**: that contains NODEs [1 Master - 1.. n Worker] \
**Worker Nodes**: Contain : PODs \
**PODs**: Contain: Containers \
**Containers**: It contains applications with their own environments.

In the previous picture there are some elements that I haven't mentioned yet (**volume**), I will talk about it in next posts but you can get the idea that the POD's volume is similar at docker container volume in Docker.
These PODs are living in NODEs, but which NODEs? Who decides which node spin up an specific POD?
If we take a look at [the Master Node components](#kubernetes-basic-infrastructure) we can see **kube-scheduler**, it is responsible for creating a POD in a specific node. The kube-scheduler watches newly created PODs that have no node assigned, and selects a node for them to run on, this process is not so simple, [many factors taken into account for scheduling decisions](https://kubernetes.io/docs/concepts/overview/components/#kube-scheduler) include individual and collective resource requirements. Some times you can choose [in what node your POD can spin up](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

Some of the main components in k8s PODs creations process:

* controller
* custom resource definition

These are differents way to talk to the cluster:

* command line interface (**kubectl**) that  will be running commands against K8s clusters.
* [execute kubectl as a reverse proxy](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/#using-kubectl-proxy): **kubectl proxy --port=8080** & and then call the **k8s REST api**
* programmatic access that can be implemented [in different ways](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/#programmatic-access-to-the-api).

Let's check the scenario where we use the command line interface(**kubectl**) to run commands against Kubernetes cluster.

[kubectl syntax](https://kubernetes.io/docs/reference/kubectl/#syntax): kubectl [command] [TYPE] [NAME] [flags]

**kubectl + creations commands + definition document**

Every time that **kubectl + creations commands ...** take place the object defined in our definition document must be validated by **kube-apiserver** before configure data to any api objects(pods, services, replicationcontrollers, and others)

**creations commands(to create api objects)**:
* [create](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create)
* [apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) 
**definition document**: a custom resource definition[yaml or json document] that should describe a Controller used during the creation process. In our case our target is the POD creation.
In k8s, a [Controller](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/#kube-controller-manager) is a control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state **in this case we are talking about our desired state, it will be a POD.**

Let's see some definition document examples and its relations with the creation commands

```shell
# nginx-deployment.yaml document describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. 

create -f https://raw.githubusercontent.com/ldipotetjob/k8straining/develop/yamldefinitions/nginx-deployment.yaml

# nginx-repcontroller.yaml describe a ReplicationController object

create -f https://raw.githubusercontent.com/ldipotetjob/k8straining/develop/yamldefinitions/nginx-repcontroller.yaml

```

**Every thing done here is from a software developer perspective so in the same way when we are going to implement our project**:
1. Local projects/environments: means create/run everything in local. 
2. Production projects/environments: means upload to cloud environment created by ourselves or by a certified provider. 
For implement **Local projects/environments** previously indicated is very easy with [minikube](https://minikube.sigs.k8s.io/docs/start/). It is a configurable environment that let you start a k8s cluster in local.

Some interesting reference links to work with k8s:
1. The [kubectl command line](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) [**you can find here examples about how to use a kubectl command with its flags and arguments**]
2. [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
3. [Kubernetes architecture](https://kubernetes.io/docs/concepts/architecture/) 
4. [Containers](https://kubernetes.io/docs/concepts/containers/) 
5. Workloads([Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/)) 
