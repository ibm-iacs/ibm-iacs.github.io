# Docker image Security

The docker image that is created when building the `mq-spring-app` addresses several security considerations.
The git repo for the app contains a Dockerfile that is used to build the image.  
See the [Dockerfile](https://github.com/cloud-native-toolkit/mq-spring-app/blob/master/Dockerfile).
This Dockerfile is a multistage file where the first stage builds the java code and the second stage builds the runtime image.  The second stage of the Dockerfile looks as follows:

```
FROM registry.access.redhat.com/ubi8/ubi:8.4

RUN dnf install -y java-11-openjdk.x86_64

COPY --from=builder /workspace/app/target/*.jar ./app.jar

EXPOSE 8080/tcp
USER 1001

CMD ["java", "-jar", "./app.jar"]
```

## Use a secure and trusted base images

The base image used is the ubi8:8.4 (Red Hat Universal Base Image version 8.4)

Red Hat UBI images are tested, maintained and contain the latest security patches.
See the [UBI eBook](https://connect.redhat.com/en/partner-resources/red-hat-ubi-ebook) to understand the advantages of using UBI images.

A [blog](https://snyk.io/blog/ubi-to-minimize-container-vulnerabilities/) by a leading security company also recommends using Red Hat UBI base images as they typically have much fewer security vulnerabilities.

## Do not run as a root user

By default, if no `USER` is specified in a Dockerfile, the container will run as root on a Kubernetes platform.
A security best practice is to specify a non root user in the Dockerfile.
In our Dockerfile, we specify to run as USER 1001 which is a non root user.

Note: OpenShift Container Platform, by default, does not allow containers to run as root but it is still a good practice to specify in the Dockerfile a non-root user. In fact OpenShift ignores the USER directive of the Dockerfile and launches the container with a random UUID for enhanced security.
