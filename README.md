# Table of content


- [Set up Kafka environment](https://github.com/parminder7/kafka-sample/blob/master/README.md#set-up-a-complete-kafka-environment)

	- [Pre-requisite](https://github.com/parminder7/kafka-sample#pre-requisite)
	- [Step #1: Install JAVA on all machines](https://github.com/parminder7/kafka-sample#step-1-install-java-on-all-machines)
	- [Step #2: Install Kafka](https://github.com/parminder7/kafka-sample#step-2-install-kafka)
	- [Step #3: Configure Kafka nodes](https://github.com/parminder7/kafka-sample#step-3-configure-kafka-nodes)
	- [Step #4: Configure Kafka Zookeeper node](https://github.com/parminder7/kafka-sample#step-4-configure-kafka-zookeeper-nodes)
	
- [Testing](https://github.com/parminder7/kafka-sample/blob/master/README.md#testing)
	- [Get the list of active brokers](https://github.com/parminder7/kafka-sample/blob/master/README.md#get-the-list-of-active-brokers)
	- [Push a topic with replica](https://github.com/parminder7/kafka-sample/blob/master/README.md#push-a-topic-with-replica)
	- [Push a topic without replica](https://github.com/parminder7/kafka-sample/blob/master/README.md#push-a-topic-without-replica)
	- [Get list of topics](https://github.com/parminder7/kafka-sample/blob/master/README.md#get-the-list-of-topics)
	- [Describe topic](https://github.com/parminder7/kafka-sample/blob/master/README.md#describe-topic)
	- [Delete topic](https://github.com/parminder7/kafka-sample/blob/master/README.md#delete-topic)
	- [Run a producer](https://github.com/parminder7/kafka-sample/blob/master/README.md#run-a-producer)
	- [Create a new consumer group](https://github.com/parminder7/kafka-sample/blob/master/README.md#create-two-consumer-groups)
	- [Run a consumer](https://github.com/parminder7/kafka-sample/blob/master/README.md#run-a-consumer-process)
	- [Run multiple consumers](https://github.com/parminder7/kafka-sample/blob/master/README.md#run-multiple-consumer-processes)
	- [Run multiple consumer groups](https://github.com/parminder7/kafka-sample/blob/master/README.md#run-multiple-consumer-groups)



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

### Get the list of active brokers

```sh
./bin/zookeeper-shell.sh 9.30.42.237:2181 <<< "ls /brokers/ids"
```

<img width="1148" alt="screen shot 2018-03-23 at 3 03 16 pm" src="https://media.github.ibm.com/user/54527/files/7f692eac-2eab-11e8-959b-864f57cd6988">


### Push a topic with replica

Push a topic within Kafka cluster (containing two Kafka servers) with two partitions and replicate partition over two Kafka broker. 

```sh
./bin/kafka-topics.sh --create --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --topic test-topic --partitions 2 --replication-factor 2

Created topic "test-topic".
```

Note that under the Kafka node's logs path, two partition has been created `test-topic-0` & `test-topic-1` on both nodes.

<img width="1002" alt="screen shot 2018-03-22 at 6 52 12 pm" src="https://media.github.ibm.com/user/54527/files/5875587e-2e02-11e8-8a2d-d1c148385728">

### Push a topic without replica

Push a topic within Kafka cluster (containing two Kafka servers) with two partitions and no replication. 

```sh
./bin/kafka-topics.sh --create --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --topic non-repeat-topic --partitions 2 --replication-factor 1

Created topic "non-repeat-topic".
```

You will observe below.

On Kafka node #1:

<img width="866" alt="screen shot 2018-03-22 at 7 02 30 pm" src="https://media.github.ibm.com/user/54527/files/e0fbd118-2e03-11e8-8194-859c4c23ca3f">

On Kafka node #2:

<img width="890" alt="screen shot 2018-03-22 at 7 02 45 pm" src="https://media.github.ibm.com/user/54527/files/e0da810c-2e03-11e8-9897-fb168af0496c">


### Get the list of topics

Get the list of topics on Kafka cluster.

```sh
./bin/kafka-topics.sh --list --zookeeper 9.30.42.237:2181 9.30.118.10:2181

test-topic
non-repeat-topic
```

### Describe topic 

Describe `test-topic` and `non-repeat-topic` topics.

```sh
./bin/kafka-topics.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --describe --topic test-topic
```

<img width="1153" alt="screen shot 2018-03-22 at 7 16 24 pm" src="https://media.github.ibm.com/user/54527/files/ac64bbe8-2e05-11e8-95ae-41e38fb7f4b9">

<img width="1149" alt="screen shot 2018-03-22 at 7 19 10 pm" src="https://media.github.ibm.com/user/54527/files/f04a8d60-2e05-11e8-921b-3cf4c6d4dde2">

```sh
Topic: test-topic	Partition: 0	Leader: 1	Replicas: 1,0	Isr: 1
``` 

represents Kafka broker with id `0` acting as leader and handling all reads/writes for partition `0`. `Replicas: 1,0` meaning partition `0` is replicated on broker id 0 and 1.   

### Delete topic 

Delete topic `test-topic` from Kafka cluster.

To enable deletion, we need to make sure to add `delete.topic.enable=true` on all Kafka brokers `server.properties` config file.

```sh
./bin/kafka-topics.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --delete --topic test-topic
```

### Run a producer

Run a producer process to publish data into `test-topic` Kafka topic within the Kafka broker.

```sh
bin/kafka-console-producer.sh --broker-list  9.30.118.212:9092,9.30.214.93:9092 --topic test-topic
>
```

### Create two consumer groups 

Edit `/kafka_2.11-1.0.1/config/consumer.properties` with group id `test-consumer-group-0` and `test-consumer-group-1` by creating another `consumer.properties1` config.

```sh
# consumer group id
group.id=test-consumer-group-1
```

### Run a consumer process

Run a consumer process to process published data from a topic.

```sh
bin/kafka-console-consumer.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --from-beginning --topic topic-new
```

### Run multiple consumer processes

First, run a producer process for pushing data to a `topic-new` topic.

```sh
bin/kafka-console-producer.sh --broker-list  9.30.118.212:9092,9.30.214.93:9092 --topic topic-new
```

Run two consumer processes in a consumer group.

```sh
bin/kafka-console-consumer.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --topic topic-new --consumer.config config/consumer.properties
```

Let's add messages in producer console as below:

<img width="1154" alt="screen shot 2018-03-23 at 5 23 59 pm" src="https://media.github.ibm.com/user/54527/files/07dce2ca-2ebf-11e8-801b-e51faeec6215">

This is how consumer processes would process messages through Kafka cluster.

<img width="1151" alt="screen shot 2018-03-23 at 5 24 43 pm" src="https://media.github.ibm.com/user/54527/files/4c2be110-2ebf-11e8-86f3-b821ee951b31">

<img width="1158" alt="screen shot 2018-03-23 at 5 24 55 pm" src="https://media.github.ibm.com/user/54527/files/537e6b22-2ebf-11e8-802e-939d8ae2ffeb">

**Note**:

Each consumer in a consumer group is guaranteed to read a particular message by only one consumer in the group. In other words, data pushed to a Kafka topic is only processed once in a consumer group. Or we can say, the processing of data is distributed among consumer processes in a consumer group. 

### Run multiple consumer groups

First, run a producer process for pushing data to a `cast-topic` topic.

```sh
bin/kafka-console-producer.sh --broker-list  9.30.118.212:9092,9.30.214.93:9092 --topic cast-topic
```

Run two consumer processes in two seperate consumer groups. We need two `consumer-properties` config files as covered [above](https://github.com/parminder7/kafka-sample/blob/master/README.md#create-two-consumer-groups).

```sh
bin/kafka-console-consumer.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --topic cast-topic --consumer.config config/consumer.properties
```

```sh
bin/kafka-console-consumer.sh --zookeeper 9.30.42.237:2181 9.30.118.10:2181 --topic cast-topic --consumer.config config/consumer.properties1
```

Now, put `[one, two, three, four, five, six]` messages in `cast-topic` through producer process. Following is how consumer processes would receive messages.

<img width="1153" alt="screen shot 2018-03-23 at 8 26 37 pm" src="https://media.github.ibm.com/user/54527/files/ebbe20d0-2ed8-11e8-84fa-0940d22e0dba">

<img width="1150" alt="screen shot 2018-03-23 at 8 26 30 pm" src="https://media.github.ibm.com/user/54527/files/ec0cdd06-2ed8-11e8-8343-f1f684cf01e3">

**Note**:

Here, consumers from different consumer group receive all the messages on a Kafka topic. Meaning, if multiple consumer groups subscribe to a topic, then Kafka would broadcast messages to each of them.


## Observation

- As we have seen during testing

- What happens if all the Kafka brokers died?

The consumer pulling data from topic gets `Error while fetching metadata with correlation id 43 : {test-topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)` error.

## Summary

Kafka is a publish-subscribe based messaging system that can be used to exchange data between processes, application and services. It has build-in partitioning, replication and fault-tolerance.

As we have already seen above, Kafka has capability to *scale-processing* (by distributing the data processing among consumer processes in a consumer group) and *multi-subscriber*  (by broadcasting messages among consumer groups).
