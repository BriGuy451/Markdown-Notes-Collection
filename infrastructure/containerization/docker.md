# Docker

Content

- Overview
- Docker In-Depth
- Images
- Containers
- Containerizing Applications (Dockerfile)
- Volumes & Persistent Data
- Docker Networking
- Docker Overlay Networking
- **Don't forget to add source**

## Overview

Docker is software that runs on Linux and Windows. It creates, manages, and can even orchestrate containers. The software is currently built from various tools from the Moby open-source project. Docker, Inc. is the company that created the technology and continues to create technologies and solutions that make it easier to get the code on your laptop running in the cloud.

## Docker In-Depth

There are at least three things to be aware of when referring to Docker as a technology:

1. The runtime
2. The daemon (a.k.a. engine)
3. The orchestrator

The runtime operates at the lowest level and is responsible for starting and stopping containers (this includes building all of the OS constructs such as namespaces and control groups).

The low-level runtime is called **runc**, its job is to interface with the underlying OS and start and stop containers. Every running container on a Docker node has a runc instance managing it. The higher-level runtime is called **containerd**. containerd does a lot more than runc, it manages the entire lifecycle of a container, including pulling images, creating network interfaces, and managing lower-level runc instances. containerd is the small specialized part of Docker that does the low-level tasks of starting and stopping containers.

The Docker daemon **(dockerd)** sits above containerd and performs higher-level tasks such as; exposing the Docker remote API, managing images, managing volumes, managing networks, and more…

The orchestrator is something that will not be talked about in these notes, it is essentially a tool to manage multiple containers. The technologies that do this are Docker Compose, Docker Swarm, Kubernetes, Elastic Container Service, and many other orchestration tools.

## Images

It’s useful to think of a Docker **image** as an object that contains an OS filesystem, an application, and all application dependencies. It’s like a virtual machine template, a virtual machine template is essentially a stopped virtual machine. In the Docker world, an image is effectively a stopped container. If you’re a developer, you can think of an image as a class. It’s also worth noting that each image gets its own unique ID. When referencing images, you can refer to them using either IDs or names.

**We store images in centralized places called image registries**, this makes it easy to share and access them.

Images are made up of loosely-connected read-only layers that are stacked on top of each other, with each layer comprising one or more files. Docker takes care of stacking these layers and representing them as a single unified object. Inside of the image is a cut-down operating system (OS) and all of the files and dependencies required to run an application.

The layers are where the data lives (files, code, etc.). Each layer is fully independent, and has no concept of being part of an overall bigger image. **All Docker images start with a base layer, and as changes are made and new content is added, new layers are added on top.** Multiple images can, and do, share layers. This leads to efficiencies in space and performance. Deleting an image will remove the image and all of its layers from your Docker host.

Containers are intended to be fast and lightweight, images tend to be small.

