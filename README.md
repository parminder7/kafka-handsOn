# Set up a complete Kafka environment

Here, we have provisioned two kafka nodes with two zookeeper nodes. 

## Pre-requisite

- Create 4 linux based VM/machines. (Ubuntu 17.10, CPU 4 Core, Memory 8GB, Disk 250GB) 

## Step #1: Install JAVA on all machines

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

## Step #2: Install Kafka 

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

## Step #3: Configure Kafka nodes

After downloading Kafka, the next step is to change/update some configuration on Kafka.

- Edit `/kafka_2.12-1.0.1/config/server.properties`

```sh
	zookeeper.connect=9.30.42.237:2181,9.30.118.104:2181
	log.dirs=/tmp/kafka-logs-0
	broker.id=0
```



