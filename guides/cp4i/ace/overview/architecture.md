# App Connect Enterprise Architecture

<!--- cSpell:ignore qube cntk autoplay allowfullscreen -->

## Introduction

**Audience**: Architects, Integration developers, Integration Administrators, Site Reliability Engineers

In this topic, we're going to:

* Examine a high-level production deployment for ACE
* Examine the Kubernetes runtime component
* Understand the the GitOps model for ACE configuration and management
* Identify the different users of an ACE deployment

At the end of the topic, you will understand the major components for a production-ready, cloud native App Connect Enterprise deployment.

---

## Target architecture

The following diagram shows a typical ACE deployment:

![Architecture Overview](images/ace-arch.png)

---
You'll notice the major components that are essential to a production-ready ACE cloud native deployment:

* A Kubernetes cluster containing:
  * ACE applications
  * Cloud native components such as Tekton, ArgoCD, Kibana and Grafana which will help operate this environment


* GitHub as a source of truth for the cluster runtime containing:
  * ACE application source (one Github repository per application)
  * ACE shared library source (one Github repository per shared library)
  * ACE application configuration artifacts
  * CI/CD artifacts describing Tekton tasks and Pipelines

This use of Git as a source of truth follows the [GitOps model](https://www.openshift.com/blog/introduction-to-gitops-with-openshift) for the configuration and management of an MQ deployment in a Kubernetes cluster. We'll learn more about this model throughout the guide.

Notice the set of users who interact with these components:

* Integration Developers
* Integration Administrators
* DevOps specialists and Site Reliability Engineers
* Architects
* Business analysts
* ACE application users

In this guide, we'll see how these users work within this environment. 