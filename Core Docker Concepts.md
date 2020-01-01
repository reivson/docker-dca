Core Docker Concepts
========================
## Installing Docker CE(CentOS)

To install Docker CE on CentOS, we will need to do the following:

* **Provision a Server**
	* Image: CentOS 7
	* Size: Small

* **Install Required Packages:** Install some required packages(yum-utils, device-mapper-persistent-data, and lvm2)
* **Add the Docker Repo**
* **Install Docker and Containerd packages**
* **Start and enable the Docker Service**
* **Configure cloud_userto be Able to Use Docker:** add cloud_user to the docker group, the log out and back in.
* **Run a Container to Test the Installation:** Test the installation by running the hello-world image

```
$ sudo yum install -y device-mapper-persistent-data lvm2

$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

$ sudo yum install -y docker-ce-18.09.5 docker-ce-cli-18.09.5 containerd.io   

$ sudo systemctl start docker

$ sudo systemctl enable docker

$ sudo usermod -a -G docker cloud_user

$ docker run hello-world
```

To install Docker CE on Ubuntu, we will need to do the following:
Apenas os comandos:

```
$ sudo apt-get update

$ sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

$ sudo apt-key fingerprint 0EBFCD88

$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

$ sudo apt-get update

$ sudo apt-get install docker-ce docker-ce-cli containerd.io

$ sudo apt-get install -y docker-ce=5:18.09.5~3-0~ubuntu-bionic docker-ce-cli=5:18.09.5~3-0~ubuntu-bionic containerd.io

$ sudo usermod -a -G docker cloud_user

$ docker run hello-world
```

## Upgrading Docker Engine

Since we installed Docker using the package repositories, upgrading is simple is simple.

Just install a newer package version!

**Downgrade to a previous version:**

```
$ sudo systemctl stop docker

$ sudo apt-get remove -y docker-ce docker-ce-cli

$ sudo apt-get update

$ sudo apt-get install -y docker-ce=5:18.09.4~3-0~ubuntu-bionic docker-ce-cli=5:18.09.4~3-0~ubuntu-bionic

$ docker version
```

**Upgrade to a new version:**

```
$ sudo apt-get install -y docker-ce=5:18.09.5~3-0~ubuntu-bionic docker-ce-cli=5:18.09.5~3-0~ubuntu-bionic

$ docker version
```

## Running a Container

Now that we know how to install and configure Docker, we're ready to run some containers!

We  can run containers using the `docker run` command.

` # docker run [OPTION] IMAGE[:TAG]  [COMMAND]  [ARG]`

- IMAGE: The container image to run. By default, these are pulled from Docker Hub.
- TAG: A specific image tag. Usually used to pull a specific version.
- COMMAND: Command to run inside the container.
- ARG: Arguments to pass when runnig the command.

Some commonly-used options:

- **-d**: Run container in detached mode. The `docker run` command will exit immediately and the container will run in the background.
- **--name**: A container is assigned a random name by default, but you can give a more descriptive name with this flag.
- **--restart**: Specify when the container should be automatically restarted.
	- **no** (default): Never restart the container.
	- **on-failure**: Only if the the container fails (exits with a non-zero exit code).
	- **always**: Always restart the container whether it succeeds or fails. Also starts the container automatically on daemon startup.
	- **unless-stopped**: Always restart the container wheter it succeeds or fails, and on daemon startup, *unless* the container was manually stopped.

- **-p <host port>:<container port>**: Expose a container's port by mapping it to a port on the host. You can use **-p** multiple times to map multiple ports.
- **--rm**: Automatically remove the container when it exits. Cannot be used with **--restart**.
- **--memory**: hard limit on memory usage.
- **--memory-reservation**: A soft limit on memory usage. The container will be restricted to this limit if Docker detects memory contention(not enough memory on the host).

**Manage Containers**

docker ps: List currently running containers. Use docker ps -a to see all containers, including stopped containers.

**docker container stop <container name>**: Stop a running container.

**docker container start <container name>**: Start a stopped container.

**docker container rm <container name>**: Delete a container(It must be stopped first).

## Selecting a Storage Driver

**Storage Drivers** provide a pluggable framework for managing the temporary, internal storage of a container's writable layer.

Docker supports a variety of storage drivers. The best storage driver to use depends on your enviroment and your storage needs.

**overlay2:** File-based storage. Default for Ubuntu and CentOS 8+.

