# Azure Red Hat OpenShift (ARO)

<!--- cSpell:ignore VNET apiserver Plex CNAME RESOURCEGROUP serviceaccount tennant azuread qube cntk autoplay allowfullscreen -->

 Overview

This repository document experiences on working with ARO cluster on OpenShift 4.6 environment 2Q 2021 deployed on a ARO
For additional information, refer to [ARO documentation](https://docs.microsoft.com/en-us/azure/openshift/tutorial-create-cluster).

---

## Prerequisites
To work with ARO, you need the following:

* Download the azure command-line utility.  You will need version 2.6.0 or later.

```bash
# On macOS
$ brew update
$ brew install azure-cli

# on Linux
$ curl -L https://aka.ms/InstallAzureCli | bash
```

* Download the jq command-line utility.

```bash
# On macOS
$ brew update
$ brew install jq

# on Ubuntu Linux
$ sudo apt-get install jq

# On Fedora
$ sudo dnf isntall jq

# Other OS
# reference https://stedolan.github.io/jq/download/ for more options
```

* Verify the account deploying ARO has the `User Access Administrator` and `Contributor` permissions.

* Log on to your Azure Subscription

```bash
# login to your azure subscription
$ az login

# If you have more than one subscription, set the subscription you want to use
$ az account set --subscription <SUBSCRIPTION ID>
```

---


### Register Resource Providers

This is a one-time setup that needs to be performed on the subscription.

```bash
az provider register -n Microsoft.RedHatOpenShift --wait
az provider register -n Microsoft.Compute --wait
az provider register -n Microsoft.Storage --wait
az provider register -n Microsoft.Authorization --wait
```

### Set up Environment Variables

```bash
LOCATION=eastus                 # the location of your cluster
CLUSTER=prodref                 # the name of your cluster
RESOURCEGROUP=aro-${CLUSTER}-rg # the name of the resource group where you want to create your cluster
```

### Create VNET Infrastructure

In order to create a VNET that will meet our [Golden Topology](../golden-topology/) requirements, follow the procedure outlined bellow

1. **Create Resource Group**.  This resource group will host a single vnet, which in turn will contain two subnets, one for master nodes and one for worker nodes

    ```bash
    az group create \
    --name $RESOURCEGROUP \
    --location $LOCATION
    ```

2. **Create a VNET**

    ```bash
    az network vnet create --resource-group $RESOURCEGROUP \
    --name $CLUSTER-vnet \
    --address-prefixes 10.0.0.0/22
    ```

3. **Create a subnet for control plane nodes**

    ```bash
    az network vnet subnet create --resource-group $RESOURCEGROUP \
    --vnet-name $CLUSTER-vnet \
    --name master-subnet \
    --address-prefixes 10.0.0.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

4. **Create a subnet for worker nodes**

    ```bash
    az network vnet subnet create --resource-group $RESOURCEGROUP \
    --vnet-name $CLUSTER-vnet \
    --name worker-subnet \
    --address-prefixes 10.0.2.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

    !!!warning
        **Infrastructure nodes** are not required on a Managed Platform like ARO
  
5. **Disable subnet private endpoint policies on master subnet**. This is required for the service to be able to connect to and manage the cluster.

    ```bash
    az network vnet subnet update \
    --name master-subnet \
    --resource-group $RESOURCEGROUP \
    --vnet-name $CLUSTER-vnet \
    --disable-private-link-service-network-policies true
    ```

### Create your cluster

Run the following command to create a cluster. If you choose to use either of the following options, modify the command accordingly:

* Optionally, you can pass your Red Hat pull secret which enables your cluster to access Red Hat container registries along with additional content. Add the `--pull-secret @pull-secret.txt` argument to your command.
* Optionally, you can use a custom domain. Add the `--domain foo.example.com` argument to your command, replacing foo.example.com with your own custom domain.
* Optionally, you can deploy private endpoints for Cluster API (`--apiserver-visibility Private`) and Ingress (`--ingress-visibility Private`).  Defaults are set to `Public` for both.
* Optionally, you can use pre-existing VNET and subnets for your cluster in separate resource groups (`--vnet-resource-group my-vnet-rg`)
* Optionally, you can configure the OpenShift SDN with the following flags:
    * Cluster Network: Use `--pod-cidr x.x.x.x/n` to set the desired `clusterNetwork CIDR`
    * Service Network: Use `--service-cidr y.y.y.y/n` to set the desired `serviceNetwork CIDR`
* Optionally, you can configure your master nodes with the following flags:
    * VM Size: Use `--master-vm-size XXXX` for different compute profile.  The default is `Standard_D8_v3`
* Optionally, you can configure your worker nodes with the following flags:
    * VM Size: Use `--worker-vm-size XXXX` for different compute profile.  The default is `Standard_D8_v3`
    * Worker Count: Use `--worker-count X` to set the number of desired worker nodes. The default is `3` worker nodes, one per Availability Zone.
    * Worker Storage Size: Use `--worker-vm-disk-size-gb XXXX` to set your desired worker storage size.  The default is `128` Gb.
* Optionally, you can use a pre-generated Service Principal for your deployment:
    * Client ID: Use the `--client-id xxxxx-xxxxx-xxxxx-xxxxx` flag to specify the ClientID or AppId
    * Client Secret: Use the `--client-secret xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` flag to specify the password for `client-id`

```bash
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet $CLUSTER-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet 
```

!!!warning
    If you don't specify your <font face="IBM Plex Mono">--pull-secret</font> your cluster will not include samples or operators from Red Hat or from certified partners.

    If you specify the <font face="IBM Plex Mono">--domain foo.example.com</font> parameter you are responsible for creating the <font face="IBM Plex Mono">api</font> and <font face="IBM Plex Mono">*.apps</font> <a href="#using-a-custom-domain">DNS A or CNAME records</a>.

### Accessing your cluster

You can obtain your cluster endpoints and kubeadmin password by querying with the `az` cli.

1. Cluster API Endpoint

    ​```bash
    az aro show --name $CLUSTER \
        --resource-group $RESOURCEGROUP \
        --query "apiserverProfile.url" -o tsv
    ```

2. Kubeadmin Password

    ```bash
    az aro list-credentials --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "kubeadminPassword" -o tsv)
    ```

3. OpenShift Web Console

    ```bash
    az aro show --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
    ```

### Using a custom domain

If you use a custom domain you must create 2 DNS A records in your DNS server for the `--domain` specified:

* **api** - points to the API Server
* **\*.apps** - points to the Ingress

Once your cluster is up, run the following commands to obtain your `api` and `*.apps` endpoints and update your DNS zone.

```bash
# obtain IP Address for API Server
az aro show --name $CLUSTER \
  --resource-group $RESOURCEGROUP \
  --query "apiserverProfile.ip" -o tsv

# obtain IP Address for Ingress
az aro show --name $CLUSTER \
  --resource-group $RESOURCEGROUP \
  --query "ingressProfiles[0].ip" -o tsv
```

### Creating a ReadWriteMany storage class using azure-file (SMB)

!!!warning
    The default azure-file backed storage is not POSIX compliant.  An NFS backed POSIX compliant azure-file storage is currently in Tech Preview.  Please see <a href="https://github.com/ekleinso/openshift-azure-csi-nfs">here</a> for implementation details.

By default, your ARO cluster will contain the storage class `managed-premium` that can dynamically provision ReadWriteOnce PersistentVolumes using `azure-disk`. If your CloudPak requires a RWX storage class, you can create one using `azure-file`  with the following process:

1. Obtain the managed resource group for your ARO cluster.  This is not the same as the $RESOURCEGROUP that we've used to deploy other components into.

    ```bash
    MANAGED_RG="aro-$(az aro show \
        --name $CLUSTER \
        --resource-group $RESOURCEGROUP \
        --query "clusterProfile.domain" -o tsv)"
    ```

    !!!warning
        If you deploy with a custom domain (<font face="IBM Plex Mono">--domain foo.example.com</font>), the value of clusterProfile.domain will be your domain FQDN.  Log on to your azure portal, find the <font face="IBM Plex Mono">aro-$RANDOM</font> resource group for your cluster, and manually set the MANAGED_RG variable

2. Obtain the Storage Account created in the Managed Resource Group

    ```bash
    MANAGED_SA=$(az storage account list \
    --resource-group $MANAGED_RG \
    --query "[?starts_with(name, 'cluster')].name" -o tsv)
    ```

3. Log on to your openshift cluster via command line

    ```bash
    APISERVER=$(az aro show --name $CLUSTER \
        --resource-group $RESOURCEGROUP \
    --query "apiserverProfile.url" -o tsv)
    PASSWORD=$(az aro list-credentials --name $CLUSTER \
        --resource-group $RESOURCEGROUP \
    --query "kubeadminPassword" -o tsv)

    oc login $APISERVER -u kubeadmin -p $PASSWORD --insecure-skip-tls-verify
    ```

4. Create a ClusterRole to access secrets

    ```bash
    cat << EOF | oc create -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
    name: system:azure-file-volume-binder
    rules:
    - apiGroups: ['']
    resources: ['secrets']
    verbs:     ['get','create']
    EOF
    ```

5. Assign ClusterRole to a ServiceAccount

    ```bash
    oc adm policy add-cluster-role-to-user \
        system:azure-file-volume-binder \
        system:serviceaccount:kube-system:persistent-volume-binder
    ```

6. Create the azure-file storage class

    ```bash
    cat << EOF | oc create -f -
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
    name: azure-file
    provisioner: kubernetes.io/azure-file
    parameters:
    storageAccount: $MANAGED_SA
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
    allowVolumeExpansion: true
    EOF
    ```

### Integrating with Azure AD

1. Set up Environment Variables

    ```bash
    SP_AAD_PASSWORD='Y0urS3cr3tPa55w@rd'
    AZURE_TENNANT_ID=$(az account show --query tenantId -o tsv)
    CONSOLE_URL=$(az aro show --name $CLUSTER \
        --resource-group $RESOURCEGROUP \
        --query "consoleProfile.url" -o tsv)
    OAUTH_ENDPOINT=$(echo $CONSOLE_URL|sed 's/console-openshift-console/oauth-openshift/')
    ```

2. Log on to your openshift cluster via command line (if needed)

    ```bash
    APISERVER=$(az aro show --name $CLUSTER \
        --resource-group $RESOURCEGROUP \
        --query "apiserverProfile.url" -o tsv)
    PASSWORD=$(az aro list-credentials \
        --name $CLUSTER \
        --resource-group $RESOURCEGROUP \
        --query "kubeadminPassword" -o tsv)
    oc login $APISERVER -u kubeadmin -p $PASSWORD --insecure-skip-tls-verify
    ```

3. Create a secret for Azure AD integration

    ```bash
    oc create secret generic openid-client-secret-azuread \
    --from-literal=clientSecret=$SP_AAD_PASSWORD \
    -n openshift-config
    ```

4. Create an App Registration with callbacks to your console URL

    ```bash
    AZUREAD_APPID=$(az ad app create --display-name $CLUSTER-azuread-auth \
    --reply-urls $OAUTH_ENDPOINT/oauth2callback/AAD \
    --homepage $CONSOLE_URL \
    --identifier-uris $CONSOLE_URL \
    --password $SP_AAD_PASSWORD \
    --query appId -o tsv)
    ```

5. Update app optionalClaims

    ```bash
    cat > /tmp/manifest.json<< EOF
    [{
    "name": "upn",
    "source": null,
    "essential": false,
    "additionalProperties": []
    },
    {
    "name": "email",
    "source": null,
    "essential": false,
    "additionalProperties": []
    }]
    EOF
    ```

    ```bash
    az ad app update \
    --set optionalClaims.idToken=@/tmp/manifest.json \
    --id $AZUREAD_APPID
    ```

6. Update AAD app scope permissions

    ```bash
    az ad app permission add \
    --api 00000002-0000-0000-c000-000000000000 \
    --api-permissions 311a71cc-e848-46a1-bdf8-97ff7156d8e6=Scope \
    --id $AZUREAD_APPID
    az ad app permission grant \
    --id c89c64ce-2061-4c51-846a-d318a07e40fa \
    --api 00000002-0000-0000-c000-000000000000
    ```

    !!!warning 
        You may need to ask a subscription administrator with Active Directory Graph access to run
        <font face="IBM Plex Mono">az ad app permission grant --id c89c64ce-2061-4c51-846a-d318a07e40fa --api 00000002-0000-0000-c000-000000000000</font>
        for you

7. Update OpenShift OAuth

    ```bash
    cat << EOF | oc replace -f -
    apiVersion: config.openshift.io/v1
    kind: OAuth
    metadata:
    name: cluster
    spec:
    identityProviders:
    - name: AAD
        mappingMethod: claim
        type: OpenID
        openID:
        clientID: ${AZUREAD_APPID}
        clientSecret:
            name: openid-client-secret-azuread
        extraScopes:
        - email
        - profile
        extraAuthorizeParameters:
            include_granted_scopes: "true"
        claims:
            preferredUsername:
            - email
            - upn
            name:
            - name
            email:
            - email
        issuer: https://login.microsoftonline.com/${AZURE_TENNANT_ID}
    EOF
    ```

## Cleanup

Remove your cluster and all associated resources from your Azure subscription

```bash
az aro delete -g $RESOURCEGROUP -n $CLUSTER -y
az ad app delete --id $AZUREAD_APPID
az group delete -g $RESOURCEGROUP --no-wait -y
```

