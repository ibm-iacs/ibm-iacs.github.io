## The sample GitOps repository

Let's understand how the GitOps repository works by using it to install the
components we've highlighted in the above diagram.

In the first part of this topic, let's look at the high level structure of the
GitOps repository.

  1. *Ensure you're logged in to the cluster*

    Start a terminal window and log into your OCP cluster, substituting the
    `--token` and `--server` parameters with your values:

    ```bash
      oc login --token=<token> --server=<server>
    ```

    If you are unsure of these values, click your user ID in the OpenShift web
    console and select "Copy Login Command".

  2. *Locate your GitOps repository*

    If necessary, change to the root of your GitOps repository, which is typically
    `$HOME/git`.

    Issue the following command to change to your GitOps repository:

    ```bash
    cd $HOME/git
    cd multi-tenancy-gitops
    ```

  3. *Explore the high level folder structure*

    Use the following command to display the folder structure:

    ```bash
    tree . -d -L 2
    ```

    We can see the different folders in the GitOps repository:

    ```{ .text .no-copy }
    .
    ├── 0-bootstrap
    │   ├── others
    │   └── single-cluster
    ├── doc
    │   ├── diagrams
    │   ├── experimental
    │   ├── images
    │   └── scenarios
    ├── scripts
    │   └── bom
    └── setup
        └── ocp47
    ```

    The `0-bootstrap` folder is the key folder that contains all the available
    profiles. Within this folders are **Cluster Profiles** that control which
    resources are deployed to the cluster.

    There are other folders, but these contain utility scripts; and some document
    related information.

    Let us examine the `0-bootstrap` folder structure:

    ```bash
    tree ./0-bootstrap/ -d -L 2
    ```

    We can see the different folders:

    ```{ .text .no-copy }
    ./0-bootstrap/
    ├── others
    │   ├── 1-shared-cluster
    │   ├── 2-isolated-cluster
    │   └── 3-multi-cluster
    └── single-cluster
        ├── 1-infra
        ├── 2-services
        └── 3-apps
    ```

    If we consider `single-cluster` profile, you can see the three layers
    under that covering infrastructure, services, and applications.

  4. *Use single-cluster profile to configure the cluster*

    Use the following command to display the folder structure:

    ```bash
    tree ./0-bootstrap/single-cluster/ -L 3
    ```

    We can see the different folders:

    ```{ .text .no-copy }
    ./0-bootstrap/single-cluster/
    ├── 1-infra
    │   ├── 1-infra.yaml
    │   ├── argocd
    │   │   ├── consolelink.yaml
    │   │   ├── consolenotification.yaml
    │   │   ├── infraconfig.yaml
    │   │   ├── machinesets.yaml
    │   │   ├── namespace-baas.yaml
    │   │   ├── namespace-ci.yaml
    │   │   ├── namespace-cloudpak.yaml
    │   │   ├── namespace-dev.yaml
    │   │   ├── namespace-ibm-common-services.yaml
    │   │   ├── namespace-instana-agent.yaml
    │   │   ├── namespace-istio-system.yaml
    │   │   ├── namespace-openldap.yaml
    │   │   ├── namespace-openshift-storage.yaml
    │   │   ├── namespace-prod.yaml
    │   │   ├── namespace-robot-shop.yaml
    │   │   ├── namespace-sealed-secrets.yaml
    │   │   ├── namespace-spp-velero.yaml
    │   │   ├── namespace-spp.yaml
    │   │   ├── namespace-staging.yaml
    │   │   ├── namespace-tools.yaml
    │   │   ├── serviceaccounts-tools.yaml
    │   │   └── storage.yaml
    │   └── kustomization.yaml
    ├── 2-services
    │   ├── 2-services.yaml
    │   ├── argocd
    │   │   ├── instances
    │   │   └── operators
    │   └── kustomization.yaml
    ├── 3-apps
    │   ├── 3-apps.yaml
    │   ├── argocd
    │   │   ├── ace
    │   │   ├── apic
    │   │   ├── bookinfo
    │   │   ├── cp4a
    │   │   ├── mq
    │   │   └── soapserver
    │   └── kustomization.yaml
    ├── bootstrap.yaml
    └── kustomization.yaml
    ```

    Notice that each high level folder has an `argocd` folder. Within these
    `argocd` folders are **ArgoCD applications** that control which resources are
    deployed to the cluster.

  5. *Sample ArgoCD application*

    We'll see later how these ArgoCD applications work in detail. For now, notice
    how each layer of the architecture (**infrastructure**, **service**,
    **application**) has a set of ArgoCD applications.

    Type the following command:

    ```bash
    tree 0-bootstrap/single-cluster/1-infra/
    ```

    It shows a list of ArgoCD applications that are used to manage Kubernetes infrastructure resources:

    ```{ .text .no-copy }
    0-bootstrap/single-cluster/1-infra/
    ├── 1-infra.yaml
    ├── argocd
    │   ├── consolelink.yaml
    │   ├── consolenotification.yaml
    │   ├── infraconfig.yaml
    │   ├── machinesets.yaml
    │   ├── namespace-baas.yaml
    │   ├── namespace-ci.yaml
    │   ├── namespace-cloudpak.yaml
    │   ├── namespace-dev.yaml
    │   ├── namespace-ibm-common-services.yaml
    │   ├── namespace-instana-agent.yaml
    │   ├── namespace-istio-system.yaml
    │   ├── namespace-openldap.yaml
    │   ├── namespace-openshift-storage.yaml
    │   ├── namespace-prod.yaml
    │   ├── namespace-robot-shop.yaml
    │   ├── namespace-sealed-secrets.yaml
    │   ├── namespace-spp-velero.yaml
    │   ├── namespace-spp.yaml
    │   ├── namespace-staging.yaml
    │   ├── namespace-tools.yaml
    │   ├── serviceaccounts-tools.yaml
    │   └── storage.yaml
    └── kustomization.yaml
    ```

    Notice the `namespace-` YAMLs; we'll see in a moment how these each define an
    ArgoCD application dedicated to managing a Kubernetes namespace in our
    reference architecture.

  6. *A word on terminology*

    As we get started with ArgoCD, it can be easy to confuse the term **ArgoCD
    application** with your application. That's because ArgoCD uses the term
    **application** to refer to a Kubernetes **custom resource** that was
    initially designed to automate application management. However, an **ArgoCD
    application**  can automate the deployment of any Kubernetes resource within a
    cluster, such as namespaces or queue managers, as we'll see a little later.

    Let's now customize this repository to deploy the highlighted Kubernetes
    resources to our cluster.
