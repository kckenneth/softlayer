
|Title |  Softlayer Application |
|-----------|----------------------------------|
|Author | Kenneth Chen, Ph.D |
|Utility | |
|Date | 9/10/2018 |

__Synopsis__  

   This is a walk through for how to set up softlayer via CLI. 
   
<p align="center">
<img src="img/before_kafka.png" width="600"></p>
<p align="center">Figure 1. Data Communications</p>

__Dockerfile__  

```
$ cd ~
$ mkdir softlayer
$ cd softlayer
$ vi Dockerfile
```
click i to insert the following text. I changed YOUR_SL_API_ID and YOUR_SL_API_KEY accordingly.

```
FROM ubuntu:16.04

RUN apt-get update 
RUN apt-get install -y \
    python \
    python-pip \
    python-setuptools \
    python-dev \ 
    openssh-client
RUN pip install SoftLayer
RUN echo '[softlayer]' > ~/.softlayer
RUN echo 'username = YOUR_SL_API_ID' >> ~/.softlayer
RUN echo 'api_key = YOUR_SL_API_KEY' >> ~/.softlayer
RUN echo 'endpoint_url = https://api.softlayer.com/xmlrpc/v3.1/' >> ~/.softlayer
RUN ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa -q -N ""

ENTRYPOINT ["/bin/bash"]
```

Since we're using the Ubuntu and we don't need any dependencies (eg. Redis) and web app (eg. Flask), we just created Dockerfile and build the image on it. `-t` or `--tag`. 
```
$ docker build -t slcli --file Dockerfile
```

To overcome the latency issue and impact on real time data analytics, our team deployed kafka messaging system (open-source) in our test centers. Kafka serves as a central warehouse from which several service centers retrieve (consume) or immediately feed (produce) their real-time data for instant implementation. The goal of our data scientist team is to facilitate test takers exams schedules, adjust their exam questions based on their real-time answers and instant scoring system. Kafka facilitates all of our needs in our data warehouse and provides an efficiency and timeliness (Figure 2).  

<p align="center">
<img src="img/after_kafka.png" width="500"></p>
<p align="center">Figure 2. Kafka Real Time Data Communications</p>

__Lambda Architecture__  

By implementing a real time data analytics by kafka, our team establish more efficient and powerful database management system, known as **Lambda Architecture (LA)** coined by Nathan Marz. Deploying two parallel warehouses; batch layer and speed layer, we are now able to tackle big data as well as provide real time answers to our potential students and test takers (Figure 3).  

<p align="center">
<img src="img/LA.png" width="700"></p>
<p align="center">Figure 3. Lambda Architecture</p>

__Procedure__  

We first deployed several docker containers: each equipped with essential libraries for necessary applications such as kafka containers for kafka applications and midsw205/spark-python container for spark applications. By doing so eliminates conventional cumbersome heavy weight instantiation of virual machine (VM) with unnecessary libraries and applications. In order to successfully load our containers, we listed them in docker-compose.yml file. Once all containers are up and running, we double-checked each container logs file to make sure they have already started running. 

In next step, we launched Kafka and created a topic. Data from json file was released in Kafka producer mode. We then piped our Kafka topic into spark and analyzed the data. All data engineered in spark was later stored in HDFS. 

I laid out a step by step implementation of Kafka in details in the following.  

## In Droplet, Update images 
### 1. Updating Docker Images 
```
docker pull midsw205/base:latest   
docker pull confluentinc/cp-zookeeper:latest  
docker pull confluentinc/cp-kafka:latest  
docker pull midsw205/spark-python:0.0.5  
```
### 2. Logging into the assignment folder
```
cd w205/assignment-07-kckenneth/
```
### 3. Checking what's in my directory 
```
ls  
```

### 4. Making sure at which branch I am on git
```
git branch   
```
### 5. Checking if there's any pre-existing docker containers running
```
docker-compose ps  
docker ps -a  
```
   ##### if need be, remove any running containers by rm
```
docker rm -f $(docker ps -aq) 
```
### 6. Run a single docker container midsw205 in bash mode
```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```
## Inside the Docker Container
1. check into assignment 7 folder,
2. check git branch, create assignment branch if necessary  
3. download json file,  
4. create docker-compose.yml  

```
cd assignment-07-kckenneth  
ls  
git status  
git branch 
git checkout -b assignment  
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/f5bRm4  
vi docker-compose.yml  
exit  
```
## In Droplet, I spin up the cluster in detached mode by -d
```
docker-compose up -d
```
### Check if the zookeeper is up and running by finding the *binding* word in the logs file
```
docker-compose logs zookeeper | grep -i binding  
```
### I also checked kafka is up and running by searching the word *started*
```
docker-compose logs kafka | grep -i started
```
## I. Kafka 1st step -- Create a Topic
#### I created a topic *exams* with partition 1, replication-factor 1
```
docker-compose exec kafka kafka-topics --create --topic exams --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
#### I checked the broker I just created by *describe* function  
```
docker-compose exec kafka kafka-topics --describe --topic exams --zookeeper zookeeper:32181
```  
## II. Kafka 2nd step -- Produce Messages 
#### Since we have added midsw205 image in our cluster, we can directly pipe our json file query (mids image) into kafka producer (kafka image)[Cluster magic]
```
docker-compose exec mids bash -c "cat /w205/assignment-07-kckenneth/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t exams && echo 'Produced EXAM messages.'"
```
## III. Kafka 3rd step -- Consume Messages
- (1) We can consume Kafka messages independently as follows:  
```
docker-compose exec kafka kafka-console-consumer --bootstrap-server kafka:29092 --topic exams --from-beginning --max-messages 42
```
- (2) With Apache Spark container, we can directly pipe our Kafka messages into Spark and analyze the data in Spark  
```
docker-compose exec spark pyspark  
```  
## In Spark Environment  
1. I first assigned the Kafka messages as "messages" object.  
2. I counted the number of messages which counted at 3280 entries.  
3. I also checked the first 20 rows. 
4. Printing the format shows that Kafka messages in "binary" format. 
5. I transformed them into strings format.  

```
>>> messages = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","exams").option("startingOffsets", "earliest").option("endingOffsets", "latest").load()  
>>> messages.count()  
>>> messages.show()   
>>> messages.printSchema()   
>>> messages_as_strings = messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")  
```
