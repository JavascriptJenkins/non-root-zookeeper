[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-zookeeper/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-zookeeper/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/zookeeper)](https://hub.docker.com/r/bitnami/zookeeper/)
[![Slack](http://slack.oss.bitnami.com/badge.svg)](http://slack.oss.bitnami.com)

# What is zookeeper?
>ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or other by distributed applications.

[https://zookeeper.apache.org/](https://zookeeper.apache.org/)


# TL;DR;

```bash
docker run --name zookeeper bitnami/zookeeper:latest
```

## Docker Compose

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Get this image

The recommended way to get the Bitnami Zookeeper Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/zookeeper).

```bash
docker pull bitnami/zookeeper:latest
```

To use a specific version, you can pull a versioned tag. You can view the
[list of available versions](https://hub.docker.com/r/bitnami/zookeeper/tags/)
in the Docker Hub Registry.

```bash
docker pull bitnami/zookeeper:[TAG]
```

If you wish, you can also build the image yourself.

```bash
docker build -t bitnami/zookeeper:latest https://github.com/bitnami/bitnami-docker-zookeeper.git
```

# Persisting your data

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

**Note!**
If you have already started using Zookeeper, follow the steps on
[backing up](#backing-up-your-container) and [restoring](#restoring-a-backup) to pull the data from your running container down to your host.

The image exposes a volume at `/bitnami/zookeeper` for the Zookeeper data and configurations. For persistence you can mount a directory at this location from your host. If the mounted directory is empty, it will be initialized on the first run.

```bash
docker run -v /path/to/zookeeper-persistence:/bitnami/zookeeper bitnami/zookeeper:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    volumes:
      - /path/to/zookeeper-persistence:/bitnami/zookeeper
```

# Connecting to other containers

Using [Docker container networking](https://docs.docker.com/engine/userguide/networking/), a Zookeeper server running inside a container can easily be accessed by your application containers.

Containers attached to the same network can communicate with each other using the container name as the hostname.

## Using the Command Line

In this example, we will create a Zookeeper client instance that will connect to the server instance that is running on the same docker network as the client.

### Step 1: Create a network

```bash
$ docker network create app-tier --driver bridge
```

### Step 2: Launch the Zookeeper server instance

Use the `--network app-tier` argument to the `docker run` command to attach the Zookeeper container to the `app-tier` network.

```bash
$ docker run -d --name zookeeper-server \
    --network app-tier \
    bitnami/zookeeper:latest
```

### Step 3: Launch your Zookeeper client instance

Finally we create a new container instance to launch the Zookeeper client and connect to the server created in the previous step:

```bash
$ docker run -it --rm \
    --network app-tier \
    bitnami/zookeeper:latest zkCli.sh -server zookeeper-server:2181  get /
```

## Using Docker Compose

When not specified, Docker Compose automatically sets up a new network and attaches all deployed services to that network. However, we will explicitly define a new `bridge` network named `app-tier`. In this example we assume that you want to connect to the Zookeeper server from your own custom application image which is identified in the following snippet by the service name `myapp`.

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    networks:
      - app-tier
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - app-tier
```

> **IMPORTANT**:
>
> 1. Please update the `YOUR_APPLICATION_IMAGE` placeholder in the above snippet with your application image
> 2. In your application container, use the hostname `zookeeper` to connect to the Zookeeper server

Launch the containers using:

```bash
$ docker-compose up -d
```

# Configuration

The configuration can easily be setup with the Bitnami Zookeeper Docker image using the following environment variables:

 - `ZOO_PORT_NUMBER`: Zookeeper client port. Default: **2181**
 - `ZOO_SERVER_ID`: ID of the server in the ensemble. Default: **1**
 - `ZOO_TICK_TIME`: Basic time unit in milliseconds used by ZooKeeper for heartbeats. Default: **2000**
 - `ZOO_INIT_LIMIT`: ZooKeeper uses to limit the length of time the ZooKeeper servers in quorum have to connect to a leader. Default: **10**
 - `ZOO_SYNC_LIMIT`: How far out of date a server can be from a leader. Default: **5**
 - `ZOO_SERVERS`: Comma, space or colon separated list of servers. Example: zoo1:2888:3888,zoo2:2888:3888. No defaults.
 - `ZOO_CLIENT_USER`: User that will use Zookeeper clients to auth. Default: No defaults.
 - `ZOO_CLIENT_PASSWORD`: Password that will use Zookeeper clients to auth. No defaults.
 - `ZOO_SERVER_USERS`: Comma, semicolon or whitespace separated  list of user to be created.  Example: user1,user2,admin. No defaults
 - `ZOO_SERVER_PASSWORDS`: Comma, semicolo or whitespace separated list of passwords to assign to users when created. Example: pass4user1, pass4user2, pass4admin. No defaults
 - `ZOO_ENABLE_AUTH`: Enable Zookeeper auth. It uses SASL/Digest-MD5. Default: **no**
 - `ZOO_HEAP_SIZE`: Size in MB for the Java Heap options (Xmx and XMs). This env var is ignored if Xmx an Xms are configured via `JVMFLAGS`. Default: **1024**
 - `ALLOW_ANONYMOUS_LOGIN`: If set to true, Allow to accept connections from unauthenticated users. Default: **no**
 - `JVMFLAGS`: Default JVMFLAGS for the ZooKeeper process. No defaults


```bash
docker run --name zookeeper -e ZOO_SERVER_ID=1 bitnami/zookeeper:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    environment:
      - ZOO_SERVER_ID=1
```


## Configuration
The image looks for configuration in the `conf/` directory of `/bitnami/zookeeper`.

```
docker run --name zookeeper -v /path/to/my_custom_conf_directory:/bitnami/zookeeper bitnami/zookeeper:latest
```
After that, your changes will be taken into account in the server's behaviour.

### Step 1: Run the Zookeeper image

Run the Zookeeper image, mounting a directory from your host.

```bash
docker run --name zookeeper -v /path/to/zookeeper-persistence:/bitnami/zookeeper bitnami/zookeeper:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    volumes:
      - /path/to/zookeeper-persistence:/bitnami/zookeeper
```

### Step 2: Edit the configuration

Edit the configuration on your host using your favorite editor.

```bash
vi /path/to/zookeeper-persistence/conf/zoo.cfg
```

### Step 3: Restart Zookeeper

After changing the configuration, restart your Zookeeper container for changes to take effect.

```bash
docker restart zookeeper
```

or using Docker Compose:

```bash
docker-compose restart zookeeper
```
## Security

Authentication based on SASL/Digest-MD5 can be easily enabled by passing the `ZOO_ENABLE_AUTH` env var.
When enabling the ZooKeeper authentication, it is also required to pass the list of users and passwords that will
be able to login.

```bash
docker run -it -e ZOO_ENABLE_AUTH=yes \
               -e ZOO_SERVER_USERS=user1,user2 \
               -e ZOO_SERVER_PASSWORDS=pass4user1,pass4user2 \
               bitnami/zookeeper
```

or using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    environment:
      - ZOO_ENABLE_AUTH=yes
      - ZOO_SERVER_USERS=user1,user2
      - ZOO_SERVER_PASSWORDS=pass4user1,pass4user2
```

## Setting up a Zookeeper ensemble

A Zookeeper (https://zookeeper.apache.org/doc/r3.1.2/zookeeperAdmin.html) cluster can easily be setup with the Bitnami Zookeeper Docker image using the following environment variables:

 - `ZOO_SERVERS`: Comma or colon separated list of servers. Example: zoo1:2888:3888,zoo2:2888:3888. No defaults.

For reliable ZooKeeper service, you should deploy ZooKeeper in a cluster known as an ensemble. As long as a majority of the ensemble are up, the service will be available. Because Zookeeper requires a majority, it is best to use an odd number of machines. For example, with four machines ZooKeeper can only handle the failure of a single machine; if two machines fail, the remaining two machines do not constitute a majority. However, with five machines ZooKeeper can handle the failure of two machines.

You have to use 0.0.0.0 as the host for the server. More concretely, if the ID of the zookeeper1 container starting is 1, then the ZOO_SERVERS environment variable has to be 0.0.0.0:2888:3888,zookeeper2:2888:3888.zookeeper3:2888:3888. See below:

Create a Docker network to enable visibility to each other via the docker container name

```bash
docker network create app-tier --driver bridge
```



### Step 1: Create the first node

The first step is to create one  Zookeeper instance.

```bash
docker run --name zookeeper1 \
  --network app-tier \
  -e ZOO_SERVER_ID=1 \
  -e ZOO_SERVERS=0.0.0.0:2888:3888,zookeeper2:2888:3888,zookeeper3:2888:3888 \
  -p 2181:2181 \
  -p 2888:2888 \
  -p 3888:3888 \
  bitnami/zookeeper:latest
```

### Step 2: Create the second node

Next we start a new Zookeeper container.

```bash
docker run --name zookeeper2 \
  --network app-tier \
  -e ZOO_SERVER_ID=2 \
  -e ZOO_SERVERS=zookeeper1:2888:3888,0.0.0.0:2888:3888,zookeeper3:2888:3888 \
  -p 2181:2181 \
  -p 2888:2888 \
  -p 3888:3888 \
  bitnami/zookeeper:latest
```

### Step 3: Create the third node

Next we start another new Zookeeper container.

```bash
docker run --name zookeeper3 \
  --network app-tier \
  -e ZOO_SERVER_ID=3 \
  -e ZOO_SERVERS=zookeeper1:2888:3888,zookeeper2:2888:3888,0.0.0.0:2888:3888 \
  -p 2181:2181 \
  -p 2888:2888 \
  -p 3888:3888 \
  bitnami/zookeeper:latest
```
You now have a two node Zookeeper cluster up and running. You can scale the cluster by adding/removing slaves without incurring any downtime.

With Docker Compose the ensemble can be setup using:

```yaml
version: '2'

services:
  zookeeper1:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181'
      - '2888'
      - '3888'
    volumes:
      - /path/to/zookeeper-persistence:/bitnami/zookeeper
    environment:
      - ZOO_SERVER_ID=1
      - ZOO_SERVERS=0.0.0.0:2888:3888,zookeeper2:2888:3888,zookeeper3:2888:3888
  zookeeper2:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181'
      - '2888'
      - '3888'
    volumes:
      - /path/to/zookeeper-persistence:/bitnami/zookeeper
    environment:
      - ZOO_SERVER_ID=2
      - ZOO_SERVERS=zookeeper1:2888:3888,0.0.0.0:2888:3888,zookeeper3:2888:3888
  zookeeper3:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181'
      - '2888'
      - '3888'
    volumes:
      - /path/to/zookeeper-persistence:/bitnami/zookeeper
    environment:
      - ZOO_SERVER_ID=3
      - ZOO_SERVERS=zookeeper1:2888:3888,zookeeper2:2888:3888,0.0.0.0:2888:3888
```

# Logging

The Bitnami Zookeeper Docker image sends the container logs to the `stdout`. To view the logs:

```bash
docker logs zookeeper
```

or using Docker Compose:

```bash
docker-compose logs zookeeper
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

# Maintenance

## Backing up your container

To backup your data, configuration and logs, follow these simple steps:

### Step 1: Stop the currently running container

```bash
docker stop zookeeper
```

or using Docker Compose:

```bash
docker-compose stop zookeeper
```

### Step 2: Run the backup command

We need to mount two volumes in a container we will use to create the backup: a directory on your host to store the backup in, and the volumes from the container we just stopped so we can access the data.

```bash
docker run --rm -v /path/to/zookeeper-backups:/backups --volumes-from zookeeper busybox \
  cp -a /bitnami/zookeeper:latest /backups/latest
```

or using Docker Compose:

```bash
docker run --rm -v /path/to/zookeeper-backups:/backups --volumes-from `docker-compose ps -q zookeeper` busybox \
  cp -a /bitnami/zookeeper:latest /backups/latest
```

## Restoring a backup

Restoring a backup is as simple as mounting the backup as volumes in the container.

```bash
docker run -v /path/to/zookeeper-backups/latest:/bitnami/zookeeper bitnami/zookeeper:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    volumes:
      - /path/to/zookeeper-backups/latest:/bitnami/zookeeper
```

## Upgrade this image

Bitnami provides up-to-date versions of Zookeeper, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

### Step 1: Get the updated image

```bash
docker pull bitnami/zookeeper:latest
```

or if you're using Docker Compose, update the value of the image property to
`bitnami/zookeeper:latest`.

### Step 2: Stop and backup the currently running container

Before continuing, you should backup your container's data, configuration and logs.

Follow the steps on [creating a backup](#backing-up-your-container).

### Step 3: Remove the currently running container

```bash
docker rm -v zookeeper
```

or using Docker Compose:


```bash
docker-compose rm -v zookeeper
```

### Step 4: Run the new image

Re-create your container from the new image, [restoring your backup](#restoring-a-backup) if necessary.

```bash
docker run --name zookeeper bitnami/zookeeper:latest
```

or using Docker Compose:

```bash
docker-compose start zookeeper
```

# Notable Changes

## 3.4.10-r4

- The zookeeper container has been migrated to a non-root container approach. Previously the container run as `root` user and the zookeeper daemon was started as `zookeeper` user. From now own, both the container and the zookeeper daemon run as user `1001`.
  As a consequence, the configuration files are writable by the user running the zookeeper process.

## 3.4.10-r0

- New release


# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-zookeeper/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-zookeeper/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-zookeeper/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# Community

Most real time communication happens in the `#containers` channel at [bitnami-oss.slack.com](http://bitnami-oss.slack.com); you can sign up at [slack.oss.bitnami.com](http://slack.oss.bitnami.com).

Discussions are archived at [bitnami-oss.slackarchive.io](https://bitnami-oss.slackarchive.io).

# License

Copyright (c) 2015-2018 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
