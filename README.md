# Manual Kafka single broker setup

## Downloading Kafka

Open a terminal on your local machine (assuming JDK is already installed).

```shell
wget https://downloads.apache.org/kafka/3.3.2/kafka_2.12-3.3.2.tgz
```

```shell
tar -xzf kafka_2.12-3.3.2.tgz
```

## Staring Zookeeper

```shell
./kafka_2.12-3.3.2/bin/zookeeper-server-start.sh kafka_2.12-3.3.2/config/zookeeper.properties
```

## Starting Kafka

```shell
./kafka_2.12-3.3.2/bin/kafka-server-start.sh kafka_2.12-3.3.2/config/server.properties
```

## Creating a topic

```shell
./kafka_2.12-3.3.2/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test
```

## Test producing events

```shell
./kafka_2.12-3.3.2/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

## Test consuming events

```shell
./kafka_2.12-3.3.2/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

# Manual Kafka multi-broker setup

## Creating VMs

1. Create new DataCenter in the DCD.
2. Darg & drop server into canvas and connect to internet connection. Repeat that for three Broker servers and three Zookeeper servers.
3. Click on a server, go to Storage tab and add a storage with an Ubuntu image. Do that for all six servers.
4. Open a terminal on your local machine and generate an SSH key `ssh-keygen -t rsa`.
5. Add the generated public key to the SSH management in the DCD. 

## Connect to VM

You can login as root by password the SSH key like so:

```shell
ssh root@ipaddress -i ./kafkabrokers
```

Or by clicking `Console` in the DCD.

|          | Kafka-Broker-1  | Kafka-Broker-2  | Kafka-Broker-3  |
|----------|-----------------|-----------------|-----------------|
| User     | root            | root            | root            |
| Password | broker1passwort | broker2passwort | broker3passwort |
| IP       | 217.160.120.48  | 217.160.117.143 | 212.227.224.208 |

|          | Zookeeper-Node-1   | Zookeeper-Node-2   | Zookeeper-Node-3   |
|----------|--------------------|--------------------|--------------------|
| User     | root               | root               | root               |
| Password | zookeeper1passwort | zookeeper2passwort | zookeeper3passwort |
| IP       | 217.160.216.182    | 217.160.216.159    | 217.72.202.14      |


## Installing JDK

```shell
apt-get update
```

```shell
apt-get install openjdk-8-jdk
```

Execute those two commands on all six servers.

## Downloading Kafka

```shell
wget https://downloads.apache.org/kafka/3.3.2/kafka_2.12-3.3.2.tgz
```

```shell
tar -xzf kafka_2.12-3.3.2.tgz
```

Execute those two commands on all six servers.

## Preparing and starting Zookeeper

Create `/home/zookeeper_data` directory.
```shell
mkdir /home/zookeeper_data
```

Create file `/home/zookeeper_data/myid` and add integer character of the individual Zookeeper node ID (so either `1`, `2` or `3`).

```shell
vi /home/zookeeper_data/myid
```

Change data directory for Zookeeper in the `zookeeper.properties` file to `/home/zookeeper_data`.
```shell
vi kafka_2.12-3.3.2/config/zookeeper.properties
```

Add and/or change the following lines to the `zookeeper.properties`:

```
initLimit=5
syncLimit=2
server.1=217.160.216.182:2888:3888
server.2=217.160.216.159:2888:3888
server.3=217.72.202.14:2888:3888
```

Run Zookeeper in the background:

```shell
/root/kafka_2.12-3.3.2/bin/zookeeper-server-start.sh /root/kafka_2.12-3.3.2/config/zookeeper.properties > /dev/null 2>&1 &
```

> Repeat for all three Kafka brokers.

Test if Zookeeper is running:

```shell
/root/kafka_2.12-3.3.2/bin/zookeeper-shell.sh localhost:2181 ls /brokers/ids
```

Shouldn't show any brokers currently.

## Preparing and starting Brokers

Create `/home/kafka_data` directory.
```shell
mkdir /home/kafka_data
```

Change data directory for Kafka in the `server.properties` file to `/home/kafka_data`.
```shell
vi kafka_2.12-3.3.2/config/server.properties
```

Add and/or change the following lines to the `server.properties`:

```
advertised.listeners=PLAINTEXT://212.227.224.208:9092
log.dirs=/home/kafka_data
zookeeper.connect=217.160.216.182:2181,217.160.216.159:2181,217.72.202.14:2181
```

Run Kafka in background:

```shell
/root/kafka_2.12-3.3.2/bin/kafka-server-start.sh /root/kafka_2.12-3.3.2/config/server.properties > /dev/null 2>&1 &
```

> Repeat for all three Kafka brokers.

# Kubernetes setup

## Setting the Kubernetes context

To make sure our command-line tools are working on the correct Kubernetes cluster, we set the `$KUBECONFIG` environment variable so that it points to the right `kubeconfig.yaml`.

```export
export KUBECONFIG=/Users/myuser/kubeconfig.yaml
```

## Using the Stackable operator

> Tested on a Kubernetes cluster with Stackabale Relaese 23.4.1 installed.

Pass the `kafka-zookeeper-deployment.yaml` that describes the deployment for the operators:

```shell
kubectl apply -f kafka-zookeeper-deployment.yaml
```

## Get address of NodePort service

> To install `stackablectl` visit the [official Stackable documentation](https://docs.stackable.tech/stackablectl/stable/installation.html).

A fast and easy way to get the NodePort service address is to use `stackablectl`:

```shell
stackablectl services list
```

Copy the `kafka` endpoint from the console output.

Alternatively to using `stackablectl` you can pick any `EXTERNAL-IP` of a node of your choice and the high-numbered port of the `simple-kafka` service that maps to `9092`. To get the necessary information run the following two commands:

```shell
kubectl get nodes -o wide
kubectl get service simple-kafka
```

## Creating a topic

```shell
./kafka_2.12-3.3.2/bin/kafka-topics.sh --create --bootstrap-server 82.165.231.240:31429 --topic test
```

## Test producing events

```shell
./kafka_2.12-3.3.2/bin/kafka-console-producer.sh --broker-list 82.165.231.240:31429 --topic test
```

## Test consuming events

```shell
./kafka_2.12-3.3.2/bin/kafka-console-consumer.sh --bootstrap-server 82.165.231.240:31429 --topic test --from-beginning
```