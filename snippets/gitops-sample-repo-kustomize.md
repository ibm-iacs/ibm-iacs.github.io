## Customizing the GitOps repositories

The cloned GitOps configuration repositories (specifically, `multi-tenancy-gitops`
and `multi-tenancy-gitops-apps`) needs to be customized for your
cluster. The ArgoCD `Application` and `AppProject` resources and `kustomization.yaml`
resources contains environment variables which will need to be updated to your
specific GitHub organization and branch.

### Update multi-tenancy-gitops repository

Have a look at the following `kustomization.yaml` as an example of what will need to
be updated.

```bash
cat 0-bootstrap/single-cluster/1-infra/kustomization.yaml
```

```{ .yaml .no-copy }
resources:
#- argocd/consolelink.yaml
#- argocd/consolenotification.yaml
#- argocd/namespace-ibm-common-services.yaml
#- argocd/namespace-ci.yaml
#- argocd/namespace-dev.yaml
#- argocd/namespace-staging.yaml
#- argocd/namespace-prod.yaml
#- argocd/namespace-cloudpak.yaml
#- argocd/namespace-istio-system.yaml
#- argocd/namespace-openldap.yaml
#- argocd/namespace-sealed-secrets.yaml
#- argocd/namespace-tools.yaml
#- argocd/namespace-instana-agent.yaml
#- argocd/namespace-robot-shop.yaml
#- argocd/namespace-openshift-storage.yaml
#- argocd/namespace-spp.yaml
#- argocd/namespace-spp-velero.yaml
#- argocd/namespace-baas.yaml
#- argocd/serviceaccounts-tools.yaml
#- argocd/storage.yaml
#- argocd/infraconfig.yaml
#- argocd/machinesets.yaml
patches:
- target:
    group: argoproj.io
    kind: Application
    labelSelector: "gitops.tier.layer=infra"
  patch: |-
    - op: add
      path: /spec/source/repoURL
      value: ${GIT_BASEURL}/${GIT_ORG}/${GIT_GITOPS_INFRA}
    - op: add
      path: /spec/source/targetRevision
      value: ${GIT_GITOPS_INFRA_BRANCH}
```

Once we've customized the local repository, we'll push our updates back to our
repository branch on GitHub where it can be accessed by ArgoCD.

  1. *Run the customization script*

    Let's customize the cloned `multi-tenancy-gitops` repository with the relevant values for our
    cluster.

    The sample GitOps repository provides the `set-git-source.sh` script to make
    this task easy.

    Run script to replace the git url and branch to your git organization where
    you created the git repositories:

    ```bash
    GIT_ORG=<Your Organization> GIT_BRANCH=master ./scripts/set-git-source.sh
    ```

    The script will list customizations it will use and all the files that it
    customizes:

    ```{ .text .no-copy }
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-infra.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-services.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-apps.git on branch master
    done replacing variables in kustomization.yaml files
    git commit and push changes now
    ```

    You can examine your local clone of the GitOps repository to verify these
    customizations. You should see lots of amended ArgoCD YAMLs that point to your
    GitOps repository on GitHub.

    This script has also made some customizations that we'll use much later in the
    tutorial; we'll refer to those at the relevant time.

  2. *Add the changes to a git index, ready to push to GitHub*

    We've now customized our local clone of the `multi-tenancy-gitops` repository. Let's
    commit these changes and make the customized repository available to the
    cluster via GitHub.

    Add all changes in the current folder to a git index:

    ```bash
    git add .
    ```

  3. *Commit the changes to git*

    Use the following command to create a commit record:

    ```bash
    git commit -s -m "GitOps customizations for organization and cluster"
    ```

    See the Git commit message for the customized files:

    ```{ .bash .no-copy }
    [master a900c39] GitOps customizations for organization and cluster
    199 files changed, 290 insertions(+), 290 deletions(-)
    ```

  4. *Set your GitOps repository branch*

    For this guide, we will be using the `master` branch for the `multi-tenancy-gitops` repository.

    ```bash
    export GIT_BRANCH=master
    ```

    You can verify your `$GIT_BRANCH` as follows:

    ```bash
    echo $GIT_BRANCH
    ```

  5. *Push changes to GitHub*

    Push this commit back to the  branch on GitHub:

    ```bash
    git push origin $GIT_BRANCH
    ```

    The changes have now been pushed to your GitOps repository:

    ```{ .text .no-copy }
    Enumerating objects: 90, done.
    Counting objects: 100% (90/90), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (47/47), done.
    Writing objects: 100% (48/48), 4.70 KiB | 1.57 MiB/s, done.
    Total 48 (delta 32), reused 0 (delta 0)
    remote: Resolving deltas: 100% (32/32), completed with 23 local objects.
    To https://github.com/prod-ref-guide/multi-tenancy-gitops.git
       d95eca5..a900c39  master -> master
    ```


