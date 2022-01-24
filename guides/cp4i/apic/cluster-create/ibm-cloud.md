# Creating a cluster on IBM Cloud

!!! abstract
    This document explains what are the options to create a Red Hat OpenShift cluster on IBM Cloud.


## Create a Red Hat OpenShift cluster instance.

You can choose either IBM Technology Zone or your IBM Cloud account to create a Red Hat OpenShift cluster.

1. You can use IBM Technology Zone to request a Red Hat OpenShift cluster on IBM Cloud. See the instructions [here](../../mq/cluster-create/ibm-setup.md). The Red Hat OpenShift cluster on IBM Cloud you will be provided with will be hosted on **classic** infrastructure.
1. You can use your IBM Cloud account to create a new Red Hat OpenShift cluster following these instructions [here](https://cloud.ibm.com/docs/openshift?topic=openshift-getting-started). You can choose to create your Red Hat OpenShift cluster on IBM Cloud either on **classic** infrastructure or on **Virtual Private Cloud (VPC)** infrastructure.

!!! warning "Important"
    Make sure you create your Red Hat OpenShift cluster on IBM Cloud with the following specs:
        
      * OCP Version = **4.7**
      * Worker Size (at least) = **14 CPU x 48 GB**
      * Worker Node Count = **3**

!!! note
    It is important that you are aware and remember what kind of infrastructure, **classic** or **vpc**, your Red Hat OpenShift cluster on IBM Cloud has been deployed onto as the type of storage that IBM API Connect will require depends on this.

### Tools

In order to interact with your Red Hat OpenShift cluster(s) and complete this tutorial successfully, we strongly recommend to install the following tools in your workstation.

1. The `oc` command that matches the version of your cluster. Use these [instructions](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html) to get the latest version of `oc`. Use `oc version` to confirm that you have
    - `Client Version: 4.6` or higher
    - `Server Version: 4.7` or higher
1. The `npm`, `git`, `tree` and `jq` commands.