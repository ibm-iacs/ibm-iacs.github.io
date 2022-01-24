## Other important ArgoCD features

<!--- cSpell:ignore resynched consolenotification subfolders rolebindings rolebinding CICD qube cntk autoplay allowfullscreen -->

In this final section of this chapter, let's explore ArgoCD features you may
have noticed as you explored different YAML files in this chapter:

### **SyncWave**

  1. *Using* `SyncWave` *to control deployment sequencing*

      When ArgoCD uses a GitOps repository to apply ArgoCD applications to the
      cluster, it applies all ArgoCD applications concurrently. Even though
      Kubernetes use of **eventual consistency** means that resources which depend
      on each other can be deployed in any order, it often makes sense to help
      deploy certain resources in a certain order. This can avoid spurious error
      messages or wasted compute resources for example.

      Let's compare two ArgoCD YAML files.

      Firstly, let's examine the `namespace-tools` in the **infrastructure** layer:

      ```bash
      cat 0-bootstrap/single-cluster/1-infra/argocd/namespace-tools.yaml
      ```

      which shows the ArgoCD application YAML:

      ``` {.bash hl_lines="7-8" .no-copy}
      apiVersion: argoproj.io/v1alpha1
      kind: Application
      metadata:
        name: tools
        labels:
          gitops.tier.layer: infra
        annotations:
          argocd.argoproj.io/sync-wave: "100"
      ...
      ```

      Then examine the `artifactory` in the **services** layer:

      ```bash
      cat 0-bootstrap/single-cluster/2-services/argocd/instances/artifactory.yaml
      ```

      which shows the ArgoCD application YAML:

      ``` {.bash hl_lines="5-6" .no-copy}
      apiVersion: argoproj.io/v1alpha1
      kind: Application
      metadata:
        name: artifactory
        annotations:
          argocd.argoproj.io/sync-wave: "250"
        labels:
          gitops.tier.group: cntk
          gitops.tier.layer: services
        finalizers:
          - resources-finalizer.argocd.argoproj.io
      ...
      ```

      Notice the use of `sync-wave: "100"` for `tools` and how it contrasts
      with `sync-wave: "250"` for `artifactory`.  The lower `100` will be deployed
      before the higher sync-wave `250` because it only makes sense to deploy the
      pipelines into the `tools` namespace, **after** the `tools` namespace has been
      created.

      You are free to choose any number for `sync-wave`. In our deployment, we have
      chosen `100-199` for infrastructure, `200-299` for services and `300-399` for
      applications; it provides alignment with high level folder numbering such as
      `1-infra` and so on.

      In our tutorial, we're going to incrementally add infrastructure, services and
      applications so that you can understand how everything fits together
      incrementally. This makes `sync-wave` less important than in a real-world
      system, where you might be making updates to each of these deployed layers
      simultaneously. In such cases the use of `sync-wave` provides a high degree of
      confidence in the effectiveness of a deployment change because all changes go
      through in the correct order.
