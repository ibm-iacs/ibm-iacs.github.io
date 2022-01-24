# Cloud Native MQ 

<!--- cSpell:ignore CNCF qube cntk autoplay allowfullscreen -->

## Overview
**Audience**: Architects, Application developers, Administrators

In this introduction to Cloud Native MQ, we're going to:

* Examine a system context diagram for an MQ deployment
* Identify the different categories of concern for production operations
* Understand how the
  [IBM CloudPak for
  Integration](https://cloud.ibm.com/docs/cloud-pak-integration) is used to
  build production deployments
* Highlight the benefits of a cloud native approach to MQ deployment

---

## System context

Examine the following [system context](https://en.wikipedia.org/wiki/System_context_diagram) diagram:

![Application-Landscape](images/mq-system-context.png)

We can see the different entities that interact with a typical MQ deployment.
These include physical users as well as applications and systems.

This guide is divided into chapters that help you build, step-by-step, a working
MQ deployment, based on a cloud native approach. The guide addresses all the
major aspects and best practices for:

* Installation
* Deployment
* Configuration management
* Operational monitoring
* High Availability
* Security
* Disaster recovery
* Performance

The guide makes extensive use of the [IBM Cloud Pak for Integration
(CP4I)](https://cloud.ibm.com/docs/cloud-pak-integration) and other cloud native
technologies. We'll use the the containers, operators, microservices, immutable
infrastructure and declarative APIs provided by the CloudPak to to create a
best-practice, production-ready MQ deployment. Alongside the CloudPak, we'll
also learn how [CNCF technologies](https://landscape.cncf.io/) such as
Kubernetes, Operators, Tekton, ArgoCD, or Prometheus integrate with the CloudPak
in a production environment.

Cloud native systems like MQ are loosely coupled, making them resilient,
manageable, and observable. A cloud native approach also brings robust DevOps
automation which allows MQ developers, administrators and architects to make
high quality changes with predictability and minimal toil as frequently as their
organization requires.

Now that we've identified the entities that interact with a cloud native MQ
deployment, let's examine the architecture of an MQ deployment in more detail.