Commands: `docker image pull` `docker image ls` `docker image rm` ([Docker_Image_Commands](https://docs.docker.com/reference/cli/docker/image/))

## Containers

A **container** is the runtime instance of an image. You start one or more containers from a single image. It’s also common for containers to be based on minimalist images that only include software and dependencies required by the application. **Containers run until the app they are executing exits.**

The Container Model: The server is powered on and the OS boots. In the Docker world this can be Linux, or a modern version of Windows that supports the container primitives in its kernel. Similar to the VM model, the OS claims all hardware resources. On top of the OS, we install a container engine such as Docker. The container engine then takes OS resources such as the process tree, the filesystem, and the network stack, and carves them into isolated constructs called containers. Each container looks, smells, and feels just like a real OS. Inside of each container we run an application. One thing that’s not so great about the container model is security. Out of the box, containers are less secure and provide less workload isolation than VMs. Technologies exist to secure containers and lock them down.

At a high level, hypervisors perform hardware virtualization — they carve up physical hardware resources into virtual versions called VMs. On the other hand, containers perform OS virtualization — they carve OS resources into virtual versions called containers.

Containers are designed to be immutable. This is just a buzzword that means read-only — it’s a best practice not to change the configuration of a container after it’s deployed. If something breaks or you need to change something, you should create a new container with the fixes/updates and deploy it in place of the old container. You shouldn’t log into a running container and make configuration changes. Docker provides volumes that exist separately from the container, but can be mounted into the container at runtime. You can stop, start, pause, and restart a container as many times as you want. It’s not until you explicitly delete a container that you run a chance of losing its data. Even then, if you’re storing data outside the container in a volume, that data’s going to persist even after the container has gone.

It’s often a good idea to run containers with a restart policy. This is a form of self-healing that enables Docker to automatically restart them after certain events or failures have occurred.

Commands: `docker container run` `docker container ls` `docker container exec` `docker container start` `docker container rm` `docker container inspect` `Ctrl-PQ` ([Docker_Container_Commands](https://docs.docker.com/reference/cli/docker/container/))

## Containerizing Applications (Dockerfile)

Docker is all about taking applications and running them in containers. The process of taking an application and configuring it to run as a container is called **“containerizing”**. Containers are all about making applications simple to build, ship, and run.

The process of containerizing an application looks like this:

1. Start with your application code and dependencies
2. Create a `Dockerfile` that describes your app, its dependencies, and how to run it
3. Feed the `Dockerfile` into the `docker image build` command
4. Push the new image to a registry (optional)
5. Run container from the image

A `Dockerfile` is a plain-text document that tells Docker how to build an application and dependencies into a Docker image, it is the starting point for creating a container image.

The directory containing the application and dependencies is referred to as the **build context**. It’s a common practice to keep your `Dockerfile` in the root directory of the build context. It’s also important that `Dockerfile` starts with a capital “D” and is all one word.

**All Dockerfiles start with the FROM instruction. This will be the base layer of the image, and the rest of the app will be added on top as additional layers.** The basic premise is this — if an instruction is adding content such as files and programs to the image, it will create a new layer. If it is adding instructions on how to build the image and run the application, it will create metadata.

- Commands That Add Image Layers (Non-Exhaustive): `FROM`, `RUN`, `COPY`
- Commands That Don't Add Image Layers (Non-Exhaustive): `WORKDIR`, `EXPOSE`, `ENTRYPOINT`, `ENV`

It is considered a good practice to use images from official repositories with the FROM instruction. This is because their content has been vetted and they are quick to release new versions when vulnerabilities are fixed. It is also a good idea to start from (FROM) small images as this keeps images small and reduces attack surface and potential vulnerabilities.

> The RUN apk add --update nodejs nodejs-npm instruction uses the Alpine apk package manager to install nodejs and nodejs-npm into the image. It creates a new image layer directly above the Alpine base layer, and installs the packages in this layer. The COPY . /src instruction creates another new layer and copies in the application and dependency files from the build context. (book excerpt)

Dockerfile's are a great form of documentation. It’s a great document for bridging the gap between development and operations. It also has the power to speed up on-boarding of new developers. This is because the file accurately describes the application and its dependencies in an easy-to-read format. You should treat it like you treat source code and check it into a version control system.

### Multi-Stage Builds

**Docker images should be small.** The aim of the game is to only ship production images with the stuff needed to run your application in production. The way you write your Dockerfiles has a huge impact on the size of your images.

A common example is that every `RUN` instruction adds a new layer. As a result, it’s usually considered a best practice to include multiple commands as part of a single `RUN` instruction — all glued together with double-ampersands (`&&`) and backslash (`\`) line-breaks. Another issue is that we don’t clean up after ourselves. We’ll `RUN` a command against an image that pulls some build-time tools, and we’ll leave all those tools in the image when we ship it to production.

**Multi-stage builds** are all about optimizing builds without adding complexity.

Multi-stage builds have a single Dockerfile containing multiple FROM instructions. Each FROM instruction is a new build stage that can easily COPY artifacts from previous stages.

> **An important thing to note, is that COPY --from instructions are used to only copy production-related application code from the images built by the previous stages. They do not copy build artifacts that are not needed for production.**

Commands: `docker image build -f`, `FROM`, `RUN`, `COPY`, `EXPOSE`, `ENTRYPOINT`, `LABEL`, `ENV`, `ONBUILD`, `HEALTHCHECK,`, `CMD`, ... and more. [DOCKERFILE_REFERENCE](https://docs.docker.com/reference/dockerfile/)

## Volumes and Persistent Data

There are two main categories of data — persistent and non-persistent. Persistent is the data you need to keep. Things like; customer records, financial data, research results, audit logs, and even some types of application log data. Non-persistent is the data you don’t need to keep. Stateful applications that persist data are becoming more and more important in the world of cloud-native and microservices applications.

Every Docker container gets its own non-persistent storage. This is automatically created for every container and is tightly coupled to the lifecycle of the container. As a result, deleting the container will delete the storage and any data on it.

To deal with persistent data, a container needs to store it in a volume. Volumes are separate objects that have their lifecycles decoupled from containers. This means you can create and manage volumes independently, and they’re not tied to the lifecycle of any container. Net result, you can delete a container that’s using a volume, and the volume won’t be deleted.

Every Docker container is created by adding a thin read-write layer on top of the read-only image it’s based on. This thin writable layer is an integral part of a container and enables all read/write operations. If you, or an application, update files or add new files, they’ll be written to this layer. The writable container layer exists in the filesystem of the Docker host, and you’ll hear it called various names. These include local storage, ephemeral storage, and graphdriver storage.

Volumes are the recommended way to persist data in containers. There are three major reasons for this:

1. Volumes are independent objects that are not tied to the lifecycle of a container
2. Volumes can be mapped to specialized external storage systems
3. Volumes enable multiple containers on different Docker hosts to access and share the same data

At a high-level, you create a volume, then you create a container and mount the volume into it. The volume is mounted into a directory in the container’s filesystem, and anything written to that directory is stored in the volume. If you delete the container, the volume and its data will still exist. Volumes are first-class citizens in Docker. Among other things, this means they are their own object in the API and have their own docker volume sub-command.

By default, Docker creates new volumes with the built-in local driver. As the name suggests, volumes created with the local driver are only available to containers on the same node as the volume. You can use the -d flag to specify a different driver. Third-party volume drivers are available as plugins. These provide Docker with seamless access to external storage systems such as cloud storage services and on-premises storage systems.

> All volumes created with the local driver get their own directory under /var/lib/docker/volumes on Linux, and C:\ProgramData\Docker\volumes on Windows.

Integrating external storage systems with Docker makes it possible to share volumes between cluster nodes. These external systems can be cloud storage services or enterprise storage systems in your on-premises data centers. As an example, a single storage LUN or NFS share can be presented to multiple Docker hosts, allowing it to be used by containers and service replicas no-matter which Docker host they’re running on.

> Building a setup like this requires a lot of things. You need access to a specialized storage systems and knowledge of how it works and presents storage. You also need to know how your applications read and write data to the shared storage. Finally, you need a volumes driver plugin that works with the external storage system. Docker Hub is the best place to find volume plugins. Login to Docker Hub, select the view to show plugins instead of containers, and filter results to only show Volume plugins. Once you’ve located the appropriate plugin for your storage system, you create any configuration files it might need, and install it with docker plugin install.

**A major concern with any configuration that shares a single volume among multiple containers is data corruption.**

Commands: `docker volume create`, `docker volume ls`, `docker volume inspect`, `docker volume prune`, `docker volume rm`, `docker plugin install`, `docker plugin ls`

## Docker Networking

Docker networking is based on an open-source pluggable architecture called the **Container Network Model (CNM)**. `libnetwork` is Docker’s real-world implementation of the CNM, and it provides all of Docker’s core networking capabilities. **Drivers** plug in to `libnetwork` to provide specific network topologies. Docker ships with a set of native drivers that deal with the most common networking requirements. These include single-host bridge networks, multi-host overlays, and options for plugging into existing VLANs.

The design guide for Docker networking is the CNM. It outlines the fundamental building blocks of a Docker network, it defines three major building blocks:

1. **Sandboxes**: A isolated network stack. It includes; Ethernet interfaces, ports, routing tables, and DNS config.
2. **Endpoints**: Virtual network interfaces (E.g. veth). Like normal network interfaces, they’re responsible for making connections. In the case of the CNM, it’s the job of the endpoint to connect a sandbox to a network. Endpoints behave like regular network adapters, meaning they can only be connected to a single network. Therefore, if a container needs connecting to multiple networks, it will need multiple endpoints.
3. **Networks**: Software implementation of an switch (802.1d bridge). As such, they group together and isolate a collection of endpoints that need to communicate.

Nowadays, all of the core Docker networking code lives in `libnetwork`. It implements all three of the components defined in the CNM. It also implements native service discovery, ingress-based container load balancing, and the network control plane and management plane functionality. In order to meet the demands of complex highly-fluid environments, `libnetwork` allows multiple network drivers to be active at the same time. This means your Docker environment can sport a wide range of heterogeneous networks.

> If you’re unsure about terms such as control plane and data plane, control plane traffic is cluster management traffic, whereas data plane traffic is application traffic.

The simplest type of Docker network is the `single-host` `bridge` network.

- **Single-host** tells us it only exists on a single Docker host and can only connect containers that are on the same host.
- **Bridge** tells us that it’s an implementation of an 802.1d bridge.

> Every Docker host gets a default single-host bridge network. By default, this is the network that all new containers will be connected to unless you override it on the command line with the --network flag.

So far, we’ve said that containers on bridge networks can only communicate with other containers on the same network. However, you can get around this using port mappings. Port mappings let you map a container to a port on the Docker host. Any traffic hitting the Docker host on the configured port will be directed to the container. Mapping ports like this works, but it’s clunky and doesn’t scale. For example, only a single container can bind to any port on the host. This means no other containers on that host will be able to bind to that port. This is one of the reason’s that single-host bridge networks are only useful for local development and very small applications.

Commands: `docker network ls`, `docker network create`, `docker network create -d overlay <network_name>`, `docker network inspect`, `docker network prune`, `docker network rm`

### Overlay Networking

It’s vital that containers can communicate with each other reliably and securely, even when they’re on different hosts that are on different networks. This is where overlay networking comes in to play, overlay networks are multi-host. They allow you to create a flat, secure, layer-2 network, spanning multiple hosts. Containers connect to this and can communicate directly. Overlay networks are ideal for container-to-container communication, including container-only applications, and they scale well. By default, Docker overlay networks encrypt cluster management traffic but not application traffic. You must explicitly enable encryption of application traffic.

Docker provides a native driver for overlay networks. This makes creating them as simple as adding the --d overlay flag to the docker network create command.

## Some Best Practices For Docker

Some Best Practices:

1. Leverage the build cache - The build process used by Docker has the concept of a cache that it uses to speed-up the build process. The best way to see the impact of the cache is to build a new image on a clean Docker host, then repeat the same build immediately after. The first build will pull images and take time building layers. The second build will complete almost instantaneously. This is because artifacts from the first build, such as layers, are cached and leveraged by later builds.

2. Use no-install-recommends - If you are building Linux images, and using the apt package manager, you should use the no-install-recommends flag with the apt-get install command. This makes sure that apt only installs main dependencies (packages in the Depends field) and not recommended or suggested packages. This can greatly reduce the number of unwanted packages that are downloaded into your images.

Source: _Poulton, Nigel. Docker Deep Dive: Zero to Docker in a single book_
