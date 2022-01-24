# Continuous updates

<!--- cSpell:ignore  pipelinerun mqsc msqc runmqsc dockerconfigjson queuemanager QMID podman ibmgaragecloud cntk cnkt eventid gitrevision gitrepositoryurl odowdaibm MQSC replicaset eventlistener triggerbinding triggertemplate mqcicd qmgr Artifactory configmaps OIDC CHLAUTH templating sealedsecret -->

## Overview

In the [previous topic](../qmgr-pipeline/topic3.md) of this chapter, we ran the queue manager
pipeline to build a queue manager ready for deployment. We then deployed it
using ArgoCD. Finally, we tested it using the MQ web console and MQ command
line.

In this topic, we're going to make this integration and deployment processes
fully automatic. A change to the queue manager source repository will
automatically result in a pipeline run for it. If the pipeline is successful, a
new Helm chart will be built and automatically deployed by ArgoCD into the `dev`
namespace. This deployment will update the previously active instance of `QM1`.
To recap, the whole process will be automatic from end-to-end.

Look at the following diagram:

![mqcicd7](./images/mq-cicd7.png)

We've already seen most of the highlighted components. Notice that we've added
a `webhook` for the source repository. This web hook will be triggered every
time the `QM1` repository changes and the notification will cause the queue
manager pipeline to run automatically.

As before, a successful pipeline run will result in a new image being stored in
the Image registry, and new Helm charts in Artifactory and the GitOps
repository. The already active ArgoCD application will apply the GiOps Helm
chart to the cluster resulting in an updated instance of `QM1`.

In this topic, we're going to:

* Set up a webhook for queue manager source repository.
* Make changes to the queue manager source code and push the updates to the
  source repository.
* Confirm the new pipeline run and ArgoCD deployment.
* Verify the change made to `QM1`.

By the end of this topic we'll have a fully functioning CICD set up in place to
deploy the latest changes made to QueueManager to the cluster in an automated
fashion.

---

## Pre-requisites

Before attempting this topic, you should have successfully completed the
[previous topic](../qmgr-pipeline/topic3.md).

---

## Set up the webhook and its processing

A webhook is useful because it allows a system like GitHub to inform another
component -- such as our queue manager pipeline -- that a particular repository
owned by a user or organization has changed. Importantly, a webhook provides a
loosely coupled interaction; GitHub and our pipeline are unaware of each other
-- their interaction is mediated by the webhook and some other components that
we'll now set up.

In this topic section we're going to set up the components we need for our
webhook:

![mqcicd8](./images/mq-cicd8.png)

A GitHub event containing information about the repository change is generated
by our webhook and sent to an **event listener**. In Kubernetes, this event
listener comprises a normal `route` and `service` front-ending a Tekton
`eventlistener` custom resource. The event listener is driven by the webhook,
and runs the pipeline using a Tekton **trigger**.  A trigger has two components,
a `triggertemplate` that specifies the pipeline to run, and a `triggerbinding`
that allows the trigger event data to be presented to the pipeline run in the
order it expects.

