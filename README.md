# Microservices.Ticketing.WebApi.Kafka

Using Apache Kafka with .NET


Have you ever used async processing for your applications? Whether for a web-based or a cloud-driven approach, asynchronous code seems inevitable when dealing with tasks that do not need to process immediately. Apache Kafka is one of the most used and robust open-source event streaming platforms out there. Many companies and developers take advantage of its power to create high-performance async processes along with streaming for analytics purposes, data integration for microservices, and great monitoring tools for app health metrics. This article explains the details of using Kafka with .NET applications. It also shows the installation and usage on a Windows OS and its configuration for an ASP.NET API.

How It Works
The world produces data constantly and exponentially. To embrace such an ever-growing amount of data, tools like Kafka come into existence, providing robust and impressive architecture.

But how does Kafka work behind the scenes?

Kafka works as a middleman exchanging information from producers to consumers. They are the two main actors in each edge of this linear process.


![image](https://user-images.githubusercontent.com/29863643/201304623-6a0ffce8-f266-4758-ab88-8731f11e4bec.png)

Kafka can also be configured to work in a cluster of one or more servers. Those servers are called Kafka brokers. You can benefit from multiple features such as data replication, fault tolerance, and high availability with brokers.

![image](https://user-images.githubusercontent.com/29863643/201304940-d4a3617b-8998-437a-b6ab-e78e36b494e7.png)

These brokers are managed by another tool called Zookeeper. In summary, it is a service that aims to keep configuration-like data synchronized and organized in distributed systems.

Kafka Topics
Kafka is just the broker, the stage in which all the action takes place. The producers send messages to the world while the consumers read specific chunks of data. How do you differentiate one specific portion of data from the others? How do consumers know what data to consume? To understand this, you need a new actor in the play: the topics.

Kafka topics are the channels, the carriage that transport messages around. Kafka records produced by producers are organized and stored into topics.

Imagine that you’re working on a new API project for a catalog of trees and plants. You want to make sure that everybody in the company has access to each newly registered tree. And that is why you picked Kafka.

Every new tree registered within the system is going to be broadcasted via Kafka. The name of the topic is tree_catalog.

![image](https://user-images.githubusercontent.com/29863643/201306148-13666f39-a365-4fed-9326-77aaf96d43ae.png)

In this relationship, the topics work as a stack. It keeps information in the same position as it arrived and guarantees no data loss.

Each data record that arrives is stored in a slot and registered with a unique position number called offset.

When a consumer consumes a message stored in offset 0, for example, it commits the message stating that everything was ok and moves on to the next offset, and so on. The process usually happens linearly. However, since many consumers can “plug” into the same topic simultaneously, the responsibility of knowing which data positions were already consumed is left to the consumers. That means that consumers can decide which order they will consume the messages, or even if they want to restart the processing from scratch (offset 0).

![image](https://user-images.githubusercontent.com/29863643/201306511-606d175c-1458-4174-bbd0-7c2be921f0a1.png)


Topic Partitions
One key feature of distributed systems is data replication. It allows for a more secure architecture since the data is replicated somewhere else in case bad things happen. Kafka deals with replication via partitions. Kafka topics are configured to be spread among several partitions (configurable). Each partition holds data records via unique offsets.

To achieve redundancy, Kafka creates replicas from the partitions (one or more) and spreads the data through the cluster.

This process follows the leader-followers model in which one leader replica always deals with the requests for a given partition while the followers replicate it. Every time a producer pushes a message to a topic, it goes directly to that topic leader.


Consumer Groups
The most appropriate way to consume messages from a topic in Kafka is via consumer groups.

As the name suggests, these groups are formed by one or more consumers that aim to get all the messages from a specific topic.

To do this, the group must always have a unique id (set by the property group.id). Whenever a consumer wants to join that group, it will do so via group id.


![image](https://user-images.githubusercontent.com/29863643/201307373-54642870-15e2-4237-81f6-b4df1b74f60d.png)

Every time you add or remove consumers to a group, Kafka will rebalance the load among them so that no overhead happens.

Setup
Now that you know the basics of how Kafka universally works, it’s time to move on to the environment setup. To simplify, the example will use Docker to hold the Kafka and Zookeeper images rather than installing them on your machine. This way, you save some space and complexities.

For Windows users, Docker provides a simple way to install and manage your Docker containers: the Docker 
https://docs.docker.com/desktop/install/windows-install/ 
Desktop. Go to its download page and download the installer. Run it and proceed to the end without changing the default setup options (Figure 6).

![image](https://user-images.githubusercontent.com/29863643/201307759-0d41a9ab-6934-4498-8df7-831ec5be5c02.png)

Make sure to restart your computer after the process is done. After the restart, Docker may ask you to install other dependencies so make sure to accept every one of them.

One of the fastest paths to have a valid Kafka local environment on Docker is via Docker Compose. This way, you can set up a bunch of application services via a YAML file and quickly get them running.

From a code editor (Notepad++, Visual Studio Code, etc.), create a new file called docker-compose.yml and save the contents of Listing 1 into it.

```ruby
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_CREATE_TOPICS: "simpletalk_topic:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
Notice the code imports two service images (kafka and zookeeper) from the Docker Hub’s account called wurstmeister. This is one of the most stable images when working with Kafka on Docker. The ports are also set with their recommended values, so be careful not to change them.

One of the most important settings of this listing belongs to the KAFKA_CREATE_TOPICS config. Here’s the place where you must define your topic name to be automatically created.

There are other ways to create topics, which you’ll see in the future.

Navigate via the command line to the folder where you saved the docker-compose.yml file. Then, run the following command to start the images:


```ruby
docker-compose up
```
This code will start the download of all the dependencies and start the images. During the process, you may see a lot of logs. If no ERROR logs are displayed, then the startup was successful.

Note: You must leave the docker-compose cmd window open, otherwise the images will go down.

To check if the Docker images are up, run the following command in another cmd window:


```ruby
docker ps
```

![image](https://user-images.githubusercontent.com/29863643/201309502-7e54dc4b-7427-498e-abbc-a4d627853d5f.png)

Hands-on
Great! Your Kafka environment is ready to be used. The next step is to jump into the project creation at Visual Studio (VS). Open VS, and head to the project creation window. Search for the ASP.NET Core

![image](https://user-images.githubusercontent.com/29863643/201309771-a692e7e4-f916-45d5-8cd3-c2507a53f01d.png)


In the following window, make sure to give the solution a different name, as shown in Figure 9. This is because both consumer and producer projects will coexist within the same solution.


```ruby
{
   
}
```
