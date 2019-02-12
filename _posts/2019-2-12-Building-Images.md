---
layout: post
title: Building Images without a Docker Deamon
---

## Motivation
While there have been many developments in the operations of containers in the last years, the creation process of container images has basically remained the same. Still the majority of people are using Docker to create images. In this first blog post I want to take a look at alternatives to the "old fashioned" docker deamon.

## Problem
The use of Docker to create images is nothing to complain about. However, there are some boundary conditions of an architectural and safety nature that make it difficult or nowadays even impossible to use, for example, in Kubernetes. The docker deamon itself is monolithic and if an image should be build, privileged access to the Linux kernel is needed. If you are interested in the technical background, please refer to [this article] (https://blog.jessfraz.com/post/building-container-images-securely-on-kubernetes/) or [this article](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/).

So as soon docker builds should happen inside a kubernetes cluster (e.g for scaled CI with Jenkins) one option is to use the volume mechanism to pass the docker socket for controlling the daemon into the build container.

Of course, this has implications for system security because it gives control over parts of the host system in containers. Therefore, in many environments this is not an option. Furthermore the cloud providers continuously improve security of there kubernetes implementations, which means there is [no docker engine at all](https://cloud.google.com/blog/products/containers-kubernetes/containerd-available-for-beta-testing-in-google-kubernetes-engine) which could even be used for building. IBM Kubernetes Service and Google Container Engine for example offer only a container runtime, which has no functionalities to build images, instead it focuses on efficient execution of images and is also more performant then a classical docker deamon.

Another option is to use Docker-in-Docker images. A Docker daemon is started in a separate container and driven out of the build container. For example, in a Kubernetes cluster, this can be done as another container on the same pod. This has defused the release of control over the host system. However, the Docker-in-Docker design also requires privileged access to the Linux kernel.

## Now how to solve this Problem?

In some cases, cloud providers with products such as Google Cloud Container Builder or Azure Container Registry Build also provide this functionality as a service, but not everyone offers this functionality yet. Another option would be to just use a dedicated VM to build images, but this requires additonal maintenance efforts and maybe not as efficient as scaling build containers on demand.

An option which I like very much is [The Google Java Image Builder](https://github.com/GoogleContainerTools/jib) (JIB). It offers a java based build process for your application and is fully integrated to build Tools like maven and Gradle. 

* Reproducible - Rebuilding your container image with the same contents always generates the same image. Never trigger an unnecessary update again.
- Fast - Deploy your changes fast. Jib separates your application into multiple layers, splitting dependencies from classes. Now you don’t have to wait for Docker to rebuild your entire Java application - just deploy the layers that changed.
- Daemonless - Reduce your CLI dependencies. Build your Docker image from within Maven or Gradle and push to any registry of your choice. No more writing Dockerfiles and calling docker build/push.


