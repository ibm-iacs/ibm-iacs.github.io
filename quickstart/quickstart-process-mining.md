# IBM Process Mining Quickstart

<!--- cSpell:ignore qube cntk autoplay allowfullscreen Codespaces -->


--8<-- "quickstart-overview.md"

---

--8<-- "gitops-cluster-prereq.md"

---

--8<-- "techzone-roks-bootstrapped-cluster.md"

    !!! note
        For this IBM Process Mining Quickstart we recommend you to request the Red Hat OpenShift on IBM Cloud cluster with GitOps Configuration with the following size:
          
          * Worker Node Count = **3**
          * Worker Node Flavor = **16 CPU x 64 GB**

--8<-- "techzone-roks-bootstrapped-cluster-use.md"

---

--8<-- "quickstart-install-required-cli.md"

---

--8<-- "quickstart-select-resources.md"

### Instructions

1. You can execute the following steps either locally by cloning the `multi-tenancy-gitops` GitHub repository the IBM Technology Zone environment request automation has forked into the GitHub organization you created in the Pre-requisites section above to your local workstation (and then make changes from you local workstation and commit and deliver those) or by using the new VSCode extension of GitHub Codespaces capability straight from your web browser (just open the `multi-tenancy-gitops` repository in your Git Organization in your browser and press the `.` key):
   
    1. Review the `Infrastructure` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/1-infra/kustomization.yaml) and un-comment the resources to deploy to match the [IBM Process Mining recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/process-mining-recipe.md#infrastructure---kustomizationyaml).
    1. Review the `Services` layer [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/2-services/kustomization.yaml) and un-comment the resources to deploy to match the [IBM Process Mining recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/process-mining-recipe.md#services---kustomizationyaml).
    1. Commit and push changes to your git repository (the following code refers to the commands when you cloned the `multi-tenancy-gitops` repository locally on your workstation)
    ```bash
    git add .
    git commit -m "initial bootstrap setup"
    git push origin
    ```

You can go to the Deploy Process Mining section of the in-depth IBM Process Mining tutorial [here](../guides/cp4ba/process-mining/deployment/ibm-cloud.md#ibm-db2-ibm-entitlement-key) to make sure your IBM Process Mining quickstart deployment is successful and see the instructions for accessing the IBM Process Mining User Interface.