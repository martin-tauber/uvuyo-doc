---
layout: default
title: Installation
---
# Introduction

The uvuyo application consists of two components. The actual uvuyo node and a kafka instance which is responsible
for the sending relyablby sending events between two uvuyo nodes. Even if you only plan to have one node 
running, you will need a kafka instance. Besides sending events between uvuyo nodes, the kafka instance is 
responsible for persisting events until they are send successfully to their target system.

The uvuyo application does not come with a kafka installation. You will need to connect the uvuyo application to
your existing kafka installation or install a fresh kafka to use with uvuyo. This installation documentation has a
section on how to install kafka in a docker environment to help you getting started.

uvuyo is distributed as a docker container. We recommand to use docker to setup your application, since this simplifies
the adminitration of the product. You can also start an uvuyo node as a java process. For more information on
how to start uvuyo as a java process see section starting uvuyo as a java process.

In a production environment you will install uvuyo nodes on multiple hosts for availability and perfromance
reasons. Before you begin installing uvuyo you need to have a architectural plan in place on which hosts you
would like to install the different components. This plan should include the source and target systems connected to
uvuyo.

To install uvuyo you will need to perform the following steps

1. Install Zookeeper
2. Install Kafka
3. Install uvuyo nodes

# Docker

uvuyo uses kafka to  pass events between the different nodes. In order to install uvuyo you
need to install kafka. Kafka currently still needs a zookeeper installation running for coordination.
We recommand to install at least three kafka nodes and three zookeeper nodes in production to ensure
high availablility. These nodes shopuld be installed on different machines.

You can use an existing kafka installation for uvuyo or install your own version of kafka. This quide
will give you a quick introduction on installing kafka. For more detailed information please refer
to the offical kafka documentation


## Install Zookeeper

Before installing kafka you will need to install zookeeper. We recommand to use the bitnami zookeeper docker image. For
test environments it is sufficiant to have a single instance of zookeeper running. For production
environemnts it is strongly recommanded to have at least three instances of zookeeper running.

