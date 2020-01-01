Storage and Volumes
========================

## Docker Storage in Depth


**Storage Drivers**

Storage drivers are sometimes known as Graph drivers. The proper storage driver to use often depends on your operating system and other local configuration factors.

**overlay2**: Current Ubuntu and CentOS/RHEL version.

**aufs**: Ubuntu 14.04 and older.

**devicemapper**: CentOS 7 and earlier


**Storage Models**

Persistent data can be managed using several storage models.

- Filesystem Storage:
    - Data is stored in the form of a file system.
    - Used by **overlay2** and **aufs**.
    - Efficient use of memory.
    - Inefficient with write-heavy workloads.
- Block Storage:
    - Stores data in blocks.
    - Used by **devicemapper**.
    - Efficient with write-heavy workloads.
- Object Storage:
    - Stores data in an external object-based store.
    - Application must be designed to use object-based storage.
    - Flexible and scalable.

**Storage Layers**

Docker storage consists of layers.

Both containers and images have layers. You can find the location of the layered data on disk using docker inspect.

## Configuring DeviceMapper

Device Mapper is one of the Docker storage drivers available for some Linux distributions. It is the default storage driver for CentOS 7 and earlier.

You can customize your Device Mapper configuration using the **daemon config file**.

DeviceMapper supports two modules:

- loop-lvm mode:
    - Loopback mechanism simulates an additional physical disk using files on the local disk.
    - Minimal setup, does not require an additional storage device.
    - Bad performance, only use for testing
- direct-lvm:
    - Stores data on a separate device.
    - Requires an additional storage device.
    - Good performance, use for production.

## Docker Volumes

**Bind Mounts vs. Volumes**

When mounting external storage to a container, you can use either a **bind mount** or a **volume**.

- Bind mounts
    - Mount a specific path on the host machine to the container.
    - Not portable, dependent on the host machine's file system and directory structure.
- Volumes
    - Stores data on the host file system, but the storage location is managed by Docker.
    - More portable.
    - Can mount the same volume to multiple containers.
    - Work in more scenarios.

**Working With Volumes**

--mount syntax:

`$ docker run --mount [key=value][,key=value..]`

- **type**: bind(bind mount), volume, or tmpfs(temporary in-memory storage).
- **source, src**: Volume name or bind mount path.
- **destination, dst, target**: Path to mount inside the container.
- **readonly**: Make the volume or bind mount read-only

```
$ cd ~/
$ mkdir message
$ echo Hello, world! > message/message.txt
```

```
$ docker run --mount type=bind,source=/home/cloud_user/message,destination=/root,readonly busybox cat /root/message.txt

$ docker run --mount type=volume,source=my-volume,destination=/root busybox sh -c 'echo hello > /root/message.txt && cat /root/message.txt'
```
-v syntax:

`$ docker run -v SOURCE:DESTINATION[:OPTIONS]`

- **SOURCE**: if this is a volume name, it will create a **volume**. If this is a path, it will create a **bind mount**.
- **DESTINATION**: Location to mount the data inside the container.
- **OPTIONS**: Comma-separated list of options. For example, **ro** for read-only.

```
$ docker run -v /home/cloud_user/message:/root:ro busybox cat /root/message.txt

$ docker run -v my-volume:/root busybox sh -c 'cat /root/message.txt'
```

Use Volume to share data between containers.

```
$ docker run --mount type=volume,source=shared-volume,destination=/root busybox sh -c 'echo I wrote this! > /root/message.txt'

$ docker run --mount type=volume,source=shared-volume,destination=/anotherplace busybox cat /anotherplace/message.txt
```
Create and manage volumes using `docker volume` commands.

```
$ docker volume create test-volume
$ docker volume ls
$ docker volume inspect test-volume
$ docker volume rm test-volume
```

## Image Cleanup

- Display the storage space being used by Docker.

    `$ docker system df`

- Display disk usage by individual objects.

    `$ docker system df -v`

- Delete dangling images (images with no tags or containers).

    `$ docker image prune`

- Pull an image not being used by any containers, and use `docker image prune -a `to clean up all images with no containers.

```
    $ docker image pull nginx:1.14.0
    $ docker image prune -a
```

## Storage in a Cluster

When working with multiple Docker machines, such as a swarm cluster, you may need to dhare Docker volume storage between those machines.

Some options:

- Use application logic to store data in external object storage.
- Use a volume driver to create a volume that is external to any specific machine in your cluster.


Install the `vieux/sshfs` plugin on all nodes in the swarm.

    `$ docker plugin install --grant-all-permissions vieux/sshfs`

Set up an additional server to use for storage. You can use the **Ubuntu 18.04 Bionic Beaver LTS** image with a size of Small. On this new storage server, create a directory with a file that can be used for testing.

```
$ mkdir /home/cloud_user/external
$ echo External storage! > /home/cloud_user/external/message.txt
```
On the swarm manager, manually create a Docker volume that uses the external storage server for storage. Be sure to replace the text **<STORAGE_SERVER_PRIVATE_IP>** and **<PASSWORD>** with actual values.

```
$ docker volume create --driver vieux/sshfs \
  -o sshcmd=cloud_user@172.31.34.192:/home/cloud_user/external \
  -o password=uihkbKUVKJefrBK \
  sshvolume
  
$  docker volume ls
```
Create a service that automatically manages the shared volume, creating the volume on swarm nodes as needed. Be sure to replace the text **<STORAGE_SERVER_PRIVATE_IP>** and **<PASSWORD>** with actual values.

```
$ docker service create \
  --replicas=3 \
  --name storage-service \
  --mount volume-driver=vieux/sshfs,source=cluster-volume,destination=/app,volume-opt=sshcmd=cloud_user@172.31.34.192:/home/cloud_user/external,volume-opt=password=uihkbKUVKJefrBK busybox cat /app/message.txt
```
Check the service logs to verify that the service is reading the test data from the external storage server.

`$ docker service logs storage-service`