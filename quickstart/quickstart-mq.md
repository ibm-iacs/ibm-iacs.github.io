# IBM MQ Quickstart

<!--- cSpell:ignore qube cntk autoplay allowfullscreen -->

--8<-- "quickstart-overview.md"

---

--8<-- "gitops-cluster-prereq.md"

---

--8<-- "techzone-roks-bootstrapped-cluster.md"

    !!! note
        For this IBM MQ Quickstart we recommend you to request the Red Hat OpenShift on IBM Cloud cluster with GitOps Configuration with the following size:

          * Worker Node Count = **3**
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
2. Review the `Infrastructure` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/1-infra/kustomization.yaml) and un-comment the resources to deploy to match the [IBM MQ recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/mq-recipe.md) .
3. Review the `Services` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/2-services/kustomization.yaml) and un-comment the resources to deploy to match the [IBM MQ recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/mq-recipe.md) .
4. Commit and push changes to your git repository
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
1.  Log in to the Platform Navigator console
    ```bash
    # Retrieve Platform Navigator Console URL
    oc get route -n tools integration-navigator-pn -o template --template='https://{{.spec.host}}'
    # Retrieve admin password
    oc extract -n ibm-common-services secrets/platform-auth-idp-credentials --keys=admin_username,admin_password --to=-
    ```