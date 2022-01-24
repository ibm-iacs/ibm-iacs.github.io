# Why Automate?

<!--- cSpell:ignore personel unmanaged Paks Quickstart qube cntk autoplay allowfullscreen -->

## Automation Overview
The [IBM Garage Methodology](https://www.ibm.com/garage/method/practices/manage/practice_operations_automation/) describes in detail why *Automation* is key in the Cloud world. In simple terms, Cloud computing is costly and rapidly scales beyond the capacity of human administrators. 

While compute, network, storage costs have continued to decrease, the required investment in operations personel has increased. *"The cost of infrastructure, even at scale, is negligible compared to just the average monthly overhead of an employee, regardless of the employee's country of residence."*

To estimate the cost of a solution, you can use a formula with three factors:
(1) The number of people who are required to touch (2) some number of systems (3) some number of times.
If you can reduce any of those three factors, you can decrease the cost of the solution and improve profitability.

**If you're required to log in to a server, you failed**. *Think in terms of operator-to-server ratio, which is another great metric in the cloud industry. The popular brands in cloud, such as Facebook and Google, claim openly to have operator-to-server ratios of something like 1:10,000. At that scale, operators don't have time to log in to individual servers. Instead, they are riding around data centers on scooters, looking for smoking blade servers and replacing them. Everything must be automated, from beginning to end. Servers are cattle, not pets. If the servers misbehave, they are replaced.*

While most organizations are not at the 1:10,000 scale, if you can focus your operators at the higher value tasks by using unattended automation to reduce the number of times people need to touch the systems, you can decrease the cost of the solution and improve profitability. Some might argue that you don't need to worry as much about decreasing the number of systems because an increasing number of systems in the solution is usually a sign of growth and health. Real profitability occurs when you can decrease the number of times people are required to touch the systems. This allows operators to focus on higher-value work and it also leads to an improvement in the operator-to-server ratio.

**Cloud Paks** are no different to any other software system. Any time that an operator has to touch a system to install, configure or update a Cloud Pak, cost is incurred and the operator is taken away from a high value task. This guide focuses on automation of the entire Cloud Pak stack including Infrastructure, OpenShift, Cloud Paks and end user applications to enable customers to run Cloud Paks in Production in a low-touch, high value manner.

## Tools

### Terraform
[Terraform](https://www.terraform.io/) is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services. Terraform codifies cloud APIs into declarative configuration files.

### Ansible
[Ansible](https://www.ansible.com/) is a universal language, unraveling the mystery of how work gets done. Turn tough tasks into repeatable playbooks. Roll out enterprise-wide protocols with the push of a button.

### OpenShift GitOps
[OpenShift GitOps](https://docs.openshift.com/container-platform/4.8/cicd/gitops/understanding-openshift-gitops.html) is an OpenShift add-on which provides Argo CD and other tooling to enable teams to implement GitOps workflows for cluster configuration and application delivery.  Red Hat OpenShift GitOps ensures consistency in applications when you deploy them to different clusters in different environments, such as: development, staging, and production. Red Hat OpenShift GitOps organizes the deployment process around the configuration repositories and makes them the central element.

### OpenShift Pipelines
[OpenShift Pipelines](https://docs.openshift.com/container-platform/4.8/cicd/pipelines/understanding-openshift-pipelines.html) is a cloud-native, continuous integration and continuous delivery (CI/CD) solution based on Kubernetes resources. It uses Tekton building blocks to automate deployments across multiple platforms by abstracting away the underlying implementation details. Tekton introduces a number of standard custom resource definitions (CRDs) for defining CI/CD pipelines that are portable across Kubernetes distributions.