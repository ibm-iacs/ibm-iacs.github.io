# Cluster Infrastructure Hands-On

<!--- cSpell:ignore unmanaged practioners Pak Paks Quickstart qube cntk autoplay allowfullscreen vmware airgapped -->

## Cluster Infrastructure

Deployment of IBM Cloud Pak requires RedHat OpenShift cluster. 
You can read more about the infrastructure layers and the implementation options for the different cloud platform providers in [Golden Topology](../infrastructure/golden-topology.md). 

In general, you can have one of the following clusters:

- **Managed OpenShift clusters**, on which some operation, monitoring and billing are integrated with the cloud-platform provider. These clusters are build and managed by the cloud-provider's interface:
    - [RedHat OpenShift on IBM Cloud (ROKS)](../infrastructure/roks.md)
    - [RedHat OpenShift on AWS (ROSA)](../infrastructure/rosa.md)
    - [Azure RedHat OpenShift (ARO)](../infrastructure/aro.md)

- **Installer Provisioned Infrastructure (IPI)** clusters which are created using the `openshift-install` command. These IPI clusters can typically be enhanced to accommodate [infrastructure and storage nodes](../infrastructure/infra-gitops.md) using the gitops scaffolding. 
    - [Azure](../infrastructure/azure.md)
    - [AWS](../infrastructure/aws.md)
    - [VMware](../infrastructure/vmware.md)

- **User Provisioned Infrastructure (UPI)** clusters, which are built manually to accommodate an unsupported environment. Although we did not prefer this path, we understand that there are *valid* customer situation that may warrant the UPI implementation. However, with UPI, you are responsible on building the additional structure so that the cluster is production ready.

Several customer may also request that the cluster would not have internet access (i.e [restricted network or airgapped](../infrastructure/restricted-networks.md)). Although most can be fulfilled with IPI airgapped cluster, some specific deployment may need to be UPI. 
