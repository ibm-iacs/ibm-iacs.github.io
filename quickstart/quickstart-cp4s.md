# IBM CP4S Quickstart ON ROKS

<!--- cSpell:ignore qube cntk autoplay allowfullscreen -->

--8<-- "quickstart-overview.md"

---

--8<-- "gitops-cluster-prereq.md"

---

--8<-- "techzone-roks-bootstrapped-cluster.md"

    !!! note
        For this IBM MQ Quickstart we recommend you to request the Red Hat OpenShift on IBM Cloud cluster with GitOps Configuration with the following size:

          * Worker Node Count = **4**
          * Worker Node Flavor = **8 CPU x 32 GB**

--8<-- "techzone-roks-bootstrapped-cluster-use.md"

---

--8<-- "quickstart-install-required-cli.md"

---

## Select resources to deploy

- Clone the `multi-tenancy-gitops` repository in your Git Organization if you have not already done so and select the K8s resources to deploy in the [infrastructure](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/1-infra/kustomization.yaml) and [services](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/2-services/kustomization.yaml) layers.

### Instructions
1. Select a profile and delete the others from the `0-bootstrap` directory.  If this is your first usage of the gitops workflow, Use the `single-cluster` profile.
    ```bash
    GITOPS_PROFILE="0-bootstrap/single-cluster"
    ```
2. Review the `Infrastructure` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/1-infra/kustomization.yaml) and un-comment the resources to deploy to match the [IBM CP4S recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/cp4s-recipe.md) .
3. Edit the CP4SThreatManagements custom resource instance and specify a block or file storage class (https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/2-services/argocd/instances/ibm-cp4sthreatmanagements-instance.yaml). The default is set to managed-nfs-storage.
4. Review the `Services` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/2-services/kustomization.yaml) and un-comment the resources to deploy to match the [IBM MQ recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/cp4s-recipe.md) .
5. Commit and push changes to your git repository
    ```bash
    git add .
    git commit -m "initial bootstrap setup"
    git push origin
    ```

## Validation
1.  Check the status of the `CommonService` and `PlatformNavigator` custom resource.
    ```bash
    # Verify the Common Services instance has been deployed successfully
    oc get commonservice common-service -n ibm-common-services -o=jsonpath='{.status.phase}'
    # Expected output = Succeeded

    # [Optional] If selected, verify the Platform Navigator instance has been deployed successfully
    oc get platformnavigator -n tools -o=jsonpath='{ .items[*].status.conditions[].status }'
    # Expected output = True
    ```
2.  Log in to the Platform Navigator console
    ```bash
    # Retrieve Platform Navigator Console URL
    oc get route -n tools integration-navigator-pn -o template --template='https://{{.spec.host}}'
    # Retrieve admin password
    oc extract -n ibm-common-services secrets/platform-auth-idp-credentials --keys=admin_username,admin_password --to=-
    ```
3. Check the status of the CommonService and PlatformNavigator custom resource.
    ```bash
    oc get CP4SThreatManagement threatmgmt -n tools -o jsonpath='{.status.conditions}'
    # Expected output = Cloudpak for Security Deployment is successful
    ```
4. Before users can log in to the console for Cloud Pak for Security, an identity provider must be configured. The documentation provides further instructions. For DEMO purposes, OpenLDAP can be deployed and instructions are provided below.

5. Download the cpctl utility
    ```bash
    #Log in to the OpenShift cluster
    oc login --token=<token> --server=<openshift_url> -n <namespace>

    #Retrieve the pod that contains the utility
    POD=$(oc get pod --no-headers -lrun=cp-serviceability | cut -d' ' -f1)

    #Copy the utility locally
    oc cp $POD:/opt/bin/<operatingsystem>/cpctl ./cpctl && chmod +x ./cpctl
    ```

6. Install OpenLDAP
    1. Start a session
    ```bash
    ./cpctl load
    ```
    1. Install OpenLDAP
    ```bash
    cpctl tools deploy_openldap --token $(oc whoami -t) --ldap_usernames 'adminUser,user1,user2,user3' --ldap_password cloudpak
    ```
1. Initial user log in
    1. Retrieve Cloud Pak for Security Console URL
    ```bash
    oc get route isc-route-default --no-headers -n <CP4S_NAMESPACE> | awk '{print $2}'
    ```
    1. Log in with the user ID and password specified (ie. `adminUser` / `cloudpak`).


