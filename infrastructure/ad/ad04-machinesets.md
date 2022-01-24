<!--- cSpell:ignore autoscaler -->

# AD04 - MachineSets

[< Back to architecture decision registry](../../ad/)
  
## MachineSets

***
|  **Subject Area:**      |
|-------------------------|
|    Topology             |

***

### Issue or Problem Statement

With OpenShift 4, RedHat moved to an operator-based platform. Think of these operators as code written to ease the operational complexity and capture the history and experience of managing another complex piece of software. The Machine Config Operator (MCO) is a cluster operator meant to abstract the complexities of managing the care and feed of the operating system, keeping cluster members in a consistent and well-known configuration.

At its heart, MCO manages `Machine` objects. Think of a Machine as an abstraction of the compute, storage and network characteristics that make up your cluster nodes. The `MachineSet` object is a representation of similar types of Machines. It represents machines that have similar characteristics (compute, storage and network) as well as the desired number of machines _of this type_ that a cluster should have. MachineSets are to Machines as ReplicaSets are to Pods. If you need to scale up or down your cluster, you change the replica count on your MachineSet and MCO handles all the provisioning or deprovisioning of cluster nodes against your cloud infrastructure. In a default installation (IPI), master or control plane nodes are created as Machines, and compute or worker nodes are created as MachineSets.

***

### Givens & Assumptions

* Your cluster is deployed using `openshift-install` leveraging Installer Provisioned Infrastructure (IPI)
* A key requirement for deploying your cluster with MachineSets in your cloud provider is an IP Address Management system. When deploying OpenShift clusters to cloud providers like Azure, AWS or GCP, the IP address allocation is handled by the underlying cloud provider network fabric. When deploying to OnPrem solutions like vSphere, DHCP is required to provide initial connectivity to the internal API endpoint so that your cluster nodes can obtain its ignition configuration. While deploying OpenShift clusters OnPrem without DHCP is possible, deploying MachineSets into these clusters is not.

***

### Motivation

Similar to how ReplicaSets provide resiliency to Pods, MachineSets provide resiliency and scalability to their OpenShift infrastructure. As developers start deploying more and more workloads into their cluster, they might discover that tje initial cluster compute requirements no longer meet their enterprise needs, or experience burst events where their traffic load exceeds their initial expectations. MachineSets allow them to quickly add more compute resources to your cluster and release these resources when they're no longer need.

***

### Alternatives

#### 1. Manual Provisioning

**Considerations**:  

* No integration with cloud provider
* GitOps Automation is not possible

### Justification

Let's consider the following three scenarios where MachineSets can be leveraged to quickly deploy and scale your clusters.

#### Larger Capacity Nodes

Like mentioned before, the default worker node size is 2 CPUs, 8GB of memory and 120GB disk storage.  If you're deploying a well-behaved application into your OpenShift cluster, it will define `requests` and `limits` for CPU and memory usage.  If these parameters are large enough, the cluster scheduler will not be able to "fit" your application in a single worker node. Creating nodes with a larger hardware footprint will solve this issue for you.  

#### Infrastructure Nodes

A typical OpenShift 3.x cluster has 3 types of cluster nodes: masters, workers, and infrastructure.  In OpenShift 4.x clusters, creating infrastructure nodes is no longer a Day-1 activity.  In a production environment, you still want to create infrastructure machines to make sure all the infrastructure workloads have enough capacity to run in your environment.  You don't want your ingress to contest for resources against your application workloads, as this could lead to performance issues with your router pods.  You don't want your cluster monitoring solution to starve out your applications.  Creating a MachineSet for infrastructure specific resources (ingress, monitoring, image registry and cluster logging) is the first step in resolving this issue.  A good rule of thumb is to create a MachineSet with 3 replicas and 4 CPUs and 16GB of memory for its compute resources.

#### Cluster AutoScaling

The cluster autoscaler adjusts the size of an OpenShift Container Platform cluster to meet its current deployment needs. It uses declarative, Kubernetes-style arguments to provide infrastructure management that does not rely on objects of a specific cloud provider.

The cluster autoscaler increases the size of the cluster when there are pods that failed to schedule on any of the current nodes due to insufficient resources or when another node is necessary to meet deployment needs. Similarly, the cluster autoscaler decreases the size of the cluster when some nodes are consistently not needed for a significant period, such as when it has low resource use and all of its important pods can fit on other nodes.

In order to [configure Cluster AutoScaling](https://docs.openshift.com/container-platform/4.7/machine_management/applying-autoscaling.html), MachineSets need to be provided so the cluster knows what type of nodes you want to scale.

### Implications

N/A

| Derived requirements |
|----------------------|
| N/A                  |

| Related decisions    |
|----------------------|
|  N/A                 |
