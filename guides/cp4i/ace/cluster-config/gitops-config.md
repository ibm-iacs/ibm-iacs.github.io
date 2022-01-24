# IBM AppConnect Enterprise GitOps Configuration

<!--- cSpell:ignore resynched consolenotification subfolders rolebindings rolebinding CICD qube cntk autoplay allowfullscreen -->

In the [cluster configuration](gitops-tekton-argocd.md) of this chapter, we installed the fundamental components for continuous integration and continuous deployment into our cluster. These included a sample GitOps repository, and ArgoCD.

In this section we're going to customize and enable the GitOps repository, so that we can install IBM AppConnect Enterprise(ACE) and all of the components that go along with that. We will examine these components in more detail throughout this section of the
tutorial as well.

In this topic, we're going to:

* Explore the sample GitOps repository in a little more detail
* Customize the GitOps repository for our cluster
* Connect ArgoCD to the customized GitOps repository
* Bootstrap the cluster
* Explore how the `ci` namespace is created
* Try out some dynamic changes to the cluster with ArgoCD
* Explore how ArgoCD manages configuration drift

By the end of this topic we'll have a cluster up and running, having used GitOps
to do it. We'll fully understand how ArgoCD manages cluster change and
configuration drift.

## Pre-requisites

Before attempting this section, you must have completed the previous section of this tutorial where you created your Red Hat OpenShift cluster(s) and customized the GitOps repository and ArgoCD. Please see the previous sections of this guide for information on how to do these tasks.

---

--8<-- "gitops-sample-repo-structure.md"

--8<-- "gitops-sample-repo-kustomize.md"

--8<-- "gitops-repo-argocd-connect.md"

--8<-- "gitops-argocd-cm-gov.md"

!!! success "Congratulations!"
    You've used ArgoCD and the GitOps repository to set up `ci`, `tools` and `dev` namespaces. You've seen how to create ArgoCD applications that watch their respective GitOps namespace folders for details of the namespace resources they should apply to the cluster. You've seen how you can dynamically change deployed resources by updating the resource definition in the GitOps repository. Finally, you've experience how ArgoCD keeps the cluster synchronized with the GitOps repository as a source of truth; any unexpected configuration drift will be corrected without intervention.
