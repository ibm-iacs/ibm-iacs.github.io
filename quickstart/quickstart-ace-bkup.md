# App Connect Enterprise Quickstart

<!--- cSpell:ignore qube cntk autoplay allowfullscreen Quickstart ArgoCD platformnavigator jmeter integrationserver -->

--8<-- "quickstart-overview.md"

The following instructions is from the README.md in this repository: [https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-ace](https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-ace).

---

--8<-- "gitops-cluster-prereq.md"

---

--8<-- "techzone-roks-empty-cluster.md"

---

--8<-- "quickstart-install-required-cli.md"

---

## Steps for setting up the Demo


### Bootstrap cluster

1. Clone the repository and complete the instructions on the README.md in this repo -> [https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-ace](https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-ace)

2. Login to the ArgoCD Console. A few things to note here:
    - It will take approximately 20 - 30 minutes for all the Operators and instances to be deployed.
    - Verify from the ArgoCD UI that all applications are successfully synced
    - You might notice that ArgoCD Applications under "Project: applications" are out of sync.  This is expected and they will be successfully synced once the pipelines are run.

### Validate Cloud Pak for Integration

1. Check status of all the installed operators.
     - From the OCP console, select Operators > Installed Operators for All Projects.  
     - All operators should show a status of "Succeeded".

2. Check status of "platformnavigator" custom resource.
     - From the OCP console, select Operators > Installed Operators for the "tools" project/namespace.
     - Select the "IBM Cloud Pak for Integration Platform Navigator" operator and select the "Platform Navigator" tab.
     - Click on the "integration-navigator" instance and at the bottom of the page, it should show the following: 
        - Status = True
        - Message = Platform Navigator has been provisioned.

3. Check status of the pods in the "ibm-common-services" namespace.  
     - From the OCP console, select Workloads > Pods for the "ibm-common-services" project/namespace. 
     - All pods should have a Status of "Completed" or "Running".

4. Once all the operators and instances are successfully deployed, log in to the Platform Navigator UI and validate the ACE Operator capabilities are enabled. 
     - Platform Navigator URL: From the OCP console, select Operators > Installed Operators in the "tools" project/namespace.  
     - Select the "IBM Cloud Pak for Integration Platform Navigator" operator and click on the "Platform Navigator" tab.
     - Select the "integration-navigator" instance and click the "Platform Navigator UI" link. 
     - Log in with OpenShift credentials.
     - From the capabilities view, clicking the "Create" button should show the ability to create ACE operator provided capabilities (ie. ACE dashboard).  If the operator and/or instances are not deployed correctly, the option will be greyed out.

### Run ACE pipelines
1. From the OCP console, go to Pipelines -> Pipelines and select "ci" from the Project/Namespace dropdown and select "ace-build-bar-promote-dev" pipelines to build custom ACE image and write into the gitops-apps repo for dev environment
     - Click Actions > Start. 
     - All the required "Parameters" are pre-populated. 
     - For "Workspaces", select the following: 
        - shared-workspace: PVC
        - Select the "ace-bar-pvc" PVC
        - Click Start to run the pipeline.
        - NOTE: As no change has been made to the ACE flow application source repo, the "git-ops" step of this pipeline will fail per the logs from this task.  The pipeline will have built an image of the ACE integrationserver with the bar file and pushed it into the internal registry.  The "create-customer-details-rest-is" deployment pods in DEV should be bounced to pick up the newly built image. In order for that to happen, execute the following command in your terminal:

        ```
        oc rollout restart deploy/create-customer-details-rest-is -n dev
        ```

2. In your OCP Console, go back to Pipelines > Pipelines and select "ci" Project/Namespace and select "ace-promote-dev-stage" pipelines to start pipeline to promote ACE IntegrationServer and Configuration resources to STAGING env if tests (ie. postman functional tests) pass.
     - Click Actions > Start. 
     - Set the following "Parameters" 
     - For "Workspaces", select the following: 
        - shared-workspace: PVC
        - Select the "ace-test-pvc" PVC
        - Define the source to "dev" and target to "staging"
        - Click Start to run the pipeline.
     - View the pull request created by the pipeline for promotion to STAGING in this repo, select closed PRs to see an example PR, there will not be a PR created if the image tag on dev is the same in staging already (https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-apps/pulls).
     - NOTE: The "create-customer-details-rest-is" deployment pods should be bounced to pick up the newly built image in STAGING.

     ```
    oc rollout restart deploy/create-customer-details-rest-is -n staging
     ```


3. Back in your OCP console, go back to Pipelines > Pipelines and select "ci" Project/Namespace and select "ace-promote-stage-prod" pipeline to start the pipeline to promote ACE IntegrationServer and Configuration resources to **PROD** env if tests (ie. jmeter performance tests) pass.
     - Click Actions > Start. 
     - Set the following "Parameters" 
     - For "Workspaces", select the following: 
        - shared-workspace: PVC
        - Select the "ace-test-pvc" PVC
        - Click Start to run the pipeline.
     - View the pull request created by the pipeline for promotion to PROD in this repo.
     - Select closed PRs to see an example PR, there will not be a PR created if the image tag on dev is the same in staging already (https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-apps/pulls).
     - NOTE: The "create-customer-details-rest-is" deployment pods should be bounced to pick up the newly built image in PROD.
     ```
    oc rollout restart deploy/create-customer-details-rest-is -n prod
     ```

### ACE Dashboard
You can open the ACE Dashboard running in the dev namespace and see the Integration Server and Integration running in dev.

1. Go to Installed Operators
2. Select dev namespace
3. Select App Connect Operator
4. Select App Connect Dashboard from the tabs
5. Select the ace-dashboard instance
6. Open the Admin UI url
