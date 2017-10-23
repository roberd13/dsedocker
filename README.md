# Contents
* [DataStax Platform Overview](#datastax-platform-overview)
* [Getting Started with DataStax and Docker](#getting-started-with-datastax-and-docker)
* [Prerequisites](#prerequisites)
* [Configuration Management](#configuration-management)
* [Configuration with Environment Variables](#configuration-with-environment-variables)
* [Database and Data Storage](#database-and-data-storage)
* [Exposing Ports on the Docker Host](#exposing-ports-on-the-docker-host)
* [Starting DSE nodes](#starting-dse-nodes)
* [Container shell access and viewing Cassandra logs](#container-shell-access-and-viewing-cassandra-logs)
* [Starting DSE Tools](#starting-dse-tools)
* [Starting an OpsCenter node](#starting-an-opscenter-node)
* [Starting a Studio container](#starting-a-studio-container)
* [Using Docker Compose for Automated Provisioning](#using-docker-compose-for-automated-provisioning)
* [Building](#building)
* [Getting Help](#getting-help)
* [Licensing](#licensing)


# DataStax Platform Overview

Built on the 
best distribution of Apache Cassandra™, DataStax Enterprise is the always-on 
data platform designed to allow you to effortlessly build and scale your apps, 
integrating graph, search, analytics, administration, developer tooling, 
and monitoring into a single unified platform. We power your 
apps' real-time moments so you can create instant insights 
and powerful customer experiences.

![](https://upload.wikimedia.org/wikipedia/commons/e/e5/DataStax_Logo.png)


# Getting Started with DataStax and Docker

The DataStax provided docker images are intended to be used for Development purposes in non-production environments. You can use these images to learn DSE, OpsCenter and DataStax Studio, to try new ideas, and to test and demonstrate your application.


# Prerequisites

Install Docker based on instructions found [here](https://docs.docker.com/engine/installation/) for your environment . 
Understanding of Docker and containers.

If you're building custom images based on the github repo, you'll need a DataStax Academy Account. To sign up for a [DataStax Academy account](https://academy.datastax.com/). You then pass your DataStax Academy credentials to the build script.  See the [Building](#building) section for more detailed information.


# Configuration Management

Users have four options to manage configurations. Two will be documented here. 

The first which we call the Single Mount Option is a simple mechanism to let users change or modify configurations without replacing or customizing the containers. Users can add any of the approved config files to a mounted host volume and we’ll handle the hard work of mapping them within the container.

The Second is by providing environment variables at runtime. The other two mechanisms; Docker file/directory volume mounts and overlay file systems are well documented by Docker. 

## Single Mount Configuration Management

DataStax has made it easy to make configuration changes by creating a script that looks in the exposed Volume `/conf` for any added configuration files and loads them at container start. 

To take advantage of this feature, you will need to create a mount directory on your host, mount your local directory to the exposed Volume `/conf`, place you modified configuration files in the mount directory on your host machine and start the container. 

These files will override the existing configuration files.  The configs must contain all the values to be used along with using the dse naming convention such as cassandra.yaml, dse.yaml, opscenterd.conf 

For a full list of configuration files please visit *some link here*

```
docker run -e DS_LICENSE=accept --name my-dse -d  -v /dse/conf:/conf datastax/dse-server:5.1.4
```

## Configuration with Environment Variables


When you start the DSE image, you can adjust the configuration of the Cassandra instance by passing one or more environment variables on the `docker run` command line

## Required

### EULA Acceptance 

In order to use these images, it is necessary to accept the terms of the DataStax license. This is done by setting the environment variable `DS_LICENSE` to the value accept when running containers based on the produced images. To show the license included in the images, set the variable `DS_LICENSE` to the value `accept`. *The images will not start without the variable set to the accept value.*

## Optional

### LISTEN_ADDRESS 
The IP address to listen for connections from other nodes. Defaults to the container's IP address.

### BROADCAST_ADDRESS
The IP address to advertise to other nodes. Defaults to the same value as the `LISTEN_ADDRESS`.

### RPC_ADDRESS 
The IP address to listen for client/driver connections. Defaults to 0.0.0.0 (i.e. wildcard).

### BROADCAST_RPC_ADDRESS 
The IP address to advertise to clients/drivers. Defaults to the same value as the `BROADCAST_ADDRESS`.

### SEEDS 
The comma-delimited list of seed nodes for the cluster. Defaults to this node's `BROADCAST_ADDRESS` if not set and will only be set the first time the node is started.

### START_RPC
Whether to start the Thrift RPC server. Will leave the default in the `cassandra.yaml` file if not set.

### CLUSTER_NAME
The name of the cluster. Will leave the default in the `cassandra.yaml` file if not set.

### NUM_TOKENS
The number of tokens randomly assigned to this node. Will leave the default in the `cassandra.yaml` file if not set.

### DC
Datacenter name, default: Cassandra

### RACK
Rack name, default: rack1

### OPSCENTER_IP
Optional, the address of OpsCenter instance we would like to use for DSE management it can be specified via linking the OpsCenter container using opscenter as its name.

 # Database and Data Storage

The following volumes are created and exposed with the images:  

**DSE**

* `/var/lib/cassandra`: Data from Cassandra
* `/var/lib/spark`: Data from DSE Analytics w/ Spark
* `/var/lib/dsefs`: Data from DSEFS
* `/var/log/cassandra`: Logs from Cassandra
* `/var/log/spark`: Logs from Spark
* `/conf`: Directory to add custom config files for the container to pickup.

**OpsCenter**

* `/var/lib/opscenter`: OpsCenter data

**Studio**

* `/var/lib/datastax-studio`: Studio Data

To persist data it is recommended that you pre-create directories and map them via the Docker run command to.

**DataStax recommends the following mounts be made** 

**Users that do not expose the following mounts outside of the container will lose all data**

For DSE: `/var/lib/cassandra/data`  `/var/lib/cassandra/commit_logs` and `/var/lib/cassandra/saved_caches`

For OpsCenter: `/var/lib/opscenter`

For Studio: `/var/lib/datastax-studio`

To mount a volume, you would use the `-v` flag with the docker run command when starting the container with the following syntax.  

```
docker run -v <some_root_dir>:<container_volume>:<options>
```
**For example let’s mount the host directory /dse/conf on the exposed volume /conf**

```
docker run -e DS_LICENSE=accept --name my-dse -d  -v /dse/conf:/conf datastax/dse-server:5.1.4
```

Please referece the [Docker volumes doc](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume) for more information on mounting Volumes

# Exposing Ports on the Docker Host

Chances are you'll want to expose some ports on the Docker host so that you can talk to DSE from outside of Docker (for example, from code running on a machine outside of the host). You can do that using the -p switch when calling docker run and the most common port you'll probably want to expose is `9042` which is where CQL clients communicate. 

**For example**:

```
docker run -e DS_LICENSE=accept --name my-dse -d -p 9042:9042 datastax/dse-server:5.1.4
```

This will expose the container's CQL client port (9042) on the host at port 9042. For a list of the ports used by DSE, see the [Securing DataStax Enterprise ports documentation](http://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/security/secFirewallPorts.html).



# Starting DSE nodes

The image's entrypoint script runs the command dse cassandra and will append any switches you provide to that command. So it's possible to start DSE in any of the other supported modes by adding switches to the end of your docker run command.



Docker Run Switches | Description
------------- | -------------
--name | Optional: Assign a name to the container
-e | Sets environment variables<BR>Required: DS_LICENSE=accept for containers to start<BR> Optional: Other environment variables</BR>
-d | Recommended: Starts the container in the background
-p | Publish containers ports to the host<BR>Optional : for DSE<BR>Required: for Opscenter and Studio</BR>
-v | Optional: Bind Mount a Volume

These are the most commonly used `docker run` switches used in deploying DSE.  For a full list please see [docker run](https://docs.docker.com/engine/reference/commandline/run/) reference.


DSE start switches | Description
------------- | -------------
-s | Optional: Starts a search node
-k | Optional: Starts an analytics node
-g | Optional: Starts a graph node

By default, DSE will start in Cassandra only mode.

**Example: Start DSE in Cassandra only mode**

```
docker run -e DS_LICENSE=accept --name my-dse -d datastax/dse-server:5.1.4
```
You might replace 5.1.4 with any other version available at the [Docker Hub tags](https://hub.docker.com/u/datastax/dse-server/tags/).

**Example: Start a Graph Node**

```
docker run -e DS_LICENSE=accept --name my-dse -d datastax/dse-server:5.1.4 -g
In the container, this will run dse cassandra -g to start a graph node.
```

**Example: Start an Analytics (Spark) Node**

```
docker run -e DS_LICENSE=accept --name my-dse -d datastax/dse-server:5.1.4 -k
In the container, this will run dse cassandra -k to start an analytics node.
```

**Example: Start a Search Node**

```
docker run -e DS_LICENSE=accept --name my-dse -d datastax/dse-server:5.1.4 -s
In the container, this will run dse cassandra -s to start a search node.
```

You can also use combinations of those switches. For more examples, see the Starting [DSE documentation](http://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/operations/startStop/startDseStandalone.html).



### Container shell access and viewing Cassandra logs

**Attaching to running container**

If you used the -d flag to start you containers in the background, instead of using docker exec for individual commands you may want a bash shell to run commands. To do this use 

```
docker exec -it <container_name> bash
```

**For example**

```
docker exec -it my-dse bash
```

To exit the shell without stopping the container use *`ctl P ctl Q`*

**You can view logs via Docker's container logs**

```
docker logs my-dse
```
You can also use your favorite viewer/editor for individual files

`docker exec -it <container_name> <viewer/editor> <path_to_log>`

**For example**  to view the system.log

```
docker exec -it my-dse less /var/log/cassandra/system.log
```

## Starting DSE Tools

With a node running, use` docker exec` to run other tools. 

**For example**

`nodetool status` command:

```
docker exec -it my-dse nodetool status
```

`cqlsh`:

```
docker exec -it my-dse cqlsh
```

See [DSE documentation](http://docs.datastax.com/en/dse/5.1/dse-admin/) for further info on usage/configuration 



# Starting an OpsCenter node

```
docker run -e DS_LICENSE=accept --name my-opscenter -d -p 8888:8888 datastax/dse-opscenter:6.5.0
```

Now you can start DSE nodes, providing the link to the opscenter

```
docker run -e DS_LICENSE=accept --link my-opscenter:opscenter --name my-dse -d datastax/dse-server:5.1.4
```

Open your browser and point to `http://DOCKER_HOST_IP:8888`, create the new connection: - Choose "Manage existing cluster" - Use my-dse as the host name - Choose "Install agents manually"
See [OpsCenter documentation](http://docs.datastax.com/en/opscenter/6.1/) for further info on usage/configuration.


# Starting a Studio container

```
docker run -e DS_LICENSE=accept --link my-dse --name my-studio -p 9091:9091 -d datastax/dse-studio:2.0.0
```

Open your browser and point to `http://DOCKER_HOST_IP:9091`, create the new connection using my-dse as the hostname. Check [Studio docs](http://docs.datastax.com/en/dse/5.1/dse-dev/datastax_enterprise/studio/stdToc.html) for further instructions.



# Using Docker Compose for Automated Provisioning

Bootstrapping a multi-node cluster with OpsCenter and Studio can be elegantly automated with [Docker Compose](https://docs.docker.com/compose/). You can get sample `compose.yml` file here<link to>

**3-Node Setup**

```
docker-compose up -d --scale node=2
```

**3-Node Setup with OpsCenter**

```
docker-compose -f docker-compose.yml -f docker-compose.opscenter.yml up -d --scale node=2
```

**3-Node Setup with OpsCenter and Studio**

```
docker-compose -f docker-compose.yml -f docker-compose.opscenter.yml -f docker-compose.studio.yml up -d --scale node=2
```

**Single Node Setup with Studio**

```
docker-compose -f docker-compose.yml -f docker-compose.studio.yml up -d --scale node=0
```

# Building

The code in this repository will build the images listed above. To build all of them please run the following commands:

```console
./gradlew buildImages -PdownloadUsername=<your_DataStax_Acedemy_username> -PdownloadPassword=<your_DataStax_Acedemy_passwd>
```

By default, [Gradle](https://gradle.org) will download DataStax tarballs from [DataStax Academy](https://downloads.datastax.com).
Therefore you need to provide your credentials either via the command line, or in `gradle.properties` file located
in the project root.

Run `./gradlew tasks` to get the list of all available tasks.

# Getting Help

If you are a customer of DataStax, please use the official support channels for any help you need.

# License

Please review the included LICENSE file.

[datastax-enterprise]: http://www.datastax.com/products/datastax-enterprise
[docker-hub-tags]: https://hub.docker.com/u/datastax/dse-server/tags/
[start-dse]: http://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/operations/startStop/startDseStandalone.html
[dse-ports]: http://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/security/secFirewallPorts.html
[opscenter-docs]: http://docs.datastax.com/en/opscenter/6.1/index.html
[studio-docs]: http://docs.datastax.com/en/dse/5.1/dse-dev/datastax_enterprise/studio/stdToc.html