A webhook design looks a little more complex than directly linking GitHub to our
pipeline, but as we'll see, this design is much more flexible. This [short
article](https://cloud.redhat.com/blog/guide-to-openshift-pipelines-part-6-triggering-pipeline-execution-from-github)
provides a nice overview on how to use webhooks with OpenShift.

Your GitOps apps repository contains a sample event listener. Let's customize
this sample with the relevant values for our cluster. We'll then deploy the
event listener resources and trigger resources to the cluster using GitOps.

1. *Select the GitOps terminal window*

      **Return to the terminal window** you're using for the `multi-tenancy-gitops-apps`
      GitOps repository.

      We're going to make use of a script in the sample GitOps apps repository that makes
      use of the `$GIT_ORG` and `$GIT_BRANCH_QM1` environment variables you've seen
      earlier. Previously we used them in the `mq-infra` terminal window; now we're
      going to use this in the `multi-tenancy-gitops-apps` terminal window.

      Again, **make sure** in you're in the  `multi-tenancy-gitops-apps` terminal window.

      Issue the following command to ensure you're in the correct starting folder:

      ```bash
      cd $HOME/git/multi-tenancy-gitops-apps
      ```

 2. *Set up the* `$GIT_ORG` and `$GIT_BRANCH_QM1` *environment variables*

      Let's set up the environment variables that are used by the customization
      script.

      Replace `<git-org>` in the following command with your GitHub user name:

      ```bash
      export GIT_ORG=<git-org>
      ```

      Then create the `$GIT_BRANCH_QM1` environment variable:

      ```bash
      export GIT_BRANCH_QM1=qm1-$GIT_ORG
      ```

      Verify that your `$GIT_USER` and `$GIT_BRANCH environment` variables are set:

      ```bash
      echo $GIT_ORG
      echo $GIT_BRANCH_QM1
      ```

      For example:

      ``` { .text .no-copy }
      (base) anthonyodowd/git/multi-tenancy-gitops echo GIT_ORG
      prod-ref-guide
      (base) anthonyodowd/git/multi-tenancy-gitops echo GIT_BRANCH_QM1
      qm1-prod-ref-guide
      ```

3. *The sample event listener*

      The GitOps repository contains a template for the event listener.

      Issue the following command to view the template of the event listener YAML:

      ```bash
      cat mq/environments/ci/eventlisteners/cntk-event-listener.yaml_template
      ```

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        labels:
          app: cnkt-event-listener
        name: cnkt-event-listener
      spec:
        serviceAccountName: pipeline
        triggers:
        - name: mq-infra-dev
            interceptors:
              - ref:
                  name: "cel"
                params:
                - name: "filter"
                  value: "header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/${GIT_BRANCH_QM1}' && body.repository.full_name == '${GIT_ORG}/mq-infra'"
            bindings:
              - ref: cnkt-trigger-binding
            template:
              ref: mq-infra-dev
        # - name: mq-spring-app-dev
        #    interceptors:
        #      - ref:
        #          name: "cel"
        #        params:
        #        - name: "filter"
        #          value: "header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/${GIT_BRANCH_SPRING}' && body.repository.full_name == '${GIT_ORG}/mq-spring-app'"
        #    bindings:
        #        - ref: cnkt-trigger-binding
        #    template:
        #      ref: mq-spring-app-dev
      ```

      If you are using OpenShift 4.7, replace the above template with the following:

      ```yaml
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        labels:
          app: cnkt-event-listener
        name: cnkt-event-listener
      spec:
        serviceAccountName: pipeline
        triggers:
        - bindings:
          - kind: TriggerBinding
            ref: cnkt-trigger-binding
          interceptors:
          - cel:
              filter: header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/${GIT_BRANCH_QM1}' && body.repository.full_name == '${GIT_ORG}/mq-infra'
          name: mq-infra-dev
          template:
            ref: mq-infra-dev
        # - bindings:
        #   - kind: TriggerBinding
        #     ref: cnkt-trigger-binding
        #   interceptors:
        #   - cel:
        #       filter: header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/${GIT_BRANCH_SPRING}' && body.repository.full_name == '${GIT_ORG}/mq-spring-app'
        #   name: mq-spring-app-dev
        #   template:
        #     ref: mq-spring-app-dev
      ```

4. *Customize the event listeners*

      Create the YAML for the event listener as follows.

      Issue the following command:

      ```bash
      cd mq/environments/ci/eventlisteners/
      sh ./cntk-event-listener.sh
      cd ../../../../
      ```

      Once, this script is run successfully, you should see a new file named `cntk-event-listener.yaml`.

4. *Exploring the event listener customization*

      Let's examine a customized event listener in your local clone. We'll see how
      it maps to our diagram above, and has been customized for our source
      repository.

      Issue the following command:

      ```bash
      cat mq/environments/ci/eventlisteners/cntk-event-listener.yaml
      ```

      to show the customized event listener:

      ``` { .yaml .no-copy}
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        labels:
          app: cnkt-event-listener
        name: cnkt-event-listener
      spec:
        serviceAccountName: pipeline
        triggers:
        - name: mq-infra-dev
            interceptors:
              - ref:
                  name: "cel"
                params:
                - name: "filter"
                  value: "header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/qm1-prod-ref-guide-test' && body.repository.full_name == 'prod-ref-guide-test/mq-infra'"
            bindings:
              - ref: cnkt-trigger-binding
            template:
              ref: mq-infra-dev
        # - name: mq-spring-app-dev
        #    interceptors:
        #      - ref:
        #          name: "cel"
        #        params:
        #        - name: "filter"
        #          value: "header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/master' && body.repository.full_name == 'prod-ref-guide-test/mq-spring-app'"
        #    bindings:
        #        - ref: cnkt-trigger-binding
        #    template:
        #      ref: mq-spring-app-dev
      ```

      If using OpenShift 4.7, it will be as follows:

      ```{ .yaml .no-copy}
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        labels:
          app: cnkt-event-listener
        name: cnkt-event-listener
      spec:
        serviceAccountName: pipeline
        triggers:
        - bindings:
          - kind: TriggerBinding
            ref: cnkt-trigger-binding
          interceptors:
          - cel:
              filter: header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/${GIT_BRANCH_QM1}' && body.repository.full_name == 'prod-ref-guide-test/mq-infra'
          name: mq-infra-dev
          template:
            ref: mq-infra-dev
        # - bindings:
        #   - kind: TriggerBinding
        #     ref: cnkt-trigger-binding
        #   interceptors:
        #   - cel:
        #       filter: header.match('X-GitHub-Event', 'push') && body.ref == 'refs/heads/qm1-prod-ref-guide-test' && body.repository.full_name == '${GIT_ORG}/mq-spring-app'
        #   name: mq-spring-app-dev
        #   template:
        #     ref: mq-spring-app-dev
      ```

      An event listener can produce one or more triggers as specified by the list
      `spec.triggers:`. Our event listener produces a trigger called
      `mq-infra-dev`. This trigger comprises the template `mq-infra-dev` and template
      binding `cntk-trigger-binding`.

      It also has an interceptor; the trigger will only be called if `filter:`
      evaluates to `true`. Even though we haven't explored interceptors yet,
      `filter value of the interceptor:` is clearly filtering GitHub `push` events for the
      `prod-ref-guide/mq-infra` repository, i.e. the source repository for `QM1`. It was
      the customization script that made this customization.

      You can read more about event listeners, triggers and interceptors
      [here](https://tekton.dev/docs/triggers/).

5. *Push GitOps changes to GitHub*

      Let’s make these GitOps changes and push them.

      Add all changes in the current folder to a git index, commit them, and push them to GitHub:

      ```bash
      git add .
      git commit -s -m "Customizing the mq-infra event trigger"
      git push origin $GIT_BRANCH
      ```

      The changes have now been pushed to your GitOps repository:

      ``` { .text .no-copy }
      Enumerating objects: 12, done.
      Counting objects: 100% (12/12), done.
      Delta compression using up to 8 threads
      Compressing objects: 100% (7/7), done.
      Writing objects: 100% (7/7), 863 bytes | 863.00 KiB/s, done.
      Total 7 (delta 5), reused 0 (delta 0)
      remote: Resolving deltas: 100% (5/5), completed with 5 local objects.
      To https://github.com/prod-ref-guide/multi-tenancy-gitops-apps.git
         edf584c..9d1e713  master -> master
      ```

6. *Activate the webhook components*

      Now that we've customized the event listener for our source repository, let's
      activate all the components necessary to process our webhook: the route, event
      listener, trigger template and trigger binding.

      Open the `kustomization.yaml` that is under `mq/environments/ci/`. This
      contains all the resources that belong to the `ci` namespace.

      ```bash
      cat mq/environments/ci/kustomization.yaml
      ```

      ``` { .yaml .no-copy }
      resources:
      #- certificates/ci-mq-client-certificate.yaml
      #- certificates/ci-mq-server-certificate.yaml
      - configmaps/gitops-repo-configmap.yaml
      #- eventlisteners/cntk-event-listener.yaml
      #- triggerbindings/cntk-binding.yaml
      #- triggertemplates/mq-infra-dev.yaml
      #- triggertemplates/mq-spring-app-dev.yaml
      - pipelines/ibm-test-pipeline-for-dev.yaml
      - pipelines/ibm-test-pipeline-for-stage.yaml
      #- pipelines/java-maven-dev-pipeline.yaml
      - pipelines/mq-pipeline-dev.yaml
      - pipelines/mq-spring-app-dev-pipeline.yaml
      - roles/custom-pipeline-sa-clusterrole.yaml
      - roles/custom-pipeline-sa-role.yaml
      - roles/custom-ci-pipeline-sa-rolebinding.yaml
      - roles/custom-dev-pipeline-sa-rolebinding.yaml
      - roles/custom-staging-pipeline-sa-rolebinding.yaml
      - roles/custom-prod-pipeline-sa-rolebinding.yaml
      #- routes/cntk-route.yaml
      - secrets/artifactory-access-secret.yaml
      - secrets/git-credentials-secret.yaml
      - secrets/ibm-entitled-registry-credentials-secret.yaml
      #- secrets/mq-client-jks-password-secret.yaml
      - tasks/10-gitops.yaml
      - tasks/12-functional-tests.yaml
      - tasks/13-jmeter-performance-test.yaml
      - tasks/4-smoke-tests-mq.yaml
      - tasks/4-smoke-tests.yaml
      - tasks/ibm-build-tag-push-v2-6-13.yaml
      - tasks/ibm-helm-release-v2-6-13.yaml
      - tasks/ibm-img-release-v2-6-13.yaml
      - tasks/ibm-img-scan-v2-6-13.yaml
      - tasks/ibm-java-maven-test-v2-6-13.yaml
      - tasks/ibm-setup-v2-6-13.yaml
      - tasks/ibm-tag-release-v2-6-13.yaml
      ```

      Open this file in your editor and uncomment the below resources.

      ``` { .yaml .no-copy }
      - eventlisteners/cntk-event-listener.yaml
      - triggerbindings/cntk-binding.yaml
      - triggertemplates/mq-infra-dev.yaml
      - routes/cntk-route.yaml
      ```

      You will have the following resources un-commented:

      ``` { .yaml .no-copy }
      resources:
      #- certificates/ci-mq-client-certificate.yaml
      #- certificates/ci-mq-server-certificate.yaml
      - configmaps/gitops-repo-configmap.yaml
      - eventlisteners/cntk-event-listener.yaml
      - triggerbindings/cntk-binding.yaml
      - triggertemplates/mq-infra-dev.yaml
      #- triggertemplates/mq-spring-app-dev.yaml
      - pipelines/ibm-test-pipeline-for-dev.yaml
      - pipelines/ibm-test-pipeline-for-stage.yaml
      #- pipelines/java-maven-dev-pipeline.yaml
      - pipelines/mq-pipeline-dev.yaml
      - pipelines/mq-spring-app-dev-pipeline.yaml
      - roles/custom-pipeline-sa-clusterrole.yaml
      - roles/custom-pipeline-sa-role.yaml
      - roles/custom-ci-pipeline-sa-rolebinding.yaml
      - roles/custom-dev-pipeline-sa-rolebinding.yaml
      - roles/custom-staging-pipeline-sa-rolebinding.yaml
      - roles/custom-prod-pipeline-sa-rolebinding.yaml
      - routes/cntk-route.yaml
      - secrets/artifactory-access-secret.yaml
      - secrets/git-credentials-secret.yaml
      - secrets/ibm-entitled-registry-credentials-secret.yaml
      #- secrets/mq-client-jks-password-secret.yaml
      - tasks/10-gitops.yaml
      - tasks/12-functional-tests.yaml
      - tasks/13-jmeter-performance-test.yaml
      - tasks/4-smoke-tests-mq.yaml
      - tasks/4-smoke-tests.yaml
      - tasks/ibm-build-tag-push-v2-6-13.yaml
      - tasks/ibm-helm-release-v2-6-13.yaml
      - tasks/ibm-img-release-v2-6-13.yaml
      - tasks/ibm-img-scan-v2-6-13.yaml
      - tasks/ibm-java-maven-test-v2-6-13.yaml
      - tasks/ibm-setup-v2-6-13.yaml
      - tasks/ibm-tag-release-v2-6-13.yaml
      ```

7. *Update the GitOps repository*

      Let’s commit these changes to make the event listener and trigger resources
      active in the cluster.

      Issue the following command:

      ```bash
      git add mq/environments/ci/kustomization.yaml
      git commit -s -m "Add qmgr pipeline webhook"
      git push origin $GIT_BRANCH
      ```

      which shows that the changes are pushed to GitHub:

      ``` { .text .no-copy }
      Enumerating objects: 11, done.
      Counting objects: 100% (11/11), done.
      Delta compression using up to 8 threads
      Compressing objects: 100% (6/6), done.
      Writing objects: 100% (6/6), 522 bytes | 522.00 KiB/s, done.
      Total 6 (delta 5), reused 0 (delta 0)
      remote: Resolving deltas: 100% (5/5), completed with 5 local objects.
      To https://github.com/hp-gitops-mq/multi-tenancy-gitops-apps.git
         e28693e..25f7430  master -> master
      ```

      This change to the GitOps repository can now be used by ArgoCD.

8. *The `apps-mq-rest-ci-1` argocd application*

      Let's examine the ArgoCD application that manage the **applications in
      ci namespace**.

      In the ArgoCD UI **Applications** view, click on the icon for the
      `apps-mq-rest-ci-1` application:

      ![diagram20](./images/diagram20.png){: style="max-height:700px"}

      You will see all the necessary configurations. Now, you will see
      trigger template, trigger binding and event listener we just created
      under this application.

9. *Exploring the* `cntk-event-listener` *Kubernetes resource*

      Let's spend a few moments exploring the event listener resources.

      If you observe the above picture carefully, below are few things to note:

      * See how the `eventlistener` custom resource deployed by the ArgoCD
        application has created a service named `el-cntk-event-listener`. The
        event listener route points to this service.
      * The event listener runs in a pod, managed by a replicaset, defined by a
        deployment. It's this pod that will handle the webhook request, and call
        the trigger if the interceptor `filter` evaluates to `true`.


10. *Exploring the resources using the* `oc` *command*

      We can use also use the `oc` command to explore our webhook resources:

      To view the route that is called by the webhook, issue the following command:

      ```bash
      oc get route -n ci
      ```

      To view the event listener service that is accessed via the route, issue the
      following command:

      ```bash
      oc get eventlistener -n ci
      ```

      To view the trigger template that is used by the event service to call the
      `mq-infra-dev` pipeline, issue the following command:

      ```bash
      oc get triggertemplate -n ci
      ```

      To view the trigger binding that is used to marshal the input for the
      pipeline run, issue the following command:

      ```bash
      oc get triggerbinding -n ci
      ```

      You can explore these resources in more detail by adding the `-o yaml` option.

      For now, it's enough to understand understand that `webhook`->`route`->`event
      listener`->`trigger template`->`pipeline`.

      Let's now configure a GitHub webhook to use these components.

11. *The GitHub webhook UI*

      Let's now use the GitHub UI, to configure a webhook that creates an event and sends it to the route
      whenever the queue manager source repository changes.

      We'll configure the webhook using the GitHub UI.

      Issue the following command to determine the URL for the UI:

      ```bash
      echo https://github.com/${GIT_ORG}/mq-infra/settings/hooks/new
      ```

      for example:

      ``` { .text .no-copy }
      https://github.com/prod-ref-guide/mq-infra/settings/hooks/new
      ```

      Copy the URL into your browser to launch the GitHub webpage to configure a new
      webhook for your `mq-infra` repository:

      ![diagram92](./images/diagram92.png)

      We need to complete these details to configure our webhook.

12. *Configure the webhook*

      To configure a webhook, we need to identify the URL it will call when it
      generates an event, the format of the event, and for which GitHub events we'll
      generate an event.

      Issue the following command to determine the URL of the event listener route:

      ```bash
      echo http://$(oc get route el-cnkt-event-listener -n ci -o jsonpath='{.spec.host}')
      ```

      for example:

      ``` { .text .no-copy }
      http://el-cnkt-event-listener-ci.xxxxx.containers.appdomain.cloud
      ```

      Here's a sample webhook configuration:

      ![diagram93](./images/diagram93.png)

      Configure the following arguments:

      * Set the **Payload URL** using the event listener route address.
      * Set **Content type** to `application/json`.
      * Select **Let me select individual event**
        * Select **Pull requests** and **Pushes** from the list of available
          events.

      ![diagram94](./images/diagram94.png)

      Click on `Add webhook` to create the webhook.

13. *A new webhook has been added*

      In the GitHub UI, you can see that a new webhook has been added:

      ![diagram95](./images/diagram95.png)

      Notice the webhook's name and that it's generating and event whenever a
      `pull-request` or `push` is issued against this repository.

      Let's now make a change to the `QM1` source repository and watch the webhook
      at work.

---

## Making a change to the queue manager `QM1`

Now that we've configured our webhook, together with the Kubernetes resources to
process an event generated by it, let's put it to work.

In this section, we'll add a new queue to `QM1`using GitOps. To do this, we'll
update the `QM1` source repository -- adding a the new queue definition to the
queue manager MQSC configuration -- and push this change to GitHub. This will
trigger a new pipeline run that will build and test our change. If successful,
new versions of the queue manager image version and Helm chart will be produced.
The GitOps Helm chart will be automatically deployed to the cluster by our
ArgoCD application, resulting in an updated deployment for `QM1` that contains
the new queue definition.

1. *Select the* `QM1` *source repository terminal window*

      We're going to make changes to the `QM1` source repository, so let's make sure
      we're in the correct terminal window and folder.

      **Return to the terminal window** you're using for the `mq-infra` source
      repository. (*Rather than the terminal window you're using for the*
      `multi-tenancy-gitops-apps` *GitOps repository.*)

      Issue the following command to ensure you're in the correct folder.

      ```bash
      cd $HOME/git/mq-infra
      ```

      Issue the following command to ensure that the `$GIT_ORG` environment
      variable is set:

      ```bash
      echo $GIT_ORG
      ```

      which will show your configured value, for example:

      ``` { .text .no-copy }
      prod-ref-guide
      ```

2. *Set up environment variable for new Git branch*

      The sample repository has a single `master` branch. We're going to create a
      new branch based on `master` for our changes.

      Set the `$GIT_BRANCH_QM1` using your `$GIT_ORG` environment variable:

      ```bash
      export GIT_BRANCH_QM1=qm1-$GIT_ORG
      ```

      then create a new branch using its value:

      ```bash
      git checkout -b $GIT_BRANCH_QM1
      ```

      you'll see something like:

      ``` { .bash .no-copy }
      Switched to a new branch 'qm1-prod-ref-guide'
      ```

      Notice how we've created a new branch `$GIT_BRANCH_QM1` **based** on the
      `master` branch. All changes will be made in `$GIT_BRANCH_QM1`; whereas
      `master` will remain unchanged.

      Also note that we use `$GIT_BRANCH_QM1` to name the `mq-infra` queue manager
      branch in contrast to `$GIT_BRANCH` for the `multi-tenancy-gitops-apps` GitOps
      branch. This helps stop us accidentally promoting a change if we use the wrong
      terminal window in the tutorial.

3. *Verify you're on the new branch*

      We're doing our changes in the `$GIT_BRANCH_QM1` branch.

      Confirm you're on the new branch:

      ```bash
      git branch
      ```

      which will list all the branches for your local clone of the `mq-infra`
      repository:

      ```{ .text .no-copy }
      master
      * qm1-prod-ref-guide
      ```

      Note how `*` confirms your branch.

4. *Recap where* `QM1` *MQSC definitions are stored in the source repository*

      The `QM1` configuration is completely held within the `mq-infra` source
      repository. Let's recall this structure:

      Issue the following command:

      ```bash
      tree
      ```

      to view the folder structure:

      ``` { .text .no-copy }
      .
      ├── Dockerfile
      ├── README.md
      └── chart
            └── base
               ├── Chart.yaml
               ├── config
               │   └── config.mqsc
               ├── security
               │   └── config.mqsc
               ├── templates
               │   ├── NOTES.txt
               │   ├── _helpers.tpl
               │   ├── configmap.yaml
               │   └── qm-template.yaml
               └── values.yaml
      ```

      You may recall that the that the `values.yaml` file has `security: false` set
      by default. This value is used by the `configmap.yaml` template to generate a
      configmap  using `chart/bases/config/config.msqc` rather than
      `chart/bases/security/config.msqc`.  It's therefore this file we need to update
      to add a new queue.

5. *Update* `config.mqsc` *MQSC file*

      Let's quickly review the current MQSC file, before we add our new queue.

      Issue the following command:

      ```bash
      cat chart/base/config/config.mqsc
      ```

      to view the MQSC definitions:

      ```{ .text .no-copy }
      DEFINE QLOCAL(IBM.DEMO.Q) BOQNAME(IBM.DEMO.Q.BOQ) BOTHRESH(3) REPLACE
      DEFINE QLOCAL(IBM.DEMO.Q.BOQ) REPLACE
      * Use a different dead letter queue, for undeliverable messages
      DEFINE QLOCAL('DEV.DEAD.LETTER.QUEUE') REPLACE
      ALTER QMGR DEADQ('DEV.DEAD.LETTER.QUEUE')
      DEFINE CHANNEL('IBM.APP.SVRCONN') CHLTYPE(SVRCONN)
      ALTER QMGR CHLAUTH (DISABLED)
      REFRESH SECURITY TYPE(CONNAUTH)
      ```

      Using your favorite editor, add a new queue definition to
      `chart/base/config/config.mqsc`:

      ```text hl_lines="1"
      DEFINE QLOCAL(IBM.DEMO.Q.NEW) BOQNAME(IBM.DEMO.Q.BOQ) BOTHRESH(3) REPLACE
      DEFINE QLOCAL(IBM.DEMO.Q) BOQNAME(IBM.DEMO.Q.BOQ) BOTHRESH(3) REPLACE
      DEFINE QLOCAL(IBM.DEMO.Q.BOQ) REPLACE
      * Use a different dead letter queue, for undeliverable messages
      DEFINE QLOCAL('DEV.DEAD.LETTER.QUEUE') REPLACE
      ALTER QMGR DEADQ('DEV.DEAD.LETTER.QUEUE')
      DEFINE CHANNEL('IBM.APP.SVRCONN') CHLTYPE(SVRCONN)
      ALTER QMGR CHLAUTH (DISABLED)
      REFRESH SECURITY TYPE(CONNAUTH)
      ```

      Notice how we've added the `IBM.DEMO.Q.NEW` queue to the list of queues for
      this queue manager.

6. *Commit change to GitHub*

      Let's now commit these changes to GitHub.

      ```bash
      git add chart/base/config/config.mqsc
      git commit -s -m "Updated MQSC definition with queue IBM.DEMO.Q.NEW"
      git push origin $GIT_BRANCH_QM1
      ```

      We see that the changes are pushed to GitHub:

      ```{ .text .no-copy }
      Enumerating objects: 11, done.
      Counting objects: 100% (11/11), done.
      Delta compression using up to 8 threads
      Compressing objects: 100% (4/4), done.
      Writing objects: 100% (6/6), 508 bytes | 508.00 KiB/s, done.
      Total 6 (delta 3), reused 0 (delta 0)
      remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
      To https://github.com/prod-ref-guide/mq-infra
         5245247..cf4dd8d  qm1-prod-ref-guide -> qm1-prod-ref-guide
      ```

7. *GitHub event generated*

      This push to GitHub should generate a GitHub webhook event for the `mq-infra`
      repository.  We can look at the most recent webhook events.

      Click on the newly created webhook `http://el-cnkt-event-listener...` and
      select the **Recent Deliveries** tab.

      This is one of our sample Deliveries:

      ![diagram100](./images/diagram100.png)

      Notice:

      * The date and time when the webhook event was generated in the format
        `yyyy-mm-dd hh:mm:ss`
      * The **Request URL** header identifies the event.
        listener `http:://el-cnkt-event-listener...`
      * **"ref": "refs/head/qm1-odowdaibm"** identifies the branch that
        was pushed.
      * **"repository.full_name": "odowdaibm-mq-guide/mq-infra"** identifies the
        repository that was changed.

      The **Recent Deliveries** tab can be very helpful to diagnose any issues you
      might have webhook event generation.

      Let's now see how this webhook event is handled by the event listener in the
      cluster.

8. *Event handled in the cluster*

      Let's have a look at how the GitHub event is processed by the event listener
      in the cluster. We can look at the event listener logs to see how the event is
      received and processed using the `oc logs` command.

      Issue the following command:

      ```bash
      oc logs deploy/el-cnkt-event-listener -n ci
      ```

      to show the log messages issued by the `el-cnkt-event-listener`:

      ``` { .json .no-copy }
      {"level":"info","ts":"2021-08-05T14:31:05.484Z","logger":"eventlistener","caller":"sink/sink.go:236","msg":"ResolvedParams : [{Name:gitrevision Value:840ec76946c891ace8ebf591592077f259f4e5c1} {Name:gitrepositoryurl Value:https://github.com/odowdaibm-mq-guide/mq-infra}]","knative.dev/controller":"eventlistener","/triggers-eventid":"2dd90e6b-b97e-4a2e-8aa1-0de0704af7ba","/trigger":"mq-infra-dev"}
      {"level":"info","ts":"2021-08-05T14:31:05.490Z","logger":"eventlistener","caller":"resources/create.go:95","msg":"Generating resource: kind: &APIResource{Name:pipelineruns,Namespaced:true,Kind:PipelineRun,Verbs:[delete deletecollection get list patch create update watch],ShortNames:[pr prs],SingularName:pipelinerun,Categories:[tekton tekton-pipelines],Group:tekton.dev,Version:v1beta1,StorageVersionHash:RcAKAgPYYoo=,}, name: mq-infra-dev-","knative.dev/controller":"eventlistener"}
      {"level":"info","ts":"2021-08-05T14:31:05.490Z","logger":"eventlistener","caller":"resources/create.go:103","msg":"For event ID \"2dd90ez6b-b97e-4a2e-8aa1-0de0704af7babeta1, Resource=pipelineruns","knative.dev/controller":"eventlistener"}
      ```

      (You can scroll right in the above output.)

      Notice:

      * Multiple log entries have been issued for the same input event. Notice
        how the timestamps `"ts":` are the same.
      * The first entry identifies the data generated by the trigger template
        that will be passed to the pipeline:
        * See how `gitrevision` value matches that in the webhook `"after"`
          value.
        * See how `gitrepositoryurl` value matches your fork of the `mq-infra`
          repository.
      * The second entry informs us that a new `PipelineRun` has been created for
        the pipeline `mq-infra-dev`.
      * The third entry shows us the `event ID` that the event listener is using
        for the processing of this webhook event.

      The order of these events isn't really important; what's clear is that the
      pipeline run has been launched based on the webhook notification processes by
      the event listener.

      The `oc logs` command can be very helpful to diagnose any issues you might
      have with the event listener.

9. *The new pipeline run starts*

      We can now see that our pipeline run has started.

      In the OpenShift web console, navigate to `Pipelines->Pipelines`.  Select the
      `mq-infra-dev` pipeline and `Project: ci`:

      ![diagram101](./images/diagram101.png)

      We can see a new pipeline run for `mq-infra-dev` is in progress.

      Notice the **Labels** for this pipeline run:

      * See how the tekton event listener `cnkt-event-listener` is identified.
      * See how the `trigger-eventid` allows us to link this pipeline run back to
        a Tekton event for traceability.

---

## Exploring the new pipeline run

Our pipeline run has started and will take a little time to complete.

We already know **what** our pipeline run will do; in this section we explore
**how it does it** by looking at the `build` task in more detail.

We'll also review the new and updated resources produced by the pipeline run.
These include new versions of the image and Artifactory Helm chart, as well as
an updated GitOps Helm chart. We'll see how these combine to reflect the change
we made to `QM1`.

1. *Look at pipeline run*

      Let’s find the queue manager pipeline using the OpenShift web console.

      Navigate to `Pipelines->Pipelines` in the left hand pane, and select `Project:
      ci`, to show all pipelines in the ci namespace:

      ![diagram200](./images/diagram200.png){: style="max-height:600px"}

      You can see that there is an `mq-infra-dev` pipeline run in progress. (You may
      need to scroll through the list of pipelines.)

2. *Pipeline tasks run and complete*

      As before, because we can think of a pipeline run as an instance of a
      pipeline, we can see the pipeline structure from its run.

      In the `Pipelines` view, click on the `mq-infra-dev-xxxxx` pipeline run:

      ![diagram201](./images/diagram201.png){: style="max-height:500px"}

      Again, as before, we can see the tasks comprising the run, such as `setup`,
      `build` and `GitOps`. Notice how tasks are in one of three states:
      **completed**, **in progress** or **waiting to run**.

3. *See how the* `build` *task is implemented*

      We're going to examine the `build` task in some detail. Once you see how the
      `build` task works, you'll be able to work out how any task works and how to
      modify it, as well as how to write your own tasks.

      We'll look at the `build` task YAML to see how it's implemented. We'll see how
      it gets its input, how it is coded, how it references secrets and how it
      interacts with external components, for example storing a new image in the
      image registry.

      In the `Pipeline Run details` view, click on the **YAML** tab:

      ![diagram202](./images/diagram202.png)

      This is the pipeline run YAML.  You may recall that a `pipeline` YAML has
      no code defined within it. In contrast, a `pipelinerun`
      YAML includes all the code that implements each task. It means we
      can scroll through a single file to see the pipeline run inputs, the task
      inputs and the code that implements each of the steps within each task. It
      makes it easy to understand what's happening as the pipeline executes because
      everything is in one place.

      We can of course see the same output on the command line.

      Issue the following command, replacing `xxxxx` with the current pipeline run
      identifier:

      ```bash
      oc get pipelinerun mq-infra-dev-xxxxx -n ci -o yaml
      ```

      which will show you all the same information on the command line:

      ``` { .yaml .no-copy }
      apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
         annotations:
            app.openshift.io/runtime: mq
            ...
         creationTimestamp: "2021-08-05T14:31:05Z"
         generateName: mq-infra-dev-
         ...
      ```

      We've only included the beginning of the YAML file, as it's quite large, but
      you can see that it's the same as the web console YAML file.

4. *Exploring the* `build` *task YAML*

      We can locate the source of the task code in our GitOps source repository.

      In the `Pipeline Run details` **YAML** tab view, search for `ibm-build-tag-push-v2-6-13`
      to find where the `build` task refers to its implementation code.

      Use `Cmd+F` to search for `ibm-build-tag-push-v2-6-13`:

      ``` { .yaml .no-copy }
            ...
            - name: build
            params:
               - name: git-url
                  value: $(tasks.setup.results.git-url)
               - name: git-revision
                  value: $(tasks.setup.results.git-revision)
               - name: source-dir
                  value: $(tasks.setup.results.source-dir)
               - name: image-server
                  value: $(tasks.setup.results.image-server)
               - name: image-namespace
                  value: $(tasks.setup.results.image-namespace)
               - name: image-repository
                  value: $(tasks.setup.results.image-repository)
               - name: image-tag
                  value: $(tasks.setup.results.image-tag)
            runAfter:
               - setup
            taskRef:
               kind: Task
               name: ibm-build-tag-push-v2-6-13
            - name: smoke-tests-mq
            ...
      ```

      We've already reviewed this `build` task definition.
      Recall that the `taskRef` refers to the task `ibm-build-tag-push-v2-6-13`
      which implements the task code for `build`.

5. *The* `build` *task YAML source*

      This `build` task is just one of many tasks defined in the
      `mq-pipeline-dev.yaml` file we explored earlier. Lets have
      a look at the definition of this task.

      Issue the following command:

      ```bash
      cat mq/environments/ci/tasks/ibm-build-tag-push-v2-6-13.yaml
      ```

      to show `ibm-build-tag-push-v2-6-13` among the many YAMLs in this file:

      ``` { .yaml .no-copy }
      apiVersion: tekton.dev/v1beta1
      kind: Task
      metadata:
        name: ibm-build-tag-push-v2-6-13
        annotations:
          description: Executes logic to build, tag and push a container image using the intermediate sha tag to the image-url
          app.openshift.io/description: Executes logic to build, tag and push a container image using the intermediate sha tag to the image-url
          app.openshift.io/vcs-uri: https://github.com/IBM/ibm-garage-tekton-tasks
          app.openshift.io/vcs-ref: master
        labels:
          version: 2.6.13
      spec:
        params:
          - name: git-url
          - name: git-revision
            default: master
          - name: source-dir
            default: /source
          - name: image-server
            default: ""
          - name: image-namespace
            default: ""
          - name: image-repository
            default: ""
          - name: image-tag
            default: ""
          - name: BUILDER_IMAGE
            default: quay.io/buildah/stable:v1.15.0
          - name: DOCKERFILE
            default: ./Dockerfile
          - name: CONTEXT
            default: .
          - name: TLSVERIFY
            default: "false"
          - name: FORMAT
            default: "docker"
          - name: STORAGE_DRIVER
            description: Set buildah storage driver
            default: overlay
        volumes:
          - name: varlibcontainers
            emptyDir: {}
          - name: source
            emptyDir: {}
        stepTemplate:
          volumeMounts:
            - name: source
              mountPath: $(params.source-dir)
        steps:
          - name: git-clone
            image: quay.io/ibmgaragecloud/alpine-git
            env:
              - name: GIT_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: git-credentials
                    key: password
                    optional: true
              - name: GIT_USERNAME
                valueFrom:
                  secretKeyRef:
                    name: git-credentials
                    key: username
                    optional: true
            script: |
              set +x
              if [[ -n "${GIT_USERNAME}" ]] && [[ -n "${GIT_PASSWORD}" ]]; then
                  git clone "$(echo $(params.git-url) | awk -F '://' '{print $1}')://${GIT_USERNAME}:${GIT_PASSWORD}@$(echo $(params.git-url) | awk -F '://' '{print $2}')" $(params.source-dir)
              else
                  set -x
                  git clone $(params.git-url) $(params.source-dir)
              fi
              set -x
              cd $(params.source-dir)
              git checkout $(params.git-revision)
          - name: build
            image: $(params.BUILDER_IMAGE)
            workingDir: $(params.source-dir)
            env:
              - name: REGISTRY_USER
                valueFrom:
                  secretKeyRef:
                    name: registry-access
                    key: REGISTRY_USER
                    optional: true
              - name: REGISTRY_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: registry-access
                    key: REGISTRY_PASSWORD
                    optional: true
              - name: IBM_ENTITLED_REGISTRY_USER
                valueFrom:
                  secretKeyRef:
                    name: ibm-entitled-registry-credentials
                    key: IBM_ENTITLED_REGISTRY_USER
                    optional: true
              - name: IBM_ENTITLED_REGISTRY_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: ibm-entitled-registry-credentials
                    key: IBM_ENTITLED_REGISTRY_PASSWORD
                    optional: true
            volumeMounts:
              - mountPath: /var/lib/containers
                name: varlibcontainers
            securityContext:
              privileged: true
            script: |
              APP_IMAGE="$(params.image-server)/$(params.image-namespace)/$(params.image-repository):$(params.image-tag)"
              if [[ -n "${IBM_ENTITLED_REGISTRY_USER}" ]] && [[ -n "${IBM_ENTITLED_REGISTRY_PASSWORD}" ]]; then
                buildah login -u ${IBM_ENTITLED_REGISTRY_USER} -p ${IBM_ENTITLED_REGISTRY_PASSWORD} cp.icr.io
                echo "buildah login -u "${IBM_ENTITLED_REGISTRY_USER}" -p "xxxxx" cp.icr.io"
              fi
              buildah --layers --storage-driver=$(params.STORAGE_DRIVER) bud --format=$(params.FORMAT) --tls-verify=$(params.TLSVERIFY) -f $(params.DOCKERFILE) -t ${APP_IMAGE} $(params.CONTEXT)
              set +x
              if [[ -n "${REGISTRY_USER}" ]] && [[ -n "${REGISTRY_PASSWORD}" ]] && [[ "$(params.image-server)" != "image-registry.openshift-image-registry.svc:5000"  ]]; then
                buildah login -u "${REGISTRY_USER}" -p "${REGISTRY_PASSWORD}" "$(params.image-server)"
                echo "buildah login -u "${REGISTRY_USER}" -p "xxxxx" "$(params.image-server)""
              fi
              set -x
              buildah --storage-driver=$(params.STORAGE_DRIVER) push --tls-verify=$(params.TLSVERIFY) --digestfile ./image-digest ${APP_IMAGE} docker://${APP_IMAGE}
      ```

      Notice how:

      * `spec:` contains a list of `params:` as the set of task inputs.
      * `name: git-revision` is an example of named input, and it has a default
        value specified as `default: master`.
      * `spec:` contains `volumes:` and `volumeMounts:` that apply to every step.

6. *Exploring the* `build` *task steps -- configuration*

      We can explore the `build` task structure using the output from the OpenShift
      web console, the `oc get pipelinerun` command.

      Let's spend a few moments understanding how `build` works; you can review the
      code using your local editor or the OpenShift web console. We've also included
      the relevant snippets:

      ``` { .yaml .no-copy }
      steps:
         - name: git-clone
            image: quay.io/ibmgaragecloud/alpine-git
            env:
            - name: GIT_PASSWORD
               valueFrom:
                  secretKeyRef:
                  name: git-credentials
                  key: password
                  optional: true
            - name: GIT_USERNAME
               valueFrom:
                  secretKeyRef:
                  name: git-credentials
                  key: username
                  optional: true
            script: |
                    ...
         - name: build
            image: $(params.BUILDER_IMAGE)
            workingDir: $(params.source-dir)
            env:
            ...
            volumeMounts:
            - mountPath: /var/lib/containers
               name: varlibcontainers
            securityContext:
            privileged: true
            script: |
                    ...
      ```

      Notice how:

      * The `build` task steps are defined in the `steps:` YAML node.
      * The build task has two steps: `name: git-clone` and `name: build`.
      * The `git-clone` step runs in its own container using `image: quay.io/ibmgaragecloud/alpine-git`
      * The `build` step runs in its own container using the image specified by
        the task input variable `$(params.BUILDER_IMAGE)`. Notice this has a
        default value `quay.io/buildah/stable:v1.15.0`
      * The `git-clone` step sets up an environment variable `GIT_PASSWORD`
        using the the `password` key in the `git-credentials` secret. This allows
        the task to use the git credentials we configured in this chapter
        earlier
      * As required, a step can specify any node from the `spec.containers[]:`
        [specification](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
        such as `workingDir:`, `volumeMount:` and `securityContext:`.
      * The code executed in each step container is specified by its `script:`.

      Let's now see how a `build` step can be coded to use this configuration.

7. *Exploring the* `build` *task steps -- scripts*

      When the `build` step container runs, it executes the code specified in its
      `script:`.

      Find the `build` step script in web console pipelinerun `YAML` output, using the
      `oc` command. We've shown the `script:` below:

      ``` { .yaml .no-copy }
      script: |
         APP_IMAGE="$(params.image-server)/$(params.image-namespace)/$(params.image-repository):$(params.image-tag)"
         if [[ -n "${IBM_ENTITLED_REGISTRY_USER}" ]] && [[ -n "${IBM_ENTITLED_REGISTRY_PASSWORD}" ]]; then
            buildah login -u ${IBM_ENTITLED_REGISTRY_USER} -p ${IBM_ENTITLED_REGISTRY_PASSWORD} cp.icr.io
            echo "buildah login -u "${IBM_ENTITLED_REGISTRY_USER}" -p "xxxxx" cp.icr.io"
         fi
         buildah --layers --storage-driver=$(params.STORAGE_DRIVER) bud --format=$(params.FORMAT) --tls-verify=$(params.TLSVERIFY) -f $(params.DOCKERFILE) -t ${APP_IMAGE} $(params.CONTEXT)
         set +x
         if [[ -n "${REGISTRY_USER}" ]] && [[ -n "${REGISTRY_PASSWORD}" ]] && [[ "$(params.image-server)" != "image-registry.openshift-image-registry.svc:5000"  ]]; then
            buildah login -u "${REGISTRY_USER}" -p "${REGISTRY_PASSWORD}" "$(params.image-server)"
            echo "buildah login -u "${REGISTRY_USER}" -p "xxxxx" "$(params.image-server)""
         fi
         set -x
         buildah --storage-driver=$(params.STORAGE_DRIVER) push --tls-verify=$(params.TLSVERIFY) --digestfile ./image-digest ${APP_IMAGE} docker://${APP_IMAGE}
      ```

      This script will execute within the `step-build` container and will have
      access to the container environment set up for this step and task. The script
      makes extensive use of the `buildah` command; it is often used in combination
      with `skopeo` and `podman` as a compatible alternative to the `docker` command
      for container management. You can learn more about these commands in this
      [short
      article](https://www.redhat.com/en/blog/say-hello-buildah-podman-and-skopeo).

      See how the script will:

      * Use the `buildah login` command to login to the IBM Entitled Registry
        using the `$IBM_ENTITLED_REGISTRY_USER` and
        `$IBM_ENTITLED_REGISTRY_PASSWORD` credentials you set up
        earlier. It needs this access because the Dockerfile
        used to build the queue manager image uses an IBM entitled MQ image.
      * Use the `buildah bud` command to build a queue manager image using the
        Dockerfile specified in `$params.DOCKERFILE`.
      * Use the `buildah push` command to store the newly built queue manager
        image to the cluster image registry. The step does support external
        registries such as `quay.io` if `$REGISTRY_USER` and `$REGISTRY_PASSWORD`
        are set.
      * Name the queue manager image using the `$APP_IMAGE` variable. See how the
        name is structured as
        `$(params.image-server)/$(params.image-namespace)/$(params.image-repository):$(params.image-tag)`
        to uniquely identify it. We'll see an example in a moment.

8. *Linking task output to the* `build` *log*

      As well as reading the script, we can **see the actual** output generated by
      the script OpenShift web console.

      In the web console, select the `Pipeline Run details` **Logs** tab for
      `mq-infra-dev-xxxxx` and select the `build` task. Scroll down to the
      `STEP-BUILD` output:

      ``` { .text .no-copy }
      STEP-BUILD
         + APP_IMAGE=image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76
         + [[ -n cp ]]
         + [[ -n eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE1OTE3MDA1MTgsImp0aSI6Ijk3ODY4NWY2OTBlYjQwYzhhYzE0YTZjYTMxODA0OTgzIn0.a_pcyExLFeCMRks0DjFwW-VQAM_U5imFwEOu_qm79hY ]]
         + buildah login -u cp -p eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE1OTE3MDA1MTgsImp0aSI6Ijk3ODY4NWY2OTBlYjQwYzhhYzE0YTZjYTMxODA0OTgzIn0.a_pcyExLFeCMRks0DjFwW-VQAM_U5imFwEOu_qm79hY cp.icr.io
         Login Succeeded!
         + echo 'buildah login -u cp -p xxxxx cp.icr.io'
         buildah login -u cp -p xxxxx cp.icr.io
         + buildah --layers --storage-driver=overlay bud --format=docker --tls-verify=false -f ./Dockerfile -t image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76 .
         STEP 1: FROM cp.icr.io/cp/ibm-mqadvanced-server-integration@sha256:cfe3a4cec7a353e7496d367f9789dbe21fbf60dac46f127d288dda329560d13a
         Getting image source signatures
         Copying blob sha256:9bd64c9a8b8aef587728a77441425dfb80f760372ffb25bf4df746b1119812c5
         Copying blob sha256:33d29d2675612fb16d6440409730a67f6bfcd92e79fb34fc868084d99de717c0
         ...
         Copying config sha256:f573afcfc94cdd00bed6f497048f8adb9968475cdecd86a66aa576f0b3267d5a
         Writing manifest to image destination
         Storing signatures
         STEP 2: COMMIT image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76
         --> f573afcfc94
         f573afcfc94cdd00bed6f497048f8adb9968475cdecd86a66aa576f0b3267d5a
         + set +x
         + buildah --storage-driver=overlay push --tls-verify=false --digestfile ./image-digest image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76 docker://image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76
         Getting image source signatures
         Copying blob sha256:90c2e42f948b524cf98005073e0b0aa2065160abf9e8b314976c064e270d92ac
         ...
         Copying config sha256:f573afcfc94cdd00bed6f497048f8adb9968475cdecd86a66aa576f0b3267d5a
         Writing manifest to image destination
         Storing signatures
      ```

      This is the output from the container that runs the `build` step. You can see
      the results of the `script:` execution. Scripts often use the `set +x`, `set
      +e` option and `echo` commands to to generate log output.

      For example, we can see that the queue manager image name is:

      ``` { .text .no-copy }
      + APP_IMAGE=image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76
      ```

      We can also see this image being pushed to the OpenShift image registry:

      ``` { .text .no-copy }
      + buildah --storage-driver=overlay push --tls-verify=false --digestfile ./image-digest image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76 docker://image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76
      ```

      together with its successful completion:

      ``` { .text .no-copy }
      ...
      Copying config sha256:f573afcfc94cdd00bed6f497048f8adb9968475cdecd86a66aa576f0b3267d5a
      Writing manifest to image destination
      Storing signatures
      ```

9. *Accessing the build task logs using* `oc logs`

      Each task in the `mq-infra-dev-xxxxx` pipeline run executes in its own pod.
      Within these pods, each step runs in its own container. This allows us to view the `pipelinerun` output using the `oc logs` command.

      Issue the following command, replacing `xxxxx` with your pipelinerun
      identifier:

      ```bash
      oc get pods -n ci | grep mq-infra-dev-xxxxx
      ```

      You see all the pods used to execute the different tasks in the pipeline run:

      ``` { .bash .no-copy }
      mq-infra-dev-mhpw9-build-mqtlq-pod-glrgn             0/2     Completed   0          2d18h
      mq-infra-dev-mhpw9-gitops-8tkbp-pod-cz2rc            0/1     Completed   0          2d18h
      mq-infra-dev-mhpw9-helm-release-v2crx-pod-fqjwq      0/2     Completed   0          2d18h
      mq-infra-dev-mhpw9-img-release-w9lpx-pod-xnk26       0/1     Completed   0          2d18h
      mq-infra-dev-mhpw9-img-scan-h7cnd-pod-9xfj7          0/3     Completed   0          2d18h
      mq-infra-dev-mhpw9-setup-fz8vs-pod-h4pdm             0/1     Completed   0          2d18h
      mq-infra-dev-mhpw9-smoke-tests-mq-dc5kc-pod-ngrjb    0/4     Completed   0          2d18h
      mq-infra-dev-mhpw9-tag-release-n7nx2-pod-jc5rf       0/2     Completed   0          2d18h
      ```

      Notice that:

      * the number of pods matches the number of tasks in the `mq-infra-dev`
        pipeline. (In the above example, the pipeline run has finished.)
      * the name of the pod includes the name of the task in the pipeline.
      * each step runs in its own container, for example, the `build` pod has
        `0/2` containers running.

      We can view the logs the of any step in a tasks by viewing its container log.

      Issue the following command:

      ```bash
      oc logs mq-infra-dev-mhpw9-build-mqtlq-pod-glrgn step-build -n ci
      ```

      to see the `step-build` log in the `build` task:

      ``` { .text .no-copy }
      + APP_IMAGE=image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76
      + [[ -n cp ]]
      + [[ -n eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE1OTE3MDA1MTgsImp0aSI6Ijk3ODY4NWY2OTBlYjQwYzhhYzE0YTZjYTMxODA0OTgzIn0.a_pcyExLFeCMRks0DjFwW-VQAM_U5imFwEOu_qm79hY ]]
      + buildah login -u cp -p eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE1OTE3MDA1MTgsImp0aSI6Ijk3ODY4NWY2OTBlYjQwYzhhYzE0YTZjYTMxODA0OTgzIn0.a_pcyExLFeCMRks0DjFwW-VQAM_U5imFwEOu_qm79hY cp.icr.io
      Login Succeeded!
      + echo 'buildah login -u cp -p xxxxx cp.icr.io'
      buildah login -u cp -p xxxxx cp.icr.io
      + buildah --layers --storage-driver=overlay bud --format=docker --tls-verify=false -f ./Dockerfile -t image-registry.openshift-image-registry.svc:5000/ci/mq-infra:840ec76 .
      STEP 1: FROM cp.icr.io/cp/ibm-mqadvanced-server-integration@sha256:cfe3a4cec7a353e7496d367f9789dbe21fbf60dac46f127d288dda329560d13a
      Getting image source signatures
      Copying blob sha256:9bd64c9a8b8aef587728a77441425dfb80f760372ffb25bf4df746b1119812c5
      ...
      ```

      See how this is the same output as the web console `Pipeline Run details`
      view.

10. *Wait for pipeline run to finish*

      Return to `Pipeline Run details` view.  If the pipeline run hasn't finished
      then wait until it does. If it hasn't finished, explore some of the logs of
      the tasks still running, and see if you can find the step code that they are
      executing.

      When the pipeline run completes, you'll see:

      ![diagram203](./images/diagram203.png)

      The pipeline run is now complete. Let's explore the artifacts that
      `mq-infra-dev-xxxxx` has updated and created.

11. *New Helm chart in Artifactory*

      The first new artifact we look at is the new Helm chart in Artifactory.

      Navigate to the Artifactory UI

      On the left hand pane select `Artifactory->Artifacts`.

      Within this view, expand the following zipped Helm charts:

      * `generic-local->ci->mq-infra-0.0.1.tgz->mq-infra`
      * `generic-local->ci->mq-infra-0.0.2.tgz->mq-infra`

      Select the `Chart.yaml` in `mq-infra-0.0.2.tgz->mq-infra` and `View Source`
      tab:

      ![diagram204](./images/diagram204.png)

      Notice that:

      * The pipeline run created a new zipped version of the Helm chart
        `mq-infra-0.0.2.tgz`
      * The `tgz` file name includes a version change from `0.0.1` to `0.0.2`.
      * The `Chart.yaml` shows also contains a new version `0.0.2`.
      * The `config.mqsc` file in version `0.0.2` is significantly larger than
        its equivalent in `0.0.1` corresponding to the new MQSC definition for
        `IBM.DEMO.Q.NEW`

      In summary, the `helm-release` task in the `mq-infra-dev` pipeline run has
      created a new version of the Helm chart containing the new MQSC definition and
      stored it in Artifactory.

12. *The importance of Git tags*

      Notice how the new Helm chart in Artifactory has a version `0.0.2`,
      incremented from version `0.0.1`. Every time we make a change to a source
      queue manager repository, a pipeline run generates a new version of the Helm
      chart to capture this change.  As Helm charts (and container images) evolve
      over time, its version tag reflects the change; the highest version tag always
      points to the latest version of a Helm chart or image.

      This version matches the [git
      tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) of the `mq-infra`
      repository for the most recent change to update the `config.mqsc` file to add
      the `IBM.DEMO.Q.NEW` queue to `QM1`.

      We can see the tags for `mq-infra` in the GitHub for via the `git tags`
      command.

      Issue the following command:

      ```bash
      echo https://github.com/$GIT_ORG/mq-infra/tags
      ```

      to determine the URL for the `mq-infra` **tags** UI, for example:

      ``` { .text .no-copy }
      https://github.com/odowdaibm-mq-guide/mq-infra/tags
      ```

      Copy your generated URL into your browser to launch the `Tags` web page for
      your fork of the `mq-infra` GitHub repository.

      (*You can safely ignore any browser certificate warnings.*)

      You'll see the following page:

      ![diagram205](./images/diagram205.png)

      Notice:

      * Two tags: `0.0.1` and `0.0.2`.
      * `0.0.1` was generated by the first (manual) pipeline run.
      * `0.0.2` was generated by the second (webhook triggered) pipeline run.
      * These tags are used to version the Helm charts in Artifactory.
      * These tags are used by other tasks in the `mq-infra-dev` pipeline such as
        `image-release` and `gitops`.

13. *Generating the tag version*

    When the `mq-infra` repository is forked, it has no tags. Tags are created by the `tag-release` task in the `mq-infra-dev` pipeline. Let's spend a few moments exploring this task:

    In the `Pipeline Run details` view, select the `tag-release` task and **Logs** tab:

    Scroll to the bottom of the log output, which culminates in:

    ``` { .text .no-copy }
    Pushing to https://odowdaibm:xxxxx@github.com/odowdaibm-mq-guide/mq-infra
    POST git-receive-pack (340 bytes)
    To https://github.com/odowdaibm-mq-guide/mq-infra
    = [up to date]      0.0.1 -> 0.0.1
    * [new tag]         0.0.2 -> 0.0.2
    ++ git describe --abbrev=0 --tags
    + NEW_TAG=0.0.2
    + [[ -z 0.0.2 ]]
    + echo -n 0.0.2
    + tee /tekton/results/tag
    0.0.2
    ```

    We can see that the end result of this task is to push the new `0.0.2` tag to
    your fork of the `mq-infra` repository. This tag can now be used throughout
    the pipeline whenever we need to version a Helm chart, container image or any
    other generated artifact.

    Spend a few moments examining the **full** log output of the `step-git-tag`
    step in the `tag-release` task.

    You might also like to explore the `ibm-tag-release-v2-6-13` task code:

    ``` { .yaml .no-copy }
    apiVersion: tekton.dev/v1beta1
    kind: Task
    metadata:
    name: ibm-tag-release-v2-6-13
    ...
    spec:
    ...
    steps:
        ...
        - name: git-tag
        image: $(params.js-image)
        workingDir: $(params.source-dir)
        script: |
          #!/usr/bin/env bash
          set -ex

          echo "Current branch: $(git rev-parse --abbrev-ref HEAD)"
          git fetch --tags
          git config --global user.email "cloud-native-toolkit@example.com"
          git config --global user.name "Cloud Native Toolkit Pipeline"
          if [[ $(git describe --tag `git rev-parse HEAD`) =~ (^[0-9]+.[0-9]+.[0-9]+$) ]]; then
              echo "Latest commit is already tagged"
              NEW_TAG="$(git describe --abbrev=0 --tags)"
              echo -n "${NEW_TAG}" | tee $(results.tag.path)
              exit 0
          fi
          mkdir -p ~/.npm
          npm config set prefix ~/.npm
          export PATH=$PATH:~/.npm/bin
          npm i -g release-it

          release-it patch \
              --ci \
              --no-npm \
              --no-git.push \
              --no-git.requireCleanWorkingDir \
              --no-git.requireUpstream \
              -VV

          if [[ -z "$(params.skip-push)" ]]; then
              set +x
              git push --tags -v
              set -x
          fi

          NEW_TAG="$(git describe --abbrev=0 --tags)"
          if [[ -z "${NEW_TAG}" ]]; then
              echo "Error: NEW_TAG not defined"
              exit 1
          fi
          echo -n "${NEW_TAG}" | tee $(results.tag.path)
    ```

    See if you can work out how the code in the above `script:` generates
    `git-tag` log output.

14. *Updated Helm chart in GitOps*

      The Helm chart stored in Artifactory is often referred to as a **base chart**
      because it contains the default configuration for `QM1`. It can be overridden
      by the Helm chart stored in the GitOps repository, the GitOps Helm chart uses a `requirements.yaml`
      file to identify the relationship between these two Helm charts.

      Let's explore the GitOps Helm chart that was updated by the pipeline run.

      Issue the following command:

      ```bash
      echo https://github.com/$GIT_ORG/multi-tenancy-gitops-apps/blob/$GIT_BRANCH/mq/environments/dev/mq-infra/requirements.yaml
      ```

      to calculate the URL of the GitOps version of the `requirements.yaml` file,
      for example:

      ``` { .text .no-copy}
      https://github.com/odowdaibm-mq-guide/multi-tenancy-gitops/blob/master/mq/environments/dev/mq-infra/requirements.yaml
      ```

      Copy your version of this URL into a browser to see the GitOps version of the
      `requirements.yaml` file that is being accessed by ArgoCD:

      ``` { .text .no-copy}
      dependencies:
         - name: mq-infra
            version: 0.0.2
            repository: http://artifactory-artifactory.tools:8082/artifactory/generic-local/ci
      ```

      Notice:

      * the `version:` has been updated to `0.0.2`.
      * this chart will now override
        the newly generated `mq-infra-0.0.2.tgz` base chart in Artifactory.
      * the `0.0.1` version of the base chart is remains in Artifactory as we saw
        earlier, but is no longer referenced.

      It's the `gitops` task in the `mq-infra-dev` pipeline run that updated
      this Helm chart stored in the GitOps repository.

      You might like to explore the `gitops` step log in `Pipeline Run detail` **Logs** tab.

      When the queue manager ArgoCD application sees that this GitOps Helm chart has
      changed, it will redeploy this Helm chart. It will be the combination of the
      GitOps and Artifactory Helm charts that will define the new queue manager
      deployment in the the cluster.

15. *Viewing the new deployment in ArgoCD*

      The completion of the `gitops` task results in an update to the GitOps Helm
      chart `requirements.yaml` file to identify the base Helm chart version `0.0.2`
      in Artifactory. This change to the `requirements.yaml` file will result in the
      `ibm-mq-dev-instance` ArgoCD application redeploying the Helm chart. Let's
      explore this change using ArgoCD.

      In the ArgoCD UI, select the `ibm-mq-dev-instance` ArgoCD application:

      ![diagram206.png](./images/diagram206.png){: style="max-height:200px"}

      We can see the two components of the queue manager Kubernetes deployment,
      namely the `queuemanager` custom resource defining the properties of `QM1` and
      the `configmap` containing its MQSC configuration.

      Click on the `qm-dev` `queuemanager` custom resource and select the
      **DESIRED MANIFEST** tab:

      ![diagram207.png](./images/diagram207.png)

      Notice:

      * This queue manager uses the newly built container image
        `image-registry.openshift-image-registry.svc:5000/ci/mq-infra:0.0.2`
        stored in the OpenShift image registry.
      * This image was generated by the `build` task and tagged by
        the `tag-release` task as `0.0.2`.
      * The image version `0.0.2` matches the git repository tag `0.0.2` in the
        same way as the Helm charts.

      Now click on the `mqsc-configmap` `configmap` and select the
      **DESIRED MANIFEST** tab:

      ![diagram208.png](./images/diagram208.png)

      Notice:

      * New queue `IBM.DEMO.Q.NEW` has been added to the MQSC file.
      * This configmap matches the configmap we updated in `QM1` source
        repository.

      Let's now try out the new queue manager.

---

## Verify the updates to QueueManager

Let's now verify and explore the changes we have made to `QM1`, using both the
MQ web console and the `oc` command.

1. *Exploring* `QM1` *with the web console*

      Switch to the MQ web console

      Select `Manage QM1`from the home page to see:

      ![diagram220](./images/diagram220.png){: style="max-height:700px"}

      Notice that:

      * The new queue `IBM.DEMO.Q.NEW` has been added to the list of queues for
        `QM1`. This is the queue [we added](#anchor1) to the `config.mqsc` file in the source
        repository.
      * The **Maximum depth** for `IBM.DEMO.Q` is `0/5000` even though we put a
        message to this queue in the previous topic. We'll
        find out a little later what's happened to this message.

2. *Exploring the queue manager details*

      To understand a little more about the changes we've made to `QM1`, let's
      examine its statistics page on the MQ console.

      Click on the **Configuration** icon in the top right of the `Manage` view.

      In the `QM configuration` view, select **Statistics** in the left hand pane:

      ![diagram221](./images/diagram221.png)

      Notice the details of when `QM1` was created, as well as its QMID. The QMID is
      a unique name generated when a queue manager is first created.

      Let's compare these values to the previous values which
      we generated using the `oc` command.  We've copied the most relevant ones below:

      ```{ .text .no-copy}
      5724-H72 (C) Copyright IBM Corp. 1994, 2020.
      Starting MQSC for queue manager QM1.
            1 : dis qmgr
      AMQ8408I: Display Queue Manager details.
         QMNAME(QM1)
         ...                                     ALTDATE(2021-07-30)
         ALTTIME(17.17.48)                       ...
         CRDATE(2021-07-30)                      CRTIME(17.17.44)
         ...                                     QMID(QM1_2021-07-30_17.17.44)
      One MQSC command read.
      No commands have a syntax error.
      All valid MQSC commands were processed.
      ```

      Notice  that these two sets of values are **completely different**; it's as if
      we've got a new queue manager.  Indeed, that's exactly what happened when we
      made our configuration change, as we'll now discover.

3. *Ephemeral queue manager*

      As we've seen, pods are the fundamental building blocks of Kubernetes. The
      `QM1` queue manager runs in a pod container and when its configuration
      changes, this pod is **replaced** by a new one that whose container has an
      updated `spec:` that reflects the updated configuration for `QM1`. In our
      scenario this includes both a new image and a new MQSC file containing our new
      queue definition.

      Let's look a little more closely at the `queuemanager` custom resource for
      `QM1` to understand what's happening.

      Issue the following command:

      ```bash
      oc get queuemanager qm-dev -n dev -o yaml
      ```

      to show the details of the queue manager:

      ```{ .yaml .no-copy hl_lines="49 50"}
      apiVersion: mq.ibm.com/v1beta1
      kind: QueueManager
      ...
      spec:
         license:
            accept: true
            license: L-RJON-BN7PN3
            metric: VirtualProcessorCore
            use: NonProduction
         queueManager:
            availability:
            type: SingleInstance
            debug: false
            image: image-registry.openshift-image-registry.svc:5000/ci/mq-infra:0.0.2
            imagePullPolicy: Always
            livenessProbe:
            failureThreshold: 1
            initialDelaySeconds: 90
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
            logFormat: Basic
            metrics:
            enabled: true
            mqsc:
            - configMap:
               items:
               - config.mqsc
               name: mqsc-configmap
            name: QM1
            readinessProbe:
            failureThreshold: 1
            initialDelaySeconds: 10
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 3
            resources:
            limits:
               cpu: "1"
               memory: 1Gi
            requests:
               cpu: "1"
               memory: 1Gi
            route:
            enabled: true
            storage:
            persistedData:
               enabled: false
            queueManager:
               type: ephemeral
            recoveryLogs:
               enabled: false
         securityContext:
            initVolumeAsRoot: false
         template:
            pod:
            containers:
            - env:
               - name: MQSNOAUT
                  value: "yes"
               name: qmgr
               resources: {}
         terminationGracePeriodSeconds: 30
         tracing:
            agent: {}
            collector: {}
            enabled: false
            namespace: ""
         version: 9.2.2.0-r1
         web:
            enabled: true
      status:
         adminUiUrl: https://qm-dev-ibm-mq-web-dev.mqcloud1-odowda-d02cf90349a0fe46c9804e3ab1fe2643-0000.eu-gb.containers.appdomain.cloud/ibmmq/console
         conditions: []
         endpoints:
         - name: ui
            type: UI
            uri: https://qm-dev-ibm-mq-web-dev.mqcloud1-odowda-d02cf90349a0fe46c9804e3ab1fe2643-0000.eu-gb.containers.appdomain.cloud/ibmmq/console
         name: QM1
         phase: Running
         versions:
            available:
            channels: []
            versions:
            - name: 9.2.2.0-r1
            reconciled: 9.2.2.0-r1
      ```

      Notice the `spec.queueManager.storage.queueManager.type: ephemeral`. As you
      can see from the [API reference for the
      QueueManager](https://www.ibm.com/docs/en/ibm-mq/9.2?topic=mqibmcomv1beta1-api-reference-queuemanager#ctr_api_v1beta1_QueueManager__.spec.queueManager.storage.queueManager)
      this defines an **ephemeral** queue manager.

      An ephemeral queue manager is one whose logs and queue files are defined
      **within the container file system** rather than external to it. It means that
      when the pod is replaced, all the container queue files and log files are
      lost. That's why our QMID is changed and our message was lost -- they were
      stored in the file system of the old container. When the pod restarted for the
      updated queue manager deployment, it was like a new queue manager was being
      created because the container file system was new.

      We rarely use ephemeral queue managers for production work; usually a queue
      manager stores its queue and log files in a [persistent
      volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) using
      a persistent volume claim that is mounted into the container when the pod
      starts.

      Using a persistent volume means that   restarting a pod doesn't affect the
      long term state of the queue manager -- properties like QMID, together with
      messages in queues are stored in queue and log files that are preserved in the
      persistent volume claim that is reattached to a restarted container pod. We'll
      use persistent volumes in the [High Availability](../high-availability/overview.md)
      chapter.

      For now, our use of an ephemeral queue manager has shown nicely how a
      configuration change results in an updated queue manager deployment resulting
      in a pod restart for the queue manager.

!!! success "Congratulations!"
    You've now got a fully automated CICD process for your queue manager.

    You started the chapter by configuring a webhook to generate an event notification whenever the `QM1` source repository was changed. You then created an event listener and trigger in your cluster to process these events and start a pipeline run using the data in the event. After updating the `QM1` source repository to add a new queue to its MQSC definitions, a pipeline run was triggered automatically.

    As the pipeline run progressed, you explored its task logs and how key tasks worked. You explored the detailed structure of the `mq-infra-dev` pipeline in terms of its tasks and their steps. You explored theses task steps produced new and updated artifacts such as a container image and Helm charts. You saw how git tags were used to version control these artifacts.

    Once the pipeline run completed, you examined the updated queue manager deployment using ArgoCD. You tried out the updated queue manager to confirm that the new queue had been added to `QM1`. Finally, you explored the concept of an ephemeral queue manager its persistent volume, together with how these combine with Kubernetes pods and containers.

    In the next chapter, we're going to create an MQ application that exploits this queue manager.