---
> **NOTE:**
> For more information about how to configure the bitnami zookeeper docker image go to the official
 [Bitnami Zookeeper](https://hub.docker.com/r/bitnami/zookeeper/) documentation.

---

### Example single node installation

Example how to run two zookeeper instances on the same host

```bash
hostname=`hostname`

docker run --name zookeeper-1 --network host --detach \
-e ZOO_SERVER_ID=1 \
-e ZOO_PORT_NUMBER=2181 \
-e ALLOW_ANONYMOUS_LOGIN=yes \
-e ZOO_ADMIN_SERVER_PORT_NUMBER=8080  \
-e ZOO_SERVERS=$hostname:2888:3888::1,$hostname:2889:3889::2 \
-v zookeeper_1_data:/bitnami bitnami/zookeeper:3.8

docker run --name zookeeper-2 --network host --detach \
-e ZOO_SERVER_ID=2 \
-e ZOO_PORT_NUMBER=2182 \
-e ALLOW_ANONYMOUS_LOGIN=yes \
-e ZOO_ADMIN_SERVER_PORT_NUMBER=8081  \
-e ZOO_SERVERS=$hostname:2888:3888::1,$hostname:2889:3889::2 \
-v zookeeper_2_data:/bitnami bitnami/zookeeper:3.8
```

In the example above we are creating a host network, whcih means that the containers are reachable directly
on the port of the host that they are running on. You might consider to craete a different kind of network
to further isolate the containers.

## Install Kafka

After installing zookeeper you need to install the kafka nodes. For a test environment it is sufficient to
have a single instance of kafka running, For production environments we strongly recommand to have at least
three instances of kafka running.

---
> **NODE:**
> For more informantion about how to configure the bitnami kafka docker image go to the offical
> [Bitnami Kafka](https://hub.docker.com/r/bitnami/kafka) documentation.

---


### Example Single node installation

Example how to run two kafka instances on the same host. This example uses the zookeeper containers created 
in the previous example where we ran two zookeeper containers on one host.

```bash
hostname=`hostname`

docker run --name kafka-1 --network host --detach \
-e KAFKA_BROKER_ID=1 \
-e KAFKA_CFG_ZOOKEEPER_CONNECT=$hostname:2181,$hostname:2182 \
-e KAFKA_CFG_LISTENERS=PLAINTEXT://:9091 \
-e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://$hostname:9091 \
-e ALLOW_PLAINTEXT_LISTENER=yes \
-v kafka_1_data:/bitnami docker.io/bitnami/kafka:3.2

docker run --name kafka-2 --network host --detach \
-e KAFKA_BROKER_ID=2 \
-e KAFKA_CFG_ZOOKEEPER_CONNECT=$hostname:2181,$hostname:2182 \
-e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092 \
-e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://$hostname:9092 \
-e ALLOW_PLAINTEXT_LISTENER=yes \
-v kafka_2_data:/bitnami docker.io/bitnami/kafka:3.2
```

### Creating kafka topics

Kafka topics are used to send events between connectors and adapters. Connectors and Adapters are attached to topics
using the dispatcherId property while being configured. Every DispatcherId will automatically create two kafka topics.
One to send events from the connector to the adpater called `send` and one to send replies from the adapter to the 
connector called `reply`. The topic name is generated by using the prefix `net.2yetis.uvuyo.` 
conacted with with dispatcherId and the prefix `.send` or `.reply`.

Even though tokics are created automatically for availability and scalability reasons we would create the topics manually
to meet these requirements. For availablity we would add a `--replication-factor` parameter. For scalability we would
add the `--partition` parameter.

Our recommandation is to run two kafka nodes in test environments and at least three kafka nodes in production. In test
you should then set the `--replication-factor` to 2 and the `--partitions` to 2. 

**Example:**

login to one of the kafka nodes just created

```bash
docker exec -it <kafkaNode> bash
```

Execute the command to create a topic

```bash
kafka-topics.sh --create \
--topic net.2yetis.uvuyo.bppm2HelixITSM.send \
--replication-factor 2 \
--partitions 2 \
--bootstrap-server hostname:9092
```

Don't forget to replace the hostname in the last command with the hostname on which the kafka node is running.

## Install uvuyo

The last step is to install the uvuyo containers. To do so we need to setup the appropriate file structures so
that we are able to configure the uvuyo nodes. 

### Prepare the installation directory

The uvuyo nodes use the `etc` directory to read their configuration files. Therefor we need to create an `etc`
directory which will later be mounted into the docker container of the uvuyo node.

```bash
sudo mkdir -p /opt/2yetis/uvuyo/etc
sudo chown -R ubuntu:ubuntu /opt/2yetis
```

### Create the node configuration file

An uvuyo node is configured using the nodes configuration file. The configuration file 
needs to have at minimum the `uvuyo.license.key` and the `uvuyo.license.owner` configured.
The node configuration file is a properties file, which means that every line consists of
a key value pair. The file is stored in the etc directory.

The file is called `{nodeid}.properties` where {nodeid} is the id of the node when 
started. By default the {nodeid} is uvuyo. 

As a minimal default configuration file you would need to create the file `etc/uvuyo.properties`
with the license information you received from 2yetis.

```properties
# License Information

uvuyo.license.key=88b3-3d17-e2f9-b750-72c3-735d-3a41-bd57-08d7-3456-5810
uvuyo.license.owner=2Yetis Consulting

# Kafka Connection
uvuyo.kafka.bootstrap.servers=localhost:9092
```

### Create the gateway configuration file

The gateway configuration file configures the actual adpters and connectors in the gateway. The file is
stored in the etc directory and the name is `node.{nodeid}.yaml` where {nodeid} is the id of the node
when started. By default the {nodeid} is uvuyo.

```yaml
---
connectors:
- driver: "net.the2yetis.uvuyo.integration.test.TestEventConnector"
  id: "testConnector"
  name: "testConnector"
  node: "uvuyo"
  dispatcherId: "bppm2HelixITSM"
  active: true
  ruleModules: []
adapters:
- driver: "net.the2yetis.uvuyo.integration.test.TestEventAdapter"
  id: "testAdapter"
  name: "testAdapter"
  node: "uvuyo"
  dispatcherId: "bppm2HelixITSM"
  active: true
  ruleModules: []
```

### Create the rules file

The rules file contains the rules used to map events. Rules files are loaded by the connectors
and adapters. If you configured an adapter or connector to load a rules file in the gateway
configuration file, you would need to create them.

Rule files are stored in the etc directory and names `module.{name}.yaml`, where {name} is the
name of the rules module. You would use this name in the gateway configuration file to load 
the appropriate rules module.

#### Example rule file

```yaml
- name: MapAttributes
  type: Map
  attributes:
    First_Name: "'Activation'"
    Last_Name: "'User'"
    Description: "'REST API: Incident Creation'"
    Impact: "'1-Minor/Localized'"
    Urgency: "'4-Low'"
    Status: "'Assigned'"
    "Reported Source": "'BMC Impact Manager Event'"
    Service_Type: "'User Service Restoration'"
    "Assigned Group": "'a user group'"
    "Assigned Support Organization": "'an organization'"
    "Assigned Support Company": "'2yetis'"
```

### Create a volume

The uvuyo container is configured to read the configuration files from the etc directory. Therefor
we need to create a docker volume pointing to the just created etc directory.

```bash
docker volume create --name uvuyo_etc --opt type=none --opt device=/opt/2yetis/uvuyo/etc --opt o=bind
```

### Run the uvuyo docker container

```bash
docker run --name unode-1 --network host --detach \
-e UVUYO_NODEID=unode-1 \
-e UVUYO_GROUPID=company-dev \
-v uvuyo_etc:/app/uvuyo/etc the2yetis/uvuyo
```

# Starting uvuyo as a java process