### Update multi-tenancy-gitops-apps repository

Similar to the previous section, there are corresponding ArgoCD `Applications` resources that
will need to be updated in the `multi-tenancy-gitops-apps` repository.  

Once we've customized the local repository, we'll push our updates back to our
repository branch on GitHub where it can be accessed by ArgoCD.

  1. *Locate the multi-tenancy-gitops-apps repository*

    If necessary, change to the root of your GitOps repository, which is typically
    `$HOME/git`.

    Issue the following command to change to your GitOps repository:

    ```bash
    cd $HOME/git
    cd multi-tenancy-gitops-apps
    ```  

  2. *Run the customization script*

    Let's customize the cloned `multi-tenancy-gitops-apps` repository with the relevant values for our
    cluster.

    The sample GitOps repository provides the `set-git-source.sh` script to make
    this task easy.

    Run script to replace the git url and branch to your git organization where
    you created the git repositories:

    ```bash
    GIT_ORG=<Your Organization> GIT_BRANCH=master ./scripts/set-git-source.sh
    ```

    The script will list customizations it will use and all the files that it
    customizes:

    ```{ .text .no-copy }
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-infra.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-services.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-apps.git on branch master
    done replacing variables in kustomization.yaml files
    git commit and push changes now
    ```

    You can examine your local clone of the GitOps repository to verify these
    customizations. You should see lots of amended ArgoCD YAMLs that point to your
    GitOps repository on GitHub.

  3. *Add the changes to a git index, ready to push to GitHub*

    We've now customized our local clone of the `multi-tenancy-gitops-apps` repository. Let's
    commit these changes and make the customized repository available to the
    cluster via GitHub.

    Add all changes in the current folder to a git index:

    ```bash
    git add .
    ```

  4. *Commit the changes to git*

    Use the following command to create a commit record:

    ```bash
    git commit -s -m "GitOps customizations for organization and cluster"
    ```

    See the Git commit message for the customized files:

    ```{ .bash .no-copy }
    [master a900c39] GitOps customizations for organization and cluster
    21 files changed, 42 insertions(+), 42 deletions(-)
    ```

  5. *Set your repository branch*

    For this guide, we will be using the `master` branch for the `multi-tenancy-gitops-apps` repository.

    ```bash
    export GIT_BRANCH=master
    ```

    You can verify your `$GIT_BRANCH` as follows:

    ```bash
    echo $GIT_BRANCH
    ```

  6. *Push changes to GitHub*

    Push this commit back to the  branch on GitHub:

    ```bash
    git push origin $GIT_BRANCH
    ```

    The changes have now been pushed to your GitOps repository:

    ```{ .text .no-copy }
    Enumerating objects: 90, done.
    Counting objects: 100% (90/90), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (47/47), done.
    Writing objects: 100% (42/42), 4.70 KiB | 1.57 MiB/s, done.
    Total 42 (delta 42), reused 0 (delta 0)
    remote: Resolving deltas: 100% (32/32), completed with 21 local objects.
    To https://github.com/prod-ref-guide/multi-tenancy-gitops-apps.git
       d95eca5..a900c39  master -> master
    ```
