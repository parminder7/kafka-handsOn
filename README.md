# Table of content


- [Set up Kafka environment](https://github.com/parminder7/kafka-sample/blob/master/README.md#set-up-a-complete-kafka-environment)

	- [Pre-requisite](https://github.com/parminder7/kafka-sample#pre-requisite)
	- [Step #1: Install JAVA on all machines](https://github.com/parminder7/kafka-sample#step-1-install-java-on-all-machines)
	- [Step #2: Install Kafka](https://github.com/parminder7/kafka-sample#step-2-install-kafka)
	- [Step #3: Configure Kafka nodes](https://github.com/parminder7/kafka-sample#step-3-configure-kafka-nodes)
	- [Step #4: Configure Kafka Zookeeper node](https://github.com/parminder7/kafka-sample#step-4-configure-kafka-zookeeper-nodes)
	
- [Testing]()



## Set up a complete Kafka environment

Here, we have provisioned two kafka nodes with two zookeeper nodes. 

### Pre-requisite

- Create 4 linux based VM/machines. (Ubuntu 17.10, CPU 4 Core, Memory 8GB, Disk 250GB) 

### Step #1: Install JAVA on all machines

Followed below steps to install Java on unbutu:

```sh
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update 
sudo apt-get install openjdk-8-jdk 
```

Set up the appropriate environment variables for Java.

```sh
sudo vi /etc/profile
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
export JRE_HOME=/usr/lib/jvm/jre
```

### Step #2: Install Kafka 

Installing kafka is very simple. One can check the latest release [here](https://kafka.apache.org/downloads).

- First, download the kafka tar on all machine 

```sh
wget https://www.apache.org/dyn/closer.cgi?path=/kafka/1.0.1/kafka_2.12-1.0.1.tgz
```

- Untar it

```sh
tar -xvzf kafka_2.12-1.0.1.tgz
```

- Explore the extracted folder. You will see `bin` and `config` folders. These are important folders as you will be working with those in next steps.

### Step #3: Configure Kafka nodes

After downloading Kafka, the next step is to change/update some configuration on Kafka.

- Edit `/kafka_2.12-1.0.1/config/server.properties` file. Provide IPs of zookeeper server (don't worry, we'll be configuring zookeeper in later steps), log output path and broker identifier (remember, it should be unique id for each kafka server).

```sh
	zookeeper.connect=9.30.42.237:2181,9.30.118.104:2181
	log.dirs=/tmp/kafka-logs-0
	listeners=PLAINTEXT://:9092
	broker.id=0
```

- Make sure that log output path already exist on nodes with right execution permissions.

```sh
mkdir /tmp/kafka-logs-0
drwxr-xr-x  2 root root 4096 Mar 19 17:17 kafka-logs-0
```

### Step #4: Configure Kafka Zookeeper nodes

In this step, we will be configuring zookeeper nodes. Zookeeper is a high avalibilty coordination service that Kafka uses for coordination among brokers.

I followed the information provided in the apache zookeeper [doc](https://zookeeper.apache.org/doc/r3.4.10/zookeeperAdmin.html#sc_zkMulitServerSetup).

- Edit `/kafka_2.12-1.0.1/config/zookeeper.properties` file. Provide data directory path. Make sure `dataDir` file exists on the nodes.

```sh 
dataDir=/tmp/zookeeper-0
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
initLimit=5
syncLimit=2
server.123=9.30.118.104:2888:3888
server.125=0.0.0.0:2888:3888
```

To let every zookeeper know about every other nodes in ensemble, we need to provide IDs, IPs and ports as follows.

```sh
server.123=9.30.118.104:2888:3888
server.125=0.0.0.0:2888:3888
```

`123` and `125` are unique identifier provided in `dataDir/myid` file on each nodes.

`myid` file consists of a single line containing id of node.

`2888` this is the port used by followers to connect to leader and `3888` is for leader election. (Make sure these ports are open on nodes)

### Step #5: Start Apache Kafka and Zookeeper services on nodes

- Run Kafka as demon process on Kafka nodes

```sh
nohup bin/kafka-server-start.sh config/server.properties &
./bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```


## Testing

- *Get the list of active brokers* 

```sh
./bin/zookeeper-shell.sh 9.30.42.237:2181 <<< "ls /brokers/ids"
```

<img width="1148" alt="screen shot 2018-03-23 at 3 03 16 pm" src="https://media.github.ibm.com/user/54527/files/7f692eac-2eab-11e8-959b-864f57cd6988">


- *Push a topic* within Kafka cluster (containing two Kafka servers) with two partitions and replicate partition over two Kafka broker. 

```sh
./bin/kafka-topics.sh --create --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --topic test-topic --partitions 2 --replication-factor 2

Created topic "test-topic".
```

Note that under the Kafka node's logs path, two partition has been created `test-topic-0` & `test-topic-1` on both nodes.

<img width="1002" alt="screen shot 2018-03-22 at 6 52 12 pm" src="https://media.github.ibm.com/user/54527/files/5875587e-2e02-11e8-8a2d-d1c148385728">

- *Push a topic* within Kafka cluster (containing two Kafka servers) with two partitions and no replication. 

```sh
./bin/kafka-topics.sh --create --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --topic non-repeat-topic --partitions 2 --replication-factor 1

Created topic "non-repeat-topic".
```

You will observe below.

On Kafka node #1:

<img width="866" alt="screen shot 2018-03-22 at 7 02 30 pm" src="https://media.github.ibm.com/user/54527/files/e0fbd118-2e03-11e8-8194-859c4c23ca3f">

On Kafka node #2:

<img width="890" alt="screen shot 2018-03-22 at 7 02 45 pm" src="https://media.github.ibm.com/user/54527/files/e0da810c-2e03-11e8-9897-fb168af0496c">


- *Get the list of topics* on Kafka cluster.

```sh
./bin/kafka-topics.sh --list --zookeeper 9.30.42.237:2181 9.30.118.10:2181

test-topic
non-repeat-topic
```

- *Describe `test-topic`* and `non-repeat-topic` topics.

```sh
./bin/kafka-topics.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --describe --topic test-topic
```

<img width="1153" alt="screen shot 2018-03-22 at 7 16 24 pm" src="https://media.github.ibm.com/user/54527/files/ac64bbe8-2e05-11e8-95ae-41e38fb7f4b9">

<img width="1149" alt="screen shot 2018-03-22 at 7 19 10 pm" src="https://media.github.ibm.com/user/54527/files/f04a8d60-2e05-11e8-921b-3cf4c6d4dde2">

```sh
Topic: test-topic	Partition: 0	Leader: 1	Replicas: 1,0	Isr: 1
``` 

represents Kafka broker with id `0` acting as leader and handling all reads/writes for partition `0`. `Replicas: 1,0` meaning partition `0` is replicated on broker id 0 and 1.   

- *Delete topic* `test-topic` from Kafka cluster.

To enable deletion, we need to make sure to add `delete.topic.enable=true` on all Kafka brokers `server.properties` config file.

```sh
./bin/kafka-topics.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --delete --topic test-topic
```

- *Run a producer* process to publish data into `test-topic` Kafka topic within the Kafka broker.

```sh
bin/kafka-console-producer.sh --broker-list  9.30.118.212:9092,9.30.214.93:9092 --topic test-topic
>
```

- *Create two consumer groups*. Edit `/kafka_2.11-1.0.1/config/consumer.properties` with group id `test-consumer-group-0` and `test-consumer-group-1` by creating another `consumer.properties1` config.

```sh
# consumer group id
group.id=test-consumer-group-1
```

```sh
bin/kafka-console-consumer.sh --bootstrap-server 9.30.118.212:9092 --topic test-topic --new-consumer --consumer.config config/consumer.properties
```


## Observation

- What happens if all the Kafka brokers died?

The consumer pulling data from topic gets `Error while fetching metadata with correlation id 43 : {test-topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)` error.
