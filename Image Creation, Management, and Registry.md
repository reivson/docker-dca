Image Creation, Management, and Registry
========================

## Introduction to Docker Images


A **Docker image** is a file containing the code and components needed to run software in a container.

Containers and images use a **layered file system**. Each layer contain only the differences from the previous layer.

The **image** consists of one or more read-only layers, while the **container** adds one addition writeble layer.

The layered file system allows multiple images and contaners to share the same layers. This results in:

- Smaller overall storage footprint
- Faster image transfer.
- Faster image build.


Run a container. The image will be automatically downloaded if it does not exist on the system:

`$ docker run nginx:1.15.8`


Download an image:

`$ docker image pull nginx`


View file system layers in an image:

`$ docker image history nginx`


## Dockerfiles

If you want to create your own images, you can do so with a **Dockerfile**.

A **Dockerfile** is a set of instructions which are used to construct a Docker  image. These instructions are called **directives**.

**FROM**: Starts a new build stage and sets the base image. Usually must be the first directive in the Dockerfile(except ARG can be placed before **from**).

**ENV**: Set environment variables. These can be referenced in the DOckerfile itself and are visible to the container at runtime.

**RUN**: Creates a new layer on top the previous layer by running a command inside that new layer and committing the changes.

**CMD**:  Specify a default command used to tun a container at execution time.


Build and test the image:

```
$ docker build -t custom-nginx .
$ docker run --name custom-nginx -d -p 8080:80 custom-nginx
$ curl localhost:8080
```

**EXPOSE**: Documents which port(s) are intended to publish when running a container.

**WORKDIR**: Sets the current working directory for subsequent directives such as **ADD, COPY, CMD, ENTRYPOINT**, etc. Can be used mutiple times to change directories throughout the Dockerfile. You can also use a relative path, which sets the new working direcotry relative to the previous working directory.

**COPY**: Copy files from the local machine to the image.

**ADD**: Similar to **COPY**, but can also pull files using a URL and extract an archive into loose files in the image.

**STOPSIGNAL**: Specify the signal that will be used to stop the container.

**HEALTHCHECK**: Specify a command to run in order to perform a custom health check to verify that container is working properly.


**Efficient Docker Images**

When working with Docker in the real world, it is important to create Docker images that are as efficient as possible.

This means that they are **as small possible** and result in **ephemeral containers** that can be started, stopped, and destroyed easily.

General tips:

- Put things that are less likely to change on lower-level layers.
- Don't create unnecessary layers.
- Avoind including any unnecessary files, packages, etc. in the image.


## Multi-Stage Builds

Docker supports the ability to perform **multi-stage buils**. Multi-stage builds have more than one **FROM** directive in the Dockerfile, with each **FROM** directive starting a new stage.

Each stage begins a completely new set of sile system layers, allowing you to **selectively copy** only the files you need from previous layers.

Use the **--from** flag with **COPY** to copy files from a previous stage.

`COPY --from=0 ...`

You can also name your stages with **FROM... AS**,  then reference the name with **COPY --from**:

```
FROM <image> AS stage1
...
FROM <image> AS stage2
COPY --FROM=stage1 ...
```

**Flattening an Image**

Sometimes, images with fewer layers can perform better. In a few cases, you may want to take an image with many layers and flatten them into a single layer.

Docker does not provide an official method for doing this, but you can accomplish it by doing the following:

- Run a container from the image.
- Export the container to an archive using docker export.
- Import the archive as a new image using docker import.

The resulting image will have only one layer!
 
Create a Dockerfile that will result in a multi-layered image:

```
FROM alpine:3.9.3
RUN echo "Hello, World!" > message.txt
CMD cat message.txt
```
Build the image and check how many layers it has:

```
$ docker build -t nonflat .
$ docker image history nonflat
```
Run a container from the image and export its file system to an archive:

```
$ docker run -d --name flat_container nonflat
$ docker export flat_container > flat.tar
```

Import the archive to a new image and check how many layers the new image has:

```
$ cat flat.tar | docker import - flat:latest
$ docker image history flat
```
## Docker Registries

A **Docker Registry** is responsible for storing and distributing Docker images.

We have already pulled images from the default public registry, **Docker Hub**.

You can also create your own registries using Docker's open source registry software, or **Docker Trusted Registry**, the non-free enterprise solution.

To create a basic registry, simply run a container using the **registry** image and publish port **5000**.

Registry Server:

- **Distribution** - Ubuntu 18.04 Bionic Beaver LTS
- **Size** - Small

**Configuring a Registry**

You can override individual values in the default registry configuration by supplying environment variables with `docker run -e`.

Name the variable REGISTRY_ followed by each configuration key, all uppercase and separated by underscores.

For example, to change the config:

log:
    level: info
    
    
Set environment variable:

`REGISTRY_LOG_LEVEL=debug`

**Securing a Registry**

By default, the registry is completely unsecured.It does not use TLS and does not require authentication.

You can take some basic steps to secure your registry:

- Use TLS with a certification.
- Require user authentication.


**Using Docker Registries**

Authenticate with a remote registry

`$ docker login REGISTRY`

When working with Docker registries, if the registry is not specified, the default registry will be used(**Docker Hub**).

There are multiple ways to use a registry with a self-signed certificate.

- Turn off certificate verification(very insecure).
- Provide the public certificate to the Docker engine.

To push and pull images from your private registry, tag the images with the registry hostname(and optionally, port).


```
<registry public hostname>/<image name>:<tag>
```


Upload the image to a remote registry

`$ docker pull image`

