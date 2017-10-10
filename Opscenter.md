# Short Description
The web-based visual management and monitoring solution for DataStax Enterprise (DSE). DataStax Entperprise Image is here *KAT ADD LINK TO STORE*

# Supported Tags
* 6.1.2 (6.1.2/Dockerfile) *KAT ADD  LINK

# Quick Reference 
### Where to get help:
[DataStax Academy](https://academy.datastax.com/), [DataStax Slack](https://academy.datastax.com/slack), [DataStax Customer Support](https://support.datastax.com)

### Where to file issues:
https://github.com/datastax/docker/issues

### Maintained by 
[DataStax](https://www.datastax.com/) 

### Published image artifact details:
docs repo's opscenter/ directory *KAT ADD LINKS TO DOCS REPO AND OPSCENTER DOC*

# What is DataStax OpsCenter

OpsCenter is the web-based visual management and monitoring solution for DataStax Enterprise (DSE). 

DataStax Enterprise is the best distribution of Apache Cassandra™ with integrated Search, Analytics, and Graph capabilities. The DataStax Enterprise image can be found here *KAT PROVIDE LINKS*

# Getting Started with DataStax and Docker

The DataStax provided docker images are intended to be used for Development purposes in non-production environments. You can use these images to learn [DSE](LINK)*KAT PROVIDE, OpsCenter and [DataStax Studio]*LINK KAT PROVIDE, to try new ideas, and to test and demonstrate your application.


# EULA Acceptance
In order to use these images, it is necessary to accept the terms of the DataStax license. This is done by setting the environment variable `DS_LICENSE` to the value accept when running containers based on the produced images. To show the license included in the images, set the variable `DS_LICENSE` to the value `accept`. *The images will not start without the variable set to the accept value.*

```
docker run -e DS_LICENSE=accept --name my-opscenter -d -p 8888:8888 datastax/datastax-enterprise-opscenter:6.1.3
```

# Single Mount Configuration Management

DataStax has made it easy to make configuration changes by creating a script that looks in the exposed Volume `/conf` for any added configuration files and loads them at container start. 

To take advantage of this feature, you will need to create a mount directory on your host, mount your local directory to the exposed Volume `/conf`, place you modified configuration files in the mount directory on your host machine and start the container. 

These files will override the existing configuration files.  The configs must contain all the values to be used along with using the dse naming convention such as cassandra.yaml, dse.yaml, opscenterd.conf 

For a full list of configuration files please visit *some link here*

```
docker run -e DS_LICENSE=accept --name my-dse --name my-opscenter -d -p 8888:8888 -v datastax/datastax-enterprise-opscenter:6.1.3
```

 # Database and Data Storage

The following volumes are created and exposed with the images:  

**OpsCenter**

* `/var/lib/opscenter`: OpsCenter data

To persist data it is recommended that you pre-create directories and map them via the Docker run command to.

**DataStax recommends the following mounts be made** 

**Users that do not expose the following mounts outside of the container will lose all data**

For OpsCenter: `/var/lib/opscenter`


To mount a volume, you would use the `-v` flag with the docker run command when starting the container with the following syntax.  

```
docker run -v <some_root_dir>:<container_volume>:<options>
```
**For example let’s mount the host directory /dse/conf/opscenter on the exposed volume /conf**

```
docker run -e DS_LICENSE=accept --name my-dse -d  -v /dse/conf/opscenter:/conf datastax/datastax-enterprise-opscenter:6.1.3
```

Please referece the [Docker volumes doc](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume) for more information on mounting Volumes


# Starting an OpsCenter node

```
docker run -e DS_LICENSE=accept --name my-opscenter datastax/datastax-enterprise-opscenter:6.1.3
```

Now you can start DSE nodes, providing the link to the opscenter Please see *[DSE instructions to be provided]()* for more detailed information on DSE

```
docker run -e DS_LICENSE=accept --link my-opscenter:opscenter --name my-dse -d datastax/datastax-enterprise-node:5.1.4
```

Open your browser and point to `http://DOCKER_HOST_IP:8888`, create the new connection: - Choose "Manage existing cluster" - Use my-dse as the host name - Choose "Install agents manually"
See [OpsCenter documentation](http://docs.datastax.com/en/opscenter/6.1/) for further info on usage/configuration.


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
