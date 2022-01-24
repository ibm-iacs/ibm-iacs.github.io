# Creating a Red Hat OpenShift cluster

<!--- cSpell:ignore  -->

**Audience**: Architects, Application developers, Administrators

## Overview

In this topic we're going to:

* Introduce you to IBM Technology Zone
* Provision an OpenShift cluster

By the end of this topic you'll have created an OpenShift cluster for the MQ
tutorials.

---

## IBM Technology Zone

[IBM Technology Zone](https://techzone.ibm.com/) is a one-stop shop to get
access to technical environments and software for demos, prototyping and
deployment. We're going to use it to provision the OpenShift cluster used by our
tutorial.

Technology zone can be used to provision a Red Hat OpenShift cluster on different
infrastructure including IBM Cloud, Azure, AWS and VMWare. There is also a
degree of flexibility to configure the compute, storage and network for these
infrastructures. The MQ tutorial will work on all of these platforms.

It is also possible to provision a cluster via the command line using the
instructions in the [Hands-on
section](../../../../../infrastructure/infra-overview/#cluster-infrastructure-hands-on)
of this guide.

---

## Decide infrastructure

Decide which infrastructure platform you are going to use for your Red Hat
OpenShift cluster from the available options:

-  [Red Hat OpenShift on IBM Cloud](#openshift-on-ibm-cloud)

Click on your chosen infrastructure link above or scroll to down to the
appropriate section in this topic.

---

--8<-- "techzone-roks-empty-cluster.md"

---

Congratulations, you've now created your cluster.  In the next topic, we're
going to configure the cluster for GitOps, so that we can manage it quickly,
easily and effectively.