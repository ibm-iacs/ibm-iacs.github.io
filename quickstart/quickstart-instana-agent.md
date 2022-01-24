# Instana Agent Quickstart

<!--- cSpell:ignore qube cntk autoplay allowfullscreen -->


--8<-- "quickstart-overview.md"

---

--8<-- "gitops-cluster-prereq.md"

---

--8<-- "techzone-roks-bootstrapped-cluster.md"

    !!! note
        For this Instana Agent Quickstart we recommend you to request the Red Hat OpenShift on IBM Cloud cluster with GitOps Configuration with the following size:
          
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
2. Review the `Infrastructure` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/1-infra/kustomization.yaml) and un-comment the resources to deploy to match the [Instana Agent recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/instana-recipe.md) .
3. Review the `Services` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/2-services/kustomization.yaml) and un-comment the resources to deploy to match the [Instana Agent recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/instana-recipe.md) .
4. Commit and push changes to your git repository
    ```bash
    git add .
    git commit -m "initial bootstrap setup"
    git push origin
    ```
