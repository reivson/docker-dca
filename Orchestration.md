Orchestration
========================

## Locking and Unlocking a Swarm Cluster

**Autolock**

Docker swarm encrypts sensitive data for security reasons, such as:

- Raft logs on swarm managers.
- TLS communication between swarm nodes.

By default, Docker manages the keys used for this encryption automatically, but they are stored unencrypted on the managers disks.

Autolock is a feature that automatically locks the swarm, allowing you to manage the encryption keys yourself. This gives you control of the keys and can allow for greater security.

However, it requires you to unlock the swarm every time Docker is restarted on one of your managers.

You can enable autolock when you initialize a new swarm with the --autolock flag.

`$ docker swarm init --autolock`

Enable autolock on a running swarm

`$ docker swarm update --autolock=true`

Disable autolock on a running swarm

`$ docker swarm update --autolock=false`

Whenever Docker restart on a manager, you must **unlock the swarm**:

`$ docker swarm unlock`

Get the current unlock key for a running swarm:

`$ docker swarm unlock-key`

Rotate the unlock key:

`$ docker swarm unlock-key --rotate`


## High Availability in a Swarm Cluster

**Multiple Managers**

In order to build a highly-available and fault-tolerant Swarm, It is a good idea to have multiple swarm managers.

Docker uses the **Raft consensus algorithm** to maintain a consistent cluster state across multiple managers.

More manager nodes means better fault tolerance. However, there can be a decrease in performance as the number of managers grows, since more managers means more network traffic as managers agree to updates in the cluster state.

**Quorum**

A **Quorum** is the majority(more than half) of the managers in a swarm. For example, for a swarm with 5 managers, the qui=orum is 3.

A quorum must be maintained in order to make changes to the cluster state. If a quorum is not available, nodes cannot be added, and existing tasks cannot be changed or moved.

Note that since a quorum requires more than half of the manager nodes, it is recommended to have aan odd number of managers.

**Availability Zones**

Docker recommends that you distribute your manager nodes across at least 3 availability zones.

Distribute your managers across these zones so that you can maintain a quorum if one of them goes down.

## Introdution to Docker Services

**Services**

A **Service** is used to run an application on a Docker Swarm. A service specifies a set of one or more replica **tasks**. These tasks will be distributed automatically across the nodes in the cluster executed as **containers**.

**--replicas**: Specify the number of replica tasks to create for the service.

**--name**: Specify a name for the service.

**-p PUBLISHED PORT:SERVICE PORT**: Publish a port so the service can be accessed externally. The port is published on every node in the swarm.

`$ docker service create [OPTIONS] IMAGE ARGS`

1. Create a simple service running the nginx image.

`$ docker service create nginx`

2. Create an nginx service with a specified name, multiple replicas, and a published port

`$ docker service create --name nginx --replicas 3 -p 8080:80 nginx`


Templates can be used to give somewhat dynamic values to some flags with docker service create.

The following flags accept templates:

- --hostname
- --mount
- --env

This command sets an environment variable for each container that contains the hostname of the node that container is running on:

`$ docker service create -env NODE_HOSTNAME="{{.Node.Hostname}}" nginx` 

List current services:

`$ docker service ls`

List services tasks:

`$ docker service ps SERVICE`

Get more information about a service:

`$ docker service inspect SERVICE`

Make changes to a service:

`$ docker service update [OPTION] SERVICE`

Delete an existing service:

`$ docker service rm SERVICE`


**Replicated vs. Global Services**

**Replicated Services** run the requested number of replica tasks across the swarm cluster.

`$ docker service create --replicas 3 nginx`

**Global services** run one task on each node in the cluster.

`$ docker service create --mode global nginx`

**Scaling Services**

Scaling Services means changing the number of replica tasks.

There are two ways to scale a service. Update the service with a new number of replicas.

`$ docker service update --replicas REPLICAS SERVICE`

Use `docker service scale`.

`$ docker service scale SERVICE=REPLICAS`


## Using Docker inspect

Docker inspect is a command that allows you to get information about Docker objects, such as containers, images, services, etc.

`$ docker inspect OBJECT_ID`

if you know what kind of object you are inspecting, you can also use an alternate form of the command:

```
$ docker container inspect CONTAINER
$ docker service inspect SERVICE
```
This form allows you to specify an object name instead of an ID.

For some objects types, you can also supply the --prety flag to get a more readable output.

**Docker inspect --format**

Use the --format flag to retrieve a specific subsection of the data using a Go template.

$ docker service inspect --format='{{.ID}}' SERVICE 


## Docker Compose

Docker Compose is a tool that allows you to run multi-container applications defined using a declarative format.

**Note**: Docker Certified Associate primarily covers Docker compose as it relates to **stacks** in Docker swarm.

First, let's install.

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose version
```
Docker compose uses YAML files to declaratively defines a set of containers and other resources that will becreated as part of the larger application.

Setting up a new Docker compose project:

```
$ mkdir nginx-compose && cd nginx-compose
$ vi docker-compose.yml
```
```
version: '3'
services:
  web:
    image: nginx
    ports:
    - "8080:80"
  redis:
    image: redis:alpine
```

```
$ docker-compose up -d
$ docker-compose ps
$ docker-compose down
```
## Introdution to Docker Stacks

**Docker Stacks**

Services are capable of running a single, replicated application across nodes in the cluster, but what if you need to deploy a more complex application consisting of multiple services?

A **Stack** is a collection of interrelated services that can be deployed and scaled as a unit.

**Docker Stacks** are similar to the multi-container applications created using **Docker Compose**. howerver, they vaben scaled and executed across he swarm just like normal swarm services.

## Nodes Labels

You can add pieces of metadata to your swarm nodes using node labels.

You can then use these labels to determine which nodes tasks will run on.

Example use case:

- With nodes in multiple data centers or availability zones, you can use labels to specify which zone each node is in. Then, execute them evenly across zones.

```
 $ docker node update --label-add LABEL=VALUE NODE
 $ docker node update --label-add availability_zone=east <NODE_NAME>
```
add a label to a node

You can view existing node labels with:

`$ docker node inspect --pretty NODE`

**Node Constraints**

To run a service's tasks only on nodes with a specific label value, use the --constraint flag with docker service create.

`$ docker service create --constraint node.labels.LABEL==VALUE IMAGE`

You can also use a constraint to run only on nodes without a particular value.

`$ docker service create --constraint node.labels.LABEL!=VALUE IMAGE`

You can use the `--constraint` flag multiple times to list multiple constraints. `All` constraints must be satisfied for tasks to run on a node.

**Placement-pref**

Use --placement-pref with the spread strategy to spread tasks evenly across all values of a particularlabel.

`$ docker service create --placemet-pref spread=node.labels.LABEL IMAGE`

For example, if you have a label called `availability_zone` with three values(east, west, and south), the tasks will be divided avenly among the node groups with each of those three values, no matter how many nodes are in each group.





