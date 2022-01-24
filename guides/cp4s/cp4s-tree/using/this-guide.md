# The CP4S tutorials

**Audience**: Architects, Application developers, Administrators

## Overview

In this topic we're going to:

* Introduce you to the CP4S tutorials
* Outline the structure of the tutorials

By the end of this topic you'll understand the CP4S tutorials, and how they
can help you design and build cloud native CP4S systems.

## Introduction

The purpose of the CP4S tutorials is to teach architects, developers and operations
staff how to implement a production-ready [CP4S](https://www.ibm.com/products/cloud-pak-for-security)
deployment on the [OpenShift Container Platform](http://openshift.com). It makes
extensive use of the [IBM Cloud Pak for Integration
(CP4S)](https://www.ibm.com/docs/en/cloud-paks/cp-security) and other [cloud
native technologies](https://landscape.cncf.io/) such as Tekton, Kustomize,
ArgoCD, Prometheus, Grafana and Kibana.

Cloud native systems are loosely coupled, making them resilient, manageable, and
observable. A cloud native systems brings robust DevOps automation which allows
CP4S developers, administrators and architects to make high quality changes with
predictability and minimal toil as frequently as their organization requires.

Using worked examples, you will build and run a modern CP4S deployment according
to cloud native principles, gaining hands-on experience of these technologies
and their benefits.

The tutorial is structured as a set of tutorials and it is recommended that you
follow them in order.  Earlier tutorials are foundational, whereas later ones
such as **high availability** and **disaster recovery** cover advanced features.
If you already have a good working knowledge of CP4S and OpenShift, these
more advanced topics can be attempted stand-alone. There is an intuitive table
of contents. *Next* and *Previous* links at the bottom of each page to help you
navigate.

The tutorial is intended as a complement to CP4S product documentation; where
relevant the tutorial will refer to it and other documents.

---

## Tutorial structure

This tutorial is divided into chapters that help you build, step-by-step, a set of
working CP4S deployments, based on a cloud native approach. Each chapter comprises
topics which you should complete in order to gain a full understanding. The
tutorial starts with a simple CP4S application and queue manager. It progresses to CP4S
uniform cluster, CP4S NativeHA and multiple availability zone and region
configurations.

The tutorial is structured to match the development and operations lifecycle:

* Install and upgrade
* Build and test
* Deployment
* Promotion
* Security
* Monitoring
* Scalability
* Performance
* High Availability
* Disaster recovery
* CPU, memory, network utilization
* Server consolidation

See how the table of contents on the left of this tutorial is arranged to match
this lifecycle.

---

## Architecture decisions

Throughout the tutorial you'll see alternative implementation choices:

!!! info
    **Baking**: Extending the official CP4S image for the purpose of embedding
    configuration or adding third party binaries. Baking happens at build time.

    **Frying**: Using a ConfigMap to add configuration information to an existing CP4S
    image. Frying happens at deployment time.

    [More](../../../cp4i/mq/ad/ad01-baking-vs-frying.md)

The inline text gives a summary of the architecture decision used to make the
technical choice. The link provides a full explanation of the architecture
decision.

All architecture decisions are described in the [Architecture decisions
registry](../../../cp4i/mq/ad/index.md).
