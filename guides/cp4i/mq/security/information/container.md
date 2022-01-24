# Docker container Security

As a best practice, each workload should run with its own unique service account.
A service account is a Kubernetes resource that is used by Kubernetes entities such as pods to authenticate itself.

By having a unique service account per workload, RBAC policies can be applied for each service account to limit the resources that a service account can access.

The `mq-spring-app` workload is designed to run with a service account with the same name of `mq-spring-app`.
When the helm chart gets deployed, the service account gets created.  
See: https://github.com/cloud-native-toolkit/mq-spring-app/blob/master/chart/base/templates/serviceaccount.yaml

The service account manifest created is defined as follows:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mq-spring-app
automountServiceAccountToken: false
```

The `automountServiceAccountToken: false` will ensure that the service account secret containing the authentication token does not get mounted.  This ensures, that a malicious user who gets access to the pod, cannot get the service account token and use it for accessing the cluster.

The deployment resource is configured to run pods using the `mq-spring-app` service account.
See: https://github.com/cloud-native-toolkit/mq-spring-app/blob/457fcd26564187397ed5de07cc1c37a1ead5faf8/chart/base/templates/deployment.yaml#L36

THe `spec` section of the deployment.yaml file has the following entry:
```
spec:
  serviceAccountName: mq-spring-app
```
