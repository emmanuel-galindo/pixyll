---
published: true
title: Kafka JMS Client to Confluent Cloud example
layout: post
description: Example how to use Confluent's JMS Client Kafka library to connect from legacy systems
---

Confluent has created a JMS Client for that comes handy when working with legacy systems. Notably, it wraps a JMS 1.1 implementation and data flow goes like this:
```
[kafka-jms-client] <--- kafka protocol ---> [kafka broker]
``` 

It is important to note that compatibility with JMS 1.1 is pretty good but not complete. Preferably, read [https://docs.confluent.io/current/clients/kafka-jms-client/].


We are going to configure and run it against Confluent's Cloud. The code can be downloaded at: [https://github.com/emmanuel-galindo/kafka-jms-client]

As part of this exercise, we are going to create a producer that will create one message, and a consumer that if it is the first (from the consumer group) will download all message in the topic, if not will just use whatever offset the broker has. These clients are meant to be as much JMS compliant as possible, thus no use of specific logic, and the idea was to keep it simple. 

As a side note, I've found this exercise useful before jumping into trying to make legacy systems JMS implementation connect and interact with a topic in Confluent's Cloud. In addition to the challenge of configuring the JMS service in the legacy system, as Kafka basically works on Layer 4 ([https://en.wikipedia.org/wiki/OSI_model]), you could expect your company's proxy, firewall, vpn, etc to give you some trouble and executing this exercise could help you identifying what are the constraints in your network. 

## Requirements

If you haven't already go ahead and create a free cloud account.
You'll need to create a new topic (we are using TestTopic as name) and get some information to configure the client:
    - At API Access menu, issue an API Key and Secret to be used as username and password
    - In the Tools and client section
        - In Clients tab, at Java section, copy the properties
        - In Confluent Platform Components, at Schema Registry, create a new API Key and Secret

Then, you'll need to compile Confluent's Kafka JMS Client following [https://docs.confluent.io/current/clients/kafka-jms-client/installation.html#appendix-1-creating-a-shaded-fat-jar]. As it is mentioned there, to avoid namespace conflicts, it adds the confluent.shaded. preffix. We'll see this when configuring the login class.

After that, go ahead and grab a jar for the JMS 1.1. I'd normally refer to Maven central, but it looks there's some legal battle around this. I've found [https://repository.jboss.org/maven2/javax/jms/jms/1.1/] to have a working copy. 

## Installation

Create a new working dir, or clone the repo above, and create a new lib/ folder. In there, copy the two jar files.

### Producer

Create a Producer.java file and copy below code in it. As you can see, it is a pretty standard JMS implementation. I am importing here using wildcard to make it short.

```java
import javax.jmx.*;
import javax.naming.*;

public class Producer {
    public static void main(String[] args) throws JMSException, NamingException {
        Context ctx = new InitialContext();
        ConnectionFactory connectionFactory = (ConnectionFactory)ctx.lookup("ConnectionFactory");
        Connection connection = connectionFactory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = (Queue)ctx.lookup("TestTopic");

        MessageProducer producer = session.createProducer(queue);
        TextMessage message = session.createTextMessage();
        message.setText(new Date().toString() + " - Text message");
        producer.send(message);
    }
}
```

<BR/>

### Consumer
In this case, a Consumer.java needs to be created with this content.

```java
import javax.jmx.*;
import javax.naming.*;

public class Consumer {
    public static void main(String[] args) throws JMSException, NamingException {
        Context ctx = new InitialContext();
        ConnectionFactory connectionFactory = (ConnectionFactory)ctx.lookup("ConnectionFactory");
        Connection connection = connectionFactory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = (Queue)ctx.lookup("TestTopic");

        MessageConsumer consumer = session.createConsumer(queue);
        while(true) {
            TextMessage rmessage = (TextMessage) consumer.receive();
            System.out.println("Message received " + rmessage.getText());
        }
    }
}
```

<BR/>

### jndi.properties 
Create a new file jndi.properties at the same folder as the consumer and producer. Here is where the specifics for Kafka and Confluent Cloud are set:

```
#Kafka Cloud settings
bootstrap.servers = your_cloud_server.confluent.cloud:9092
security.protocol = SASL_SSL
#Calling the shaded version of the fat jar. Remove confluent.shaded when calling the thin jar
sasl.jaas.config = confluent.shaded.org.apache.kafka.common.security.plain.PlainLoginModule   required username="USERNAME123"   password="PASSWORD123"; 
ssl.endpoint.identification.algorithm = https
sasl.mechanism = PLAIN
# when there's no offset for this consumer, lets show a lot =)
auto.offset.reset = earliest
consumer.group.id = jms-client-test
queue.TestTopic = TestTopic

# Cloud Schema Registry settings
basic.auth.credentials.source = USER_INFO
schema.registry.basic.auth.user.info = USER345:PASSWORD345
schema.registry.url = https://your_cloud_registry.confluent.cloud

# JMS settings
java.naming.provider.url = tcp://your_cloud_server.confluent.cloud:9092
java.naming.factory.initial = io.confluent.kafka.jms.KafkaInitialContextFactory
client.id = jms-client-test-id
#register topics in JNDI. format: [topic|queue].[name] = [name] 
```

- bootstrap.servers and java.naming.provider.url has the same value
- modify the USERNAME123 and PASSWORD123 with the API key and secret from Confluent Cloud
- auto.offset.reset is an example of a kafka property extended to this JMS client. 
- consumer.group.id. If you implement this as we did above, we are letting the JMS client library know we want to use queues. In that case, the consumer group in the broker will be set to that value. As stated in the docs, if we were using topics, the lib would add a uuid to the end of the topic name and each consumer seems to end up having a sepparate consumer group. This is a very nice way to emulate what you normally experience with topics and queues in JMS.
- queue.TestTopic, defines what is the kafka topic we are going to work with. In Kafka, both JMS topics and queues are implemented as Kafka topics and I believe the intention of this library is try to emulate a normal JMS behavior. If we would like to do JMS pub/sub, we would then change the implementation at the code to use JMS Topic and here we would call this property topic.TestTopic
- I haven't tested extensively the schema registry, so please free to share what you find

## Executing 

We are using standalone java here. In Linux you would execute this:
```shell
$ javac -cp lib/*:. Consumer.java
$ java -cp lib/*:. Consumer
```
In Windows
```shell
$ javac -cp "lib/*;." Consumer.java
$ java -cp "lib/*;." Consumer
```
And the same for the Producer.

<br />

First execute the producer, and then the consumer. You can leave the consumer, or start a new one in another shell session. The producer will just send one message and you can ctrl+c after that.

When executing the clients you should expect below output (it might take one minute or two to spin up the client and connect to the broker, feel free to add some print lines):

Producer:
```shell
Message sent
```

Consumer:
```shell
Message received Tue Aug 11 19:12:49 ART 2020 - Text message
```

A nice test, is to shutdown the consumer, send another email and as you would expect from queues, when you spin up the consumer again you should receive the message. 

Another interesting one, is to modify the consumer group to a new one and it shoud download all messages. 