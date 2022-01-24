# Spectrum Protect Plus

<!--- cSpell:ignore SPP OADP automations hyperautomation unmanaged practioners Pak Paks Quickstart qube cntk autoplay allowfullscreen -->

## Unified data protection for hybrid cloud environments

IBM Spectrum® Protect Plus (SPP) is a data protection and availability solution for virtual environments and database applications. As a data resiliency solution it provides recovery, replication, retention, and reuse of data. 

The graphic below shows SPP deployed in two active locations. The IBM Spectrum Protect Plus server is deployed in only one of the sites. vSnap servers (with their corresponding disks) are deployed in each site to localize data movement in the context of the protected vSphere resources. Bi-directional replication is configured to take place between the vSnap servers at the two sites.

![SPP Overview](images/spp_overview.png)

## Information about SPP

IBM Spectrum Protect Plus is used to restore the snapshots from vSnap to the original or alternate hypervisor. In this production scenario SPP is used to backup and restore data from IBM® Cloud Pak for Data environments. SPP can be installed as:

* A virtual appliance
* A set of OpenShift containers

SPP uses OADP (OpenShift APIs for Data Protection) which is an operator that Red Hat has created to create backup and restore APIs in the OpenShift cluster.

Cloud Paks can backup the entire namespace whereas SPP also helps in backing up common services and cluster related resources. These resources can then be used to restore everything in a new environment.
  
## Links

- [IBM Spectrum Protect Plus](https://www.ibm.com/products/ibm-spectrum-protect-plus)
- [IBM Spectrum Protect Plus Knowledge Center](https://www.ibm.com/docs/en/spp/10.1.8?topic=product-overview)
- [IBM Spectrum Protect Plus Blueprints](https://www.ibm.com/support/pages/node/1119489)