# MQ High Availability Architectures: Cross Region Active Passive
<!--- cSpell:ignore clusterrole Jmeter kubeseal namespace queuemanager clusterrolebinding SPOFs MTTR MTBF qube cntk autoplay allowfullscreen -->

!!! warning
      Work in Progress

## Overview

**Audience:** Architects, Application developers, Administrators

In this topic, we're going to:

- learn the essential elements of what provides NativeHA high-availability to the latest versions of IBM MQ
- understand the impact at the infrastructure, services, and application tiers which a multi-region, high-availability requirement brings
- discuss cross-region deployment characteristics and needs
- deploy this topology to your OpenShift clusters

### Architecture Overview

![ha2-overview](./images/ha2-cr-ap-old.png)

_**NOTE:** We have to discuss applications on the cluster talking to the GLB endpoint versus having local reference the QM URL_

The following components comprise the high-availability topology at a high level. As your desired topology evolves to reduce single points of failure, this list will become more robust in nature.

1. **Virtual Private Cloud:**
    - Distribute workloads across a minimum of three (3) sets of infrastructure service resources in 3 distinct availability zones (AZ).

2. **Red Hat OpenShift Container Platform:**
    - Minimum two (2) OpenShift Compute Nodes per AZ to achieve standard OpenShift SLOs (99.99% possible).

3. **IBM MQ - Native HA Queue Manager:**
    - Minimum one (1) OpenShift Node in each AZ for normal mode operations.
    - Minimum two (2) OpenShift Nodes in each AZ during OpenShift Node maintenance.

4. **MQ Client Application:**
    - Stateless container-based client application
    - Leverage anti-affinity deployment practices where possible

5. **Cross-Region Global Load Balancer**
    - TBD
    - Discuss failover strategy (automated, manual, somewhere in between)

### Single Points of Failure

These services and infrastructure components have been identified as potential single points of failure in this topology:

1. <strike>Red Hat OpenShift Container Platform</strike>
2. <strike>In-Region Networking</strike>
3. <strike>Ingress/Egress Networking</strike>
4. In-Region Storage

### Infrastructure Requirements

This topology evolves this simplest high-availability scenario while adding minimal overhead to provide a nominal improvement in the overall availability.

- Block Storage with [ReadWriteOnce support](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)
- Multiple availability zones inside of a single region
- Multiple regions inside the same cloud provider
- A Cross-Region Global Load Balancer (ALB, NLB, or DNS still TBD) to handle traffic routing and failover

### MQ Assets

All of the high-availability topologies in this deployment guide leverage the NativeHA feature of IBM MQ for resiliency. As the topologies grow more complex, additional configurations and deployments are added to this least-common architectural denominator.

The recently released [NativeHA capability](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=openshift-native-ha-early-release) allows for multiple QueueManager pods to be running, with only one of them truly being "active" and receiving traffic from the OpenShift Route or Service. The running pods, managed by a StatefulSet on the underlying Kubernetes-based platform,  actively share log and message data between the active and passive QueueManager instances. Once the active QueueManager instance becomes unavailable for any reason, the remaining passive pods take advantage of the RAFT algorithm to elect a new leader and within seconds failover to a different QueueManager instance, while minimizing service and message unavailability.

For more details on the early release of NativeHA for IBM MQ 9.2, you can check out the official [IBM MQ Docs](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=openshift-native-ha-early-release).

Our example here sets the following properties in the QueueManager YAML file and pushes the changes to your GitOps repo:

```yaml
availability:
  type: NativeHA
  tls:
      secretName: <inter-queuemanager-tls-cert-secret>
      cipherSpec: ANY_TLS12_OR_HIGHER
storage:
  defaultClass: ibmc-vpc-block-metro-10iops-tier
  queueManager:
    enabled: true
    size: 2Gi
    type: persistent-claim
```

- The `availability.type` setting is required to be `NativeHA` to take advantage of this capability.

- The `availability.tls` setting allows for TLS-encrypted communication between QueueManager instances in the StatefulSet.

- The `storage.defaultClass` setting should point to a multi-AZ supported storage class the provides ReadWriteOnce storage capability for Block Storage.

- The `storage.queueManager` setting is required to be enabled and set to type `persistent-claim` for NativeHA fail-over capability to function.

### Client Application Recommendations

Our sample client application is available under the [Building MQ applications](../app-pipeline/topic1.md) section. There are minimal additional configuration requirements to deploy the client application connected to this topology.

**TBD** Client application configuration details

To be addressed:

- [Automatic client reconnection](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=restart-automatic-client-reconnection)
- Connects to a single URL (the global load balancer), which provides active-passive failover to the OpenShift Routes provided by the MQ QueueManager instances
- <strike>As we only have a single region deployed, there is no consideration given for message sequencing needs beyond traditional single QueueManager requirements.</strike>
- TBD
