# Deployment on IBM Cloud

<!--- cSpell:ignore Ceph singl apiconnectcluster ibmc APIC Availability unmanaged practioners Pak Paks Quickstart qube cntk autoplay allowfullscreen -->

## Overview
<!--- This document describes the deployment of [IBM Process Mining](https://www.ibm.com/docs/en/cloud-paks/1.0?topic=foundation-process-mining) on the [RedHat OpenShift Kubernetes Service on IBM Cloud](https://www.ibm.com/cloud/openshift), known as ROKS, on [Virtual Private Cloud (VPC) Gen 2](https://www.ibm.com/cloud/vpc) infrastructure.

![PM-topology](images/ibm/arch.png)

As we can see in the topology above, the RedHat OpenShift cluster has been deployed on a MultiZone Region (MZR) data center with three availability zones where Virtual Private Cloud Gen 2 is available. IBM Process Mining [requires ReadWriteMany (RWX) storage](https://www.ibm.com/docs/en/cloud-paks/1.0?topic=platform-pre-installation-requirements). In order to offer Read Write Many storage for the applications running on our RedHat OpenShift cluster, we need to make OpenShift Data Foundation [available in our RedHat OpenShift cluster](https://cloud.ibm.com/docs/openshift?topic=openshift-deploy-odf-vpc). 

[OpenShift Data Foundation](https://www.redhat.com/en/resources/openshift-data-foundation-datasheet) is a highly available storage solution that you can use to manage persistent storage for your containerized workloads in Red Hat® OpenShift® on IBM Cloud™ clusters. In order to deploy this highly available storage solution, we need three worker nodes, one per availability zone of our MZR RedHat OpenShift cluster, with a minimum of 16 CPUs and 64 GB RAM. These nodes will be labeled appropriately so that they are only used for storage.

OpenShift Data Foundation deploys [Ceph](https://ceph.com/en/), an open-source, distributed storage system, throughout [Rook](https://rook.io/) which turns distributed storage systems into self-managing, self-scaling, self-healing storage services. The actual persistent storage used by OpenShift Data Foundation to deploy a distributed storage system using Rook and Ceph is the block storage provided by IBM Cloud Object Storage from your Virtual Private Cloud. 


It is important to note that the `storageClass` you configure OpenShift Data Foundation to request storage volumes **must be of type** [**metro**](https://cloud.ibm.com/docs/openshift?topic=openshift-vpc-block#vpc-block-reference). What metro means is that the `volumeBindingMode` of that `storageClass` will be set to `WaitForFirstConsumer` as opposed to the default `Immediate`. And what that means is that the Persistent Volume creation and allocation by the IBM Cloud Object Storage, as a result of its Persistent Volume Claim, will not happen until the pod linked to that Persistent Volume Claim is scheduled. This allows IBM Cloud Object Storage to know what Availability Zone of your MultiZone Region cluster the pod requesting storage ended up on and, as a result, to be able to provision such storage in the appropriate availability zone. Otherwise, if we used a `storageClass` whose `volumeBindingMode` was the default `Immediate`, IBM Cloud Object Storage would create and allocate the Persistent Volume in one of the Availability Zones which might not be the same Availability Zone the pod requiring such storage lives (as a result of the OpenShift pod scheduler) which would make that storage inaccessible to the pod. See Kubernetes official documentation [here](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) for further detail. The other important characteristic of the `storageClass` you configure OpenShift Data Foundation with is that the Reclaim policy **must not be Retain** for the same reason. If you retain the Persistent Volume, it might end up assigned to a pod in a different Availability Zone later, making that storage inaccessible to the pod allocated to. In summary, the `storageClassName` you would need to configure OpenShift Data Foundation to request storage volumes with will need to be of either `ibmc-vpc-block-metro-10iops-tier`, `ibmc-vpc-block-metro-5iops-tier` or `ibmc-vpc-block-metro-custom` types.

Finally, we can see that the [Process Mining components](https://www.ibm.com/docs/en/cloud-paks/1.0?topic=platform-application-components) (Process Miner, Task Miner and User Interface), get deployed on the other three worker nodes of you RedHat OpenShift cluster.

## Deploy

The official IBM Process Mining deployment instructions can be found [here](https://www.ibm.com/docs/en/cloud-paks/1.0?topic=platform-how-install-operator). However, we **strongly recommend to use a GitOps approach for managing your production environments**. That is, any interaction with your production environment will be done through committing changes to that Infrastructure, Configuration, etc as Code that is stored in a SCM repository such as GitHub that describes the desired state of your cluster. We will then leave the task to apply any needed change to our production environment to the GitOps tools, such as the RedHat OpenShift GitOps operator that uses ArgoCD for the mentioned task.

![GitOps](images/ibm/gitops.png)

To deploy IBM Process Mining on an OpenShift cluster, we are going to use the [IBM Cloud Native Toolkit GitOps Framework](https://cloudnativetoolkit.dev/adopting/use-cases/gitops/gitops-toolkit/) (**Important:** CNTK instructions to be updated). There are only five steps you need to take:

1. [Prereqs](#1-prereqs) - Make sure you have a RedHat OpenShift cluster and you are able to use the RedHat OpenShift CLI against it.
1. [Sealed Secrets](#2-sealed-secrets) - Provide the private key used to seal the secrets provided with the API Connect GitOps repository.
1. [RedHat OpenShift GitOps Operator](#3-redhat-openshift-gitops-operator) - Install the RedHat OpenShift GitOps operator which provides the GitOps tools needed for installing and managing IBM API Connect instances through the GitOps approach already explained.
1. [IBM Process Mining](#4-ibm-process-mining) - Deploy an instance of IBM Process Mining on your cluster.
1. [IBM Process Mining UI](#5-ibm-process-mining-ui) - Validate the installation of your IBM Process Mining instance by making sure you are able to log into the IBM Process Mining user interface.

### 1 - Prereqs

1. Get a clean RedHat OpenShift cluster deployed through the RedHat OpenShift Kubernetes Service on IBM Cloud where nothing else has been installed on top afterwards. This RedHat OpenShift cluster must be composed of six worker nodes where three of these will be entirely dedicated to OpenShift Data Foundation. The storage nodes must be 16 CPUs and 64 GB RAM at least.

    ![nodes](images/ibm/nodes.png)

1. Once your RedHat OpenShift cluster is available on your IBM Cloud dashboard, you must install the OpenShift Data Foundation add-on:
    1. From the OpenShift clusters console, select the cluster where you want to install the add-on.
    1. On the cluster Overview page, click Add-ons.
    1. On the OpenShift Data Foundation card, click Install.
    ![addon](images/ibm/addon.png)

1. On the cluster Nodes page, select one node per availability zone and write down it's IP address.

    ![ips](images/ibm/ips.png)

1. Log into your RedHat OpenShift web console using the button on the top right corner of your cluster dashboard on IBM Cloud.

1. Go to the Compute -> Nodes section on the navigation index on the left hand side and for each of the nodes you picked in the previous step:

    ![oc nodes](images/ibm/oc_nodes.png)

    1. Click on the node and in its Node Details dashboard, click on the Actions drop down menu that appears on the top right corner and select Edit Labels.
    ![node_details](images/ibm/node_details.png)
    1. Type `node-role.kubernetes.io/storage=true` and hit enter to add that label to the node. Click Save.
    ![labels](images/ibm/labels.png)

1. Once you have completed the process above for the three nodes you selected to be entirely dedicated to storage, you should see such role in the Compute -> Nodes section of your RedHat OpenShift cluster.

    ![nodes role](images/ibm/nodes_role.png)

1. Log into your RedHat OpenShift cluster through the RedHat OpenShift CLI in order to execute commands to interact with it through your terminal.

### 2 - Sealed Secrets

1. Create the `sealed-secrets` project. This project will host the Sealed Secrets operator that will allow us to decrypt sealed secrets stored in GitHub.

    ```
    oc new-project sealed-secrets
    ```

1. Download the private key [sealed-secrets-ibm-demo-key.yaml](https://bit.ly/demo-sealed-master) used to seal any secret contained in this demonstration and apply it to the cluster. In our case, we have included a demo IBM Entitlement Key within the API Connect GitOps GitHub repository so that we are able to pull down IBM Software.

    ```
    oc apply -f sealed-secrets-ibm-demo-key.yaml
    ```

1. **IMPORTANT: DO NOT CHECK THE FILE INTO GIT**. The private key **MUST NOT** be checked into GitHub under any circumstances. Please, remove the private key from your workstation to avoid any issues.

    ```
    rm sealed-secrets-ibm-demo-key.yaml
    ```

### 3 - RedHat OpenShift GitOps Operator

1. Clone the following GitHub repository that contains the GitOps structure that the Cloud Native Toolkit GitOps Framework understands.

    ```
    git clone https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-process-mining.git
    ```

1. Change directory into `multi-tenancy-gitops-process-mining`.

    ```
    cd multi-tenancy-gitops-process-mining
    ```

1. Install the RedHat OpenShift GitOps operator on your RedHat OpenShift cluster and wait for it to be available:

    * If your RedHat OpenShift cluster version is 4.6
    ```
    oc apply -f setup/ocp46/
    while ! kubectl wait --for=condition=Established crd applications.argoproj.io; do sleep 30; done
    ```
    * If your RedHat OpenShift cluster version is 4.7
    ```
    oc apply -f setup/ocp47/
    while ! kubectl wait --for=condition=Established crd applications.argoproj.io; do sleep 30; done
    ```

    Once the above command returns, you can open your RedHat OpenShift Web Console and check out that the RedHat OpenShift GitOps operator has been successfully installed in the `openshift-gitops` project.

    ![GitOps Operator Install](images/ibm/gitops-operator-install.png)

    As you can see in the image, the RedHat OpenShift GitOps operator also installs the RedHat OpenShift Pipelines operator and ArgoCD (which will be that GitOps tool that synchronizes the Infrastructure/Configuration as Code we have stored in GitHub with the state of the RedHat OpenShift cluster). 
  
    **Important:** The RedHat OpenShift Pipelines operator gets installed by the RedHat OpenShift GitOps Subscription **only for RedHat OpenShift version 4.6**. If your RedHat OpenShift cluster is version 4.7, you will need to install the RedHat OpenShift Pipelines operator as part of the GitOps process explained in this section. For getting such RedHat OpenShift Pipelines operator installed, you would need to specify that in the `kustomize.yaml` file for the services layer [here](https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-apic/blob/kustomize/0-bootstrap/argocd/single-cluster/2-services/kustomization.yaml#L66-L67).

1. Open the ArgoCD web console by clicking on the ArgoCD console link you can see at the top of your RedHat OpenShift web console and log in.

    ![Argo Login](images/ibm/argo-login.png)

    You can find your ArgoCD login password by executing:
    * If your RedHat OpenShift cluster version is 4.6
    ```
    oc extract secrets/argocd-cluster-cluster --keys=admin.password -n openshift-gitops --to=-
    ```
    * If your RedHat OpenShift cluster version is 4.7
    ```
    oc extract secrets/openshift-gitops-cluster --keys=admin.password -n openshift-gitops --to=-
    ```

    Once you login, you should see that your ArgoCD web console is empty as we have not deployed any Argo Application yet.

    ![Argo Empty](images/ibm/argo-empty.png)

### 4 - IBM Process Mining

1. Before being able to deploy an instance of IBM Process Mining in our RedHat OpenShift cluster, we need to provide the storage node's IP addresses to the configuration of the OpenShift Data Foundation operator. You can provide custom configuration to the OpenShift Data Foundation operator in the `ibm-odf.yaml` file you can find in this IBM Process Mining GitOps GitHub repository you cloned at the beginning under `multi-tenancy-gitops-process-mining/0-bootstrap/argocd/single-cluster/1-infra/argocd`. Edit that file and provide your storage node's IP addresses to the `worker_ip_X` configuration parameters for the OpenShift Data Foundation operator.

    ![IBM ODF](images/ibm/ibm-odf.png)

1. Install the ArgoCD Bootstrap Application

    ```
    oc apply -n openshift-gitops -f 0-bootstrap/argocd/bootstrap.yaml
    ```

    This ArgoCD Bootstrap Application will bootstrap the deployment of IBM Process Mining based on the configuration you have defined in the GitOps GitHub repository we cloned earlier. You can see that we integrate [Kustomize](https://kustomize.io/) for configuration management in the GitOps approach.

    As soon as you create this ArgoCD Bootstrap Application, the rest of the ArgoCD Applications and the respective RedHat Openshift resources these manage start to get created as a result of the synchronization process the GitOps approach is based on. You can see these ArgoCD Applications being created in the ArgoCD web console.

    ![Argo Apps Creating](images/ibm/argo-apps-creating.png)

1. If you go to the Operators -> Installed Operators section of your RedHat OpenShift cluster web console and select the `openshift-storage` project in the Project drop down list at the top, you will see that the OpenShift Container Storage operator (which has been recently renamed to OpenShift Data Foundation) is being installed.

1. If you go to the Workloads -> Pods section of your RedHat OpenShift cluster web console you should see pods being created as a result of the OpenShift Container Storage operator being told to create an OpenShift Container Storage Cluster.

1. After some time, you should see the OpenShift Container Storage operator successfully installed

    ![OCS Operator](images/ibm/ocs_operator.png)

    and the following new Storage Classes available on the Storage -> Storage Classes section of your RedHat OpenShift cluster web console

    ![Storage Classes](images/ibm/storage_classes.png)

    that will be used by the IBM Process Mining operator to create an IBM Process Mining instance.

1. If you go again to the Operators -> Installed Operators section of your RedHat OpenShift cluster web console and select the `openshift-operators` project in the Project drop down list at the top, you should see that the IBM Process Mining operator has been successfully installed as well as the IBM Automation Foundation Core and IBM Cloud Pak foundational services operators it depends on.

    ![Operators](images/ibm/operators.png)

1. If you go to the Operators -> Installed Operators section of your RedHat OpenShift cluster web console and select the `prod` project in the Project drop down list at the top, since in our IBM Process Mining GitOps process we have configured the IBM Process Mining instance to be deployed in the `prod` project, and click on the IBM Process Mining operator and then on the Process Mining tab, you should see the IBM Process Mining instance and that this is Running and Ready.

    ![Process Mining instance](images/ibm/pm.png)

1. If you go back to the ArgoCD web console, you should see all of the Argo Application in green.

    ![Argo Green](images/ibm/argo_green.png)

### 5 - IBM Process Mining UI

Now, let's make sure that our IBM Process Mining instance is up and running. For doing that, we are going to open up and log into the IBM Automation platform user interface.

1. Go to the Networking -> Routes section of your RedHat OpenShift cluster web console and select the `prod` project in the Project drop down list at the top. You should see a Route called `cpd`. Now, click on the `Location` value for that Route.

    ![CPD Route](images/ibm/cpd_route.png)

1. You will be presented with the IBM Automation platform user interface login option. Select `IBM provided credentials (admin only)`.

    ![CPD Login](images/ibm/cpd_login.png)

1. Get your IBM Automation platform admin credentials by executing

    ```
    oc -n ibm-common-services get secret platform-auth-idp-credentials -o jsonpath='{.data.admin_password}' | base64 -d && echo
    ```

1. Log into the IBM Automation platform using the credentials from previous step.

    ![CPD](images/ibm/cpd.png)

1. Click on the navigation menu icon on the top left corner to get that navigation menu displayed. Finally, click on Process Mining under the Analyze section.

    ![CPD PM](images/ibm/cpd_pm.png)

1. You should now be able to see the IBM Process Mining web console and start working with it.

    ![PM UI](images/ibm/pm_ui.png)
--->
Watson Knowledge Catalog