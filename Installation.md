---
layout: default
title: Installation
---
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
docker run --name zookeeper-1 --network host --detach \
-e ZOO_SERVER_ID=1 \
-e ZOO_PORT_NUMBER=2181 \
-e ALLOW_ANONYMOUS_LOGIN=yes \
-e ZOO_ADMIN_SERVER_PORT_NUMBER=8080  \
-e ZOO_SERVERS=localhost:2888:3888::1,localhost:2889:3889::2 \
-v zookeeper_1_data:/bitnami bitnami/zookeeper:3.8

docker run --name zookeeper-2 --network host --detach \
-e ZOO_SERVER_ID=2 \
-e ZOO_PORT_NUMBER=2182 \
-e ALLOW_ANONYMOUS_LOGIN=yes \
-e ZOO_ADMIN_SERVER_PORT_NUMBER=8081  \
-e ZOO_SERVERS=localhost:2888:3888::1,localhost:2889:3889::2 \
-v zookeeper_2_data:/bitnami bitnami/zookeeper:3.8
```

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
docker run --name kafka-1 --network host --detach \
-e KAFKA_BROKER_ID=1 \
-e KAFKA_CFG_ZOOKEEPER_CONNECT=localhost:2181,localhost:2182 \
-e KAFKA_CFG_LISTENERS=PLAINTEXT://:9091 \
-e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9091 \
-e ALLOW_PLAINTEXT_LISTENER=yes \
-v kafka_1_data:/bitnami docker.io/bitnami/kafka:3.2

docker run --name kafka-2 --network host --detach \
-e KAFKA_BROKER_ID=2 \
-e KAFKA_CFG_ZOOKEEPER_CONNECT=localhost:2181,localhost:2182 \
-e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092 \
-e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
-e ALLOW_PLAINTEXT_LISTENER=yes \
-v kafka_2_data:/bitnami docker.io/bitnami/kafka:3.2
```

### Creating kafka topics

```bash
kafka-topics.sh --create \
--topic net.2yetis.uvuyo.bppm2HelixITSM.send \
--replication-factor 2 \
--partitions 2 \
--bootstrap-server kafka-server:9092 \
```

## Install uvuyo


### Prepare the installation directory

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
- driver: "net.the2yetis.uvuyo.integration.bppm.BppmEventConnector"
  node: "uvuyo"
  id: "bppm"
  name: "BPPM"
  dispatcherId: "bppm2HelixITSM"
  active: true
  ruleModules: []
  port: 1828
  timeout: 1000
adapters:
- driver: "net.the2yetis.uvuyo.integration.helixItsm.HelixItsmEventAdapter"
  node: "uvuyo"
  id: "helixItsmEventAdapter"
  name: "HelixItsmEventAdapter"
  dispatcherId: "bppm2HelixITSM"
  active: true
  ruleModules: [uvuyo2HelixITSM]
  url: "https://aena-dev-restapi.onbmc.com"
  jwtRefreshInterval: 300
  username: "activationuser"
  password: "LCu3Ecrw9-zyF6d5"
  form: "HPD:IncidentInterface_Create"
  attributes:
  - First_Name
  - Last_Name
  - Description
  - Impact
  - Urgency
  - Status
  - "Reported Source"
  - Service_Type
  - "Assigned Group"
  - "Assigned Support Organization"
  - "Assigned Support Company"
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
    "Assigned Group": "'ABC - EscaladoSMS'"
    "Assigned Support Organization": "'ABC - ESCALADOS'"
    "Assigned Support Company": "'Aena'"
```

### Create a volume

The uvuyo container is configured to read the configuration files from the etc directory. Therefor
we need to create a docker volume pointing to the just created etc directory.

```bash
docker volume create --name uvuyo_etc --opt type=none --opt device=/opt/2yetis/uvuyo/etc --opt o=bind
```

### Run the uvuyo docker container

```bash
docker run --name unode --network host --detach \
-v uvuyo_etc:/app/uvuyo/etc the2yetis/uvuyo
```

