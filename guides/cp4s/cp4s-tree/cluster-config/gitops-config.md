# Cloud Native MQ GitOps Configuration

<!--- cSpell:ignore resynched consolenotification subfolders rolebindings rolebinding CICD qube cntk autoplay allowfullscreen walkthrough Walkthrough webkitallowfullscreen msallowfullscreen -->

In the [cluster configuration](gitops-tekton-argocd.md) of this chapter, we installed the
fundamental components for continuous integration and continuous deployment into
our  cluster. These included a sample GitOps repository, and ArgoCD.

In this section we're going to customize and enable the GitOps repository, so
that we can install the components highlighted in our CP4S CICD process:

We'll examine these components in more detail throughout this section of the
tutorial; here's an initial overview of their function:

* `ci-namespace` provide an execution namespace for our pipelines
* `tools-namespace` provide an execution namespace for tools such as
  Artifactory
* `cp4s-namespace` provide an execution namespace for our deployed
  CP4S when running in the development environment. Later in the tutorial, we'll add `staging` and `prod` namespaces.
* `ArgoCD applications` manage the `ci`,`tools` and `dev` namespaces.

You may have [already
noticed](gitops-tekton-argocd.md#downloading-the-sample-gitops-repository) that
the GitOps repository contains YAMLs that refer to the ArgoCD and namespaces in
the diagram above. We're now going to use these YAMLs to configure the cluster
resources using GitOps.

Becoming comfortable with the concepts by practicing GitOps will help us in
later chapters when we CP4S Cloud Pak

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

---

## Pre-requisites

Before attempting this section, you must have completed the following tasks:

- You have created an OCP cluster instance.
- You have installed on your local machine the `oc` command that matches the
  version of your cluster.
- You have also installed `npm`, `git` and `tree` commands.
- You have customized the GitOps repository and ArgoCD.

Please see the previous sections of this guide for information on how to do
these tasks.

---

## Video Walkthrough

This video demonstrates how to use ArgoCD and the GitOps repository to set up
different infra related components.

<div align="center"><iframe src="https://ibm.ent.box.com/embed/s/jb95srdf3s1oq5lj6aqo9ickfmq8po16?sortColumn=date&view=list" width="500" height="400" frameborder="0" allowfullscreen webkitallowfullscreen msallowfullscreen></iframe></div>

This is a video walkthrough and it takes you step by step through the below sections.

---

--8<-- "gitops-sample-repo-structure.md"

---

--8<-- "gitops-sample-repo-kustomize.md"

---

--8<-- "gitops-repo-argocd-connect-cp4s.md"

---

--8<-- "gitops-argocd-cm-gov.md"

!!! success "Congratulations!"
    You've used ArgoCD and the GitOps repository to set up `ci`, `tools` and `dev` namespaces. You've seen how to create ArgoCD applications that watch their respective GitOps namespace folders for details of the namespace resources they should apply to the cluster. You've seen how you can dynamically change deployed resources by updating the resource definition in the GitOps repository. Finally, you've experience how ArgoCD keeps the cluster synchronized with the GitOps repository as a source of truth; any unexpected configuration drift will be corrected without intervention.
    
    In the next section of this tutorial, we're going to deploy some services into the infrastructure namespaces we've created in this topic. These services will include Artifactory and MQ operator.
