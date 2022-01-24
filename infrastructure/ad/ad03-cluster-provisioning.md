# AD03 - Cluster Provisioning

[< Back to architecture decision registry](../../ad/)
 
## Cluster Provisioning

***
|  **Subject Area:**      |
|-------------------------|
|    Topology             |

***
### Issue or Problem Statement

The recommended approach to deploying OpenShift Container Platform is using the Installer Provisioned Infrastructure (IPI) method.

***
### Givens & Assumptions

* **Given** Production clusters will implement best practices to deploy in a highly available, resilient infrastructure
* **Given** Cluster needs to be easily reproducible

***
### Motivation

...

***

### Alternatives

#### 1. Deploy using terraform
If the openshift-installer doesn't meet customer's constraints, then the following implementations are available to deploy OpenShift in different cloud providers.
<InlineNotification kind="warning">
These terraform implementations are meant to be used as a base and modified by the OpenShift deployer to meet requirements.  Some modification is expected.
</InlineNotification>

Azure:
***
| **Cloud Provider** | **Github Repository** |
|-------------------------|-------------------------|
| Azure | https://github.com/ibm-cloud-architecture/terraform-openshift4-azure |
| AWS | https://github.com/ibm-cloud-architecture/terraform-openshift4-aws |
| GCP | https://github.com/ibm-cloud-architecture/terraform-openshift4-gcp |
| vSphere | https://github.com/ibm-cloud-architecture/terraform-openshift4-vmware |
| VCD | https://github.com/ibm-cloud-architecture/terraform-openshift4-vcd |

  
**Considerations**:  
\+ Pro's
1. Infrastructure as Code implementations increase repeatability, and minimize human errors
2. Automation drastically reduces deployment time when compared to manual configuration of cloud resources

\- Con's
1. Requires working knowledge of terraform

#### 2. User Provided Infrastructure (UPI)
In this option, we use cloud native automation to deploy into each cloud provider.
  
**Considerations**:  
\+ Pro's  
1. Infrastructure as Code implementations increase repeatability, and minimize human errors
2. Automation drastically reduces deployment time when compared to manual configuration of cloud resources

\- Con's  
1. Requires working knowledge of cloud specific automation languages
  
***

### Justification
N/A

### Implications
N/A

| Derived requirements |
|----------------------|
| N/A                  |

| Related decisions    |
|----------------------|
|  N/A                 |