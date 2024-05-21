# Alpine SQS _(alpine-sqs)_

![banner](https://raw.githubusercontent.com/roribio/alpine-sqs/master/banner.png)

[![](https://images.microbadger.com/badges/image/roribio16/alpine-sqs.svg)](https://microbadger.com/images/roribio16/alpine-sqs "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/version/roribio16/alpine-sqs.svg)](https://microbadger.com/images/roribio16/alpine-sqs "Get your own version badge on microbadger.com") [![Docker Pulls](https://img.shields.io/docker/stars/roribio16/alpine-sqs.svg)](https://hub.docker.com/r/roribio16/alpine-sqs/) [![Docker Pulls](https://img.shields.io/docker/pulls/roribio16/alpine-sqs.svg)](https://hub.docker.com/r/roribio16/alpine-sqs/) [![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

> Dockerized ElasticMQ server + web UI over Alpine Linux for local development.

Alpine SQS provides a containerized Java implementation of the Amazon Simple Queue Service (AWS-SQS). It is based on ElasticMQ running Alpine Linux and the Oracle Java 8 Server-JRE. It is compatible with AWS's API, CLI as well as the Amazon Java SDK. This allows for quicker local development without having to incurr in infrastructure costs.

The goal of this repository is to maintain an updated Docker environment for ElasticMQ with an integrated web UI for visualizing queues and messages.

## Table of Contents

- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [Maintainer](#maintainer)
- [Contribute](#contribute)
- [License](#license)

## Background
When searching for existing local implementations of SQS I came across a Docker image by [@vsouza](https://github.com/vsouza) called [docker-SQS-local](https://github.com/vsouza/docker-SQS-local) with over 11K pulls at the time.

This introduced me to ElasticMQ, which this project is based on and is described by it's creators as:

> a  message queue system, offering an actor-based Scala and an SQS-compatible REST (query) interface.

Using his work as inspiration I decided to improve upon it by implementing the following:

- Reduce the Docker image foot-print as much as possible.
- Automatically update to the latest ElasticMQ server.
- Thorough documentation.

### See also
For more information on the different projects this work is based on, please visit:

- [ElasticMQ](https://github.com/adamw/elasticmq) by [@adamw](https://github.com/adamw).
- [docker-alpine-java](https://github.com/anapsix/docker-alpine-java) by [anapsix](https://github.com/anapsix).

## Install
### Pre-requisites

To be able to use this environment, please make sure you have installed the latest version of [Docker](https://docs.docker.com/engine/installation/). 

If you intend to build the environment yourself, it is recommended that you also install the latest version of [Docker Compose](https://docs.docker.com/compose/install/).

### Installation methods
You can obtain the environment in two ways; The easiest is to pull the image directly from Docker Hub. Also, you may clone this repository and build/run it using Docker Compose.
#### 1. Pulling from Docker Hub
```
docker pull roribio16/alpine-sqs
```
#### 2. Building from scratch
```
git clone https://github.com/roribio/alpine-sqs.git
```
## Usage
### Running the environment
Depending on how you chose to install the environment, you can initialize it in three ways:

#### 1. `docker run` method
Use this method if you're pulling directly from Docker Hub and do not have a `docker-compose.yml` file.

```
docker run --name alpine-sqs -p 9324:9324 -p 9325:9325 -d roribio16/alpine-sqs:latest
```

#### 2. `docker-compose up` method
If you've cloned the repository you can still take advantage of the image present in Docker Hub by running the container from the default `docker-compose.yml` file. This will pull the pre-built image from the public registry and run it with the same values stated in the previous method.

```
docker-compose up -d
```

#### 3. `docker-compose up --build` method
To build the image from scratch and then run the corresponding container, use this method.

```
docker-compose -f docker-compose.build up -d --build
```

### Working with queues
ElasticMQ provides an Amazon-SQS compatible interface. This means you may use the AWS command-line tool, API calls and the Java SDK, to interact with local queues the same as if interacting with the actual SQS.

#### Sending a message
To send messages to a queue you need to specify the new endpoint url and queue url along with the message payload. The following example uses the AWS CLI to send a message to the `default` queue. 

```
aws --endpoint-url http://localhost:9324 sqs send-message --queue-url http://localhost:9324/queue/default --message-body "Hello, queue!"
```

### Creating new queues
You can create new queues by using the command-line or configuring ElasticMQ directly.

##### AWS CLI
```
aws --endpoint-url http://localhost:9324 sqs create-queue --queue-name newqueue
```

##### Edit ElasticMQ configuration file
Navigate to the directory where the configuration files reside and edit the `elasticmq.conf` file to add a new entry for each queue to the `queue` block.

```
queues {
    default {
        defaultVisibilityTimeout = 10 seconds
        delay = 5 seconds
        receiveMessageWait = 0 seconds
    },
    newqueue {
        defaultVisibilityTimeout = 10 seconds
        delay = 5 seconds
        receiveMessageWait = 0 seconds
    }
}
```

> **Note**: The configuration directory location inside the container is located at `/opt/config`. If you mounted that volume onto your host, you can also find the configuration files there.

After editing the `elasticmq.conf` file, you need to restart the ElasticMQ server by running the `supervisorctl restart elasticmq` command inside the container. If you're editing the configuration file outside of the container, use this command: 

```
docker exec -it alpine-sqs sh -c "supervisorctl restart elasticmq"
``` 

## License
Copyright 2017 Ronald E. Oribio R.

This project is licensed under the GNU General Public License, version 3.0. See the [LICENSE](./LICENSE) file for details.
