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


Add empty solution then add 2 api projects 1 for producer 2 for consumer
![image](https://user-images.githubusercontent.com/29863643/201341299-e1382740-f7bf-4fac-8932-6e75da30efb6.png)
 # NuGet Packages
 
 To make your C# code understand how to produce and consume messages, you need a client for Kafka. The most used client today is Confluent’s Kafka .NET Client.

To install it, right-click the solution and select the Manage NuGet Packages for Solution… option. Type Confluent in the search box and select the Confluent.Kafka option, as shown 

Select both projects and click Install. Alternatively, you can add them via command line:


```ruby
PM> Install-Package Confluent.Kafka
```

![image](https://user-images.githubusercontent.com/29863643/201341961-c577c531-67f4-4dc2-ad97-575684147dd7.png)

Setting Up the Consumer
Now to implement the consumer project. Although it is a REST-like application, the consumer is not required. You can have any type of .NET project listening to topic messages.

The project already contains a Controllers folder. You need to create a new one called Handlers and add a new class called KafkaConsumerHandler.cs to it.

Listing 2 shows the new class code content.

This handler must run in a separate thread since it will eternally watch for incoming messages within a while loop. Therefore, it’s making use of Async Tasks in this class.

Pay attention to the topic name and consumer configs. They match exactly what was set in your docker-compose.yml file. Make sure to double-check your typing here, since divergences can lead to some blind errors.

```ruby
using Confluent.Kafka;
using Microsoft.Extensions.Hosting;
using System;
using System.Threading;
using System.Threading.Tasks;
namespace ST_KafkaConsumer.Handlers
{
    public class KafkaConsumerHandler : IHostedService
    {
        private readonly string topic = "simpletalk_topic";
        public Task StartAsync(CancellationToken cancellationToken)
        {
            var conf = new ConsumerConfig
            {
                GroupId = "st_consumer_group",
                BootstrapServers = "localhost:9092",
                AutoOffsetReset = AutoOffsetReset.Earliest
            };
            using (var builder = new ConsumerBuilder<Ignore, 
                string>(conf).Build())
            {
                builder.Subscribe(topic);
                var cancelToken = new CancellationTokenSource();
                try
                {
                    while (true)
                    {
                        var consumer = builder.Consume(cancelToken.Token);
                        Console.WriteLine($"Message: {consumer.Message.Value} received from {consumer.TopicPartitionOffset}");
                    }
                }
                catch (Exception)
                {
                    builder.Close();
                }
            }
            return Task.CompletedTask;
        }
        public Task StopAsync(CancellationToken cancellationToken)
        {
            return Task.CompletedTask;
        }
    }
}

```
The consumer group id can be anything you want. Usually, they receive intuitive names to help with maintenance and troubleshooting.

Whenever a new message is published to the simpletalk_topic topic, this consumer will consume it and log it to the console. Of course, in real-world apps, you’d make better use of this data.

You also need to add this hosted service class to the Startup class. So, open it and add the following code line to the ConfigureServices method:

services.AddSingleton<IHostedService, KafkaConsumerHandler>();
Make sure to import the proper using at the beginning of the class as well:

# Setting Up the Producer
Moving on to the producer, things will be handled a bit differently here. Since there’s no need for infinite loops to listen to messages arriving, the producers can simply publish messages from anywhere, even from the controllers. In a real application, it’d be better to have this type of code separate from the MVC layers, but this example sticks to the controllers to keep it simple.

Create a new class called KafkaProducerController.cs in the Controllers folder and add the content of Listing 3 to it.









```ruby
using System;
using Confluent.Kafka;
using Microsoft.AspNetCore.Mvc;
namespace Kafka.Producer.API.Controllers
{
    [Route("api/kafka")]
    [ApiController]
    public class KafkaProducerController : ControllerBase
    {
        private readonly ProducerConfig config = new ProducerConfig 
                             { BootstrapServers = "localhost:9092" };
        private readonly string topic = "simpletalk_topic";
        [HttpPost]
        public IActionResult Post([FromQuery] string message)
        {
            return Created(string.Empty, SendToKafka(topic, message));
        }
        private Object SendToKafka(string topic, string message)
        {
            using (var producer = 
                 new ProducerBuilder<Null, string>(config).Build())
            {
                try
                {
                    return producer.ProduceAsync(topic, new Message<Null, string> { Value = message })
                        .GetAwaiter()
                        .GetResult();
                }
                catch (Exception e)
                {
                    Console.WriteLine($"Oops, something went wrong: {e}");
                }
            }
            return null;
        }
    }
}

```

The producer code is much simpler than the consumer code. Confluent’s ProducerBuilder class takes care of creating a fully functional Kafka producer based on the provided config options, Kafka server, and topic name.

It’s important to remember that the whole process is asynchronous. However, you can use Confluent’s API to retrieve the awaiter object and then the result that will be returned from the API method. The Message object, in turn, encapsulates the message key and content, respectively. Keys are optional in topics, while the content can be of any type (provided in the second generic argument).

Testing
To test this example, you need to run the producer and consumer applications separately. In the upper toolbar, locate the Startup Projects combo box and select the Consumer project

![image](https://user-images.githubusercontent.com/29863643/201344285-34d5cea9-5314-4e47-bb78-50f2db81cf5f.png)


Pay attention to the URL and port in which it is running.

Now is time to send some messages via producer API. To do this, you can use any API testing tool such as Postman, for example. However, this example will stick to cURL for the sake of simplicity.

For the following commands to work properly, you must ensure that the Docker images are working. So, before running into them, make sure to execute docker ps again to check that out. Sometimes, restarting the computer stops these processes.

![image](https://user-images.githubusercontent.com/29863643/201345500-f921d2dc-f29f-4b79-975c-6553a0d29995.png)

![image](https://user-images.githubusercontent.com/29863643/201345545-00be7011-c233-47dc-8171-c1164c2040aa.png)
If the command doesn’t log anything, run the docker-compose up once more.

To test the publish-subscribe message, open another cmd window and issue the following command:

```ruby
curl -H "Content-Length: 0" -X POST "http://localhost:51249/api/kafka?message=Hello,kafka!"


```

This request will arrive at the producer API and trigger the publish of a new message to Kafka.

To check if the consumer received it, you may locate the Output window and select the option ST-KafkaConsumer 


Using Apache Kafka with .NET
Pretty nice, isn’t it? What about you? Have you ever used Kafka for your projects? How was your experience with it?

Kafka is a flexible and robust tool that allows for strong implementations in many types of projects, reason number one why it is so widely adopted.

This article was just a brief introduction to its world, but there’s much more to see like Kafka Streams, working in the cloud, and more complex scenarios from the real world. In the next article, I’ll explore Kafka’s capabilities. Let’s go together!







[Consider supporting me by buying me a coffee](https://www.buymeacoffee.com/MAhmoudKhalifa)
![bmc_qr](https://user-images.githubusercontent.com/29863643/201290985-b519f0ee-a842-414b-b05e-63714b5b8ff3.png)

all the informations is collected from the open source projects and blogs :)
