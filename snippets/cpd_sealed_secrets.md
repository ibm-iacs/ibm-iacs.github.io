<!--- cSpell:ignore -->

1. Create the `sealed-secrets` project. This project will host the Sealed Secrets operator that will allow us to decrypt sealed secrets stored in GitHub.

    ```
    oc new-project sealed-secrets
    ```

1. Download the private key [sealed-secrets-ibm-demo-key.yaml](https://bit.ly/demo-sealed-master) used to seal any secret contained in this demonstration and apply it to the cluster. In our case, we have included a demo IBM Entitlement Key within the GitOps GitHub repository so that we are able to pull down IBM Software.

    ```
    oc apply -f sealed-secrets-ibm-demo-key.yaml
    ```

1. Delete the pod

    ```
    oc delete pod -n sealed-secrets -l app.kubernetes.io/name=sealed-secrets
    ```

1. !!! warning "IMPORTANT WARNING: DO NOT CHECK THE FILE INTO GIT"
    The private key **MUST NOT** be checked into GitHub under any circumstances. Please, remove the private key from your workstation to avoid any issues.
    ```
    rm sealed-secrets-ibm-demo-key.yaml
    ```

1. For Cloud Pak to consume the entitlement key, restart the Platform Navigator pods

    ```
    oc delete pod -n tools -l app.kubernetes.io/name=ibm-integration-platform-navigator
    ```
