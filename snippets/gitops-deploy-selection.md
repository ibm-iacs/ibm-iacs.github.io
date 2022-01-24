## Select resources to deploy
In this section you will:

- Clone the `multi-tenancy-gitops` repository from your Git Organization to your local machine

- Use one of the recipes to select which components from the [infrastructure](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/1-infra/kustomization.yaml) and [services](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/2-services/kustomization.yaml) layers to deploy.  
    - [ACE recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/ace-recipe.md)
    - [MQ recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/mq-recipe.md)
    - [Process Mining recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/process-mining-recipe.md)

### Instructions 
1. Clone the `multi-tenancy-gitops` repository from your Git Organization to your local machine using a command like the one shown below:

    ```
    git clone https://github.com/xxxxxxx/multi-tenancy-gitops
    ```

1. Review the contents of the `0-bootstrap` folder and note the `single-cluster` folder and the other topologies are in an `others` folder. The contents of the `others` folder is provided for other scenarios and can be deleted.

    ```
    cd multi-tenancy-gitops/0-bootstrap/

    tree -L 2
    .
    ├── bootstrap.yaml
    ├── others
    │   ├── 1-shared-cluster
    │   ├── 2-isolated-cluster
    │   └── 3-multi-cluster
    └── single-cluster
        ├── 1-infra
        ├── 2-services
        ├── 3-apps
        ├── bootstrap.yaml
        └── kustomization.yaml      
    ```

1. Delete the `others` folder from the `0-bootstrap` folder to ensure the `single-cluster` topology is used

    ```
    rm -rf others/

    tree -L 2
    .
    ├── bootstrap.yaml
    └── single-cluster
        ├── 1-infra
        ├── 2-services
        ├── 3-apps
        ├── bootstrap.yaml
        └── kustomization.yaml
    ```

2. Using the `recipe` you selected above, modify the `0-bootstrap/single-cluster/1-infra/kustomization.yaml` file to configure the `Infrastructure` layer by un-commenting the resources to deploy.  

3.  Using the `recipe` you selected above, modify the `0-bootstrap/single-cluster/2-services/kustomization.yaml` and un-comment the resources to deploy. 

4. Note that the recipe may show additional files to edit. If so, make those edits now.

4. Commit and push changes to your git repository
    ```bash
    git add .
    git commit -m "initial bootstrap setup"
    git push origin
    ```

5. If an IBM Cloud Pak is installed, retrieve the Platform Navigator console URL and admin password.
    ```bash
    # Verify the Common Services instance has been deployed successfully
    oc get commonservice common-service -n ibm-common-services -o=jsonpath='{.status.phase}'
    # Expected output = Succeeded

    # Verify the Platform Navigator instance has been deployed successfully
    oc get platformnavigator -n tools -o=jsonpath='{ .items[*].status.conditions[].status }'
    # Expected output = True
    
    # 
    oc get route -n tools integration-navigator-pn -o template --template='https://{{.spec.host}}'
    oc extract -n ibm-common-services secrets/platform-auth-idp-credentials --keys=admin_username,admin_password --to=-
    ```