**devicemapper:** Block storage, more efficient for doing lots of writes. Default for CentOS 7 and earlier.

You can find out what storage driver is currently configured with `docker info`

docker automatically selects a default storage driver that is compatible with your environment.

However, in some cases you may want to **override the default** to us a different driver.

The are two ways to do this:

+ Set the *--storage-drive* flag when starting Docker (in your system unit file for example).
+ Set the *"storage-driver"* value in /etc/socker/daemon.json.

Get the current storage driver:
`$ docker info`

Set the storage driver explicitly by providing a flag to the Docker daemon:
`$ sudo vi /usr/lib/systemd/system/docker.service`

Edit the *ExecStart* line, adding the *--storage-driver devicemapper* flag:
*ExecStart=/usr/bin/dockerd --storage-driver devicemapper ...*

After any edits to the unit file, reload Systemd and restart Docker:

```
$ sudo systemctl daemon-reload

$ sudo systemctl restart docker
```

We can also set the storage driver explicitly using the daemon configuration file. This is the method that Docker recommends. Note that we cannot do this and pass the *--storage-driver* flag to the daemon at the same time:

`$ sudo vi /etc/docker/daemon.json`

Set the storage driver in the daemon configuration file:

```
{
  "storage-driver": "devicemapper"
}
```
Restart Docker after editing the file. It is also a good idea to make sure Docker is running properly after changing the configuration file:

`$ sudo systemctl restart docker`

`$ sudo systemctl status docker`


## Configuring Logging Drivers(SPLUNK, Journald, etc.)

**Login Drivers** are a pluggable framework for accessing log data from services and containers in Docker. Docker supports a variety of logging drivers.

configure the default logging driver by setting `log-driver` and `log-opts` in `/etc/docker/daemon.json `

`$ sudo docker info | grep "Logging"`
`$ vim /etc/docker/daemon.json `

{
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "15m"
		}
}

`$ sudo systemctl restart docker`


You override the default logging driver and options for a container with the `--log-driver` and `--log-opt` flags when using `docker run`

`$ sudo docker container run --log-driver syslog nginx`
`$ sudo docker container run --log-driver json-file --log-opt max-size=50m nginx`


## Docker Swarm

Docker includes a feature called swarm mode, which allows you to build a distributed cluster of docker machines to run your containers.

Docker Swarm provides many useful features, and can help facilitate orchestration, high-availability, and scaling. 

**Configuring a Swarm Manager**

Setting up a new swarm is relatively simple. All we have to do is create the first swarm manager:

- Install Docker CE on the Swarm Manager server.
- Initialize the swarm with `docker swarm init`

Once the swarm is initialized, you can see some info about the swarm with `docker info`.

You can list the current nodes in the `swarm with docker node ls`.

**Configuring Swarm Nodes**

With a manager set up, we can add some worker nodes to the swarm.

- Install Docker CE on both worker nodes.
- Get a join command from the manager: Run `docker swarm join-token worker` on the manager node to get a join command.
- Run the join command on both workers: copy the join command from the manager and run it on both workers.
- Verify both workers have successfully joined the swarm: Run `docker node ls` in the manager and verify that you can see the two worker nodes listed


**Swarm Backup and Restore**

If you are managing a swarm cluster, it is important to be able to backup current swarm data and restore a previous backup.


**Create the Backup**

On the manager:

```
$ sudo systemctl stop docker
$ sudo tar -zvcf backup.tar.gz /var/lib/docker/swarm
$ sudo systemctl start docker
```

**Restore from the Backup**

On the manager:

```
$ sudo systemctl stop docker
$ sudo rm -rf /var/lib/docker/swarm/*
$ sudo tar -zxvf backup.tar.gz -C /var/lib/docker/swarm/
$ sudo systemctl start docker
$ docker node ls
```

**Namespaces**

Namespaces are a linux technology that allows processes to be isolated in terms of the resources that they see. They can be used to prevent different processes from interfering or interacting with one another.

Docker uses namespaces to isolate containers. This technology allows containers to operate independently and securely.

Docker uses namespaces such as the following to isolate resources for containers:

- pid: Process isolation
- net: Network interfaces
- ipc: Inter-process communication
- mnt: Filesystem mounts
- uts: Kernel and version identifiers
- user namespaces: Requires special configuration. Allows container processes to run as root inside the container while mapping that user to an unprivileged user on the host.
