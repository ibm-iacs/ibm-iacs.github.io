# IBM API Connect Multi-Cluster - Create the clusters

<!--- cSpell:ignore -->

As explained in the previous overview section, we will need **at least 2 Red Hat OpenShift clusters**. It should not matter where these clusters reside (IBM Cloud, Amazon, Azure, on-premises, etc).

You can bring your own Red Hat OpenShift cluster or you can leverage the [Create the cluster](../cluster-create/ibm-cloud.md) section on this IBM API Connect in-depth tutorial to request Red Hat OpenShift clusters for the different providers available. Remember you should have at least 2 Red Hat OpenShift clusters.

!!! warning "Important"
    Make sure you create your Red Hat OpenShift clusters with the following specs:
        
      * OCP Version = **4.7**
      * Worker Size (at least) = **14 CPU x 48 GB**
      * Worker Node Count = **3**