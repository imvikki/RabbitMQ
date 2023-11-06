# RabbitMQ

## TraceId not shown after springBoot 3 :

### Problem Statement :

  When migrated springBoot from 2.x to 3.x, there were missing traceId only for the logs that generated while publishing via exchange.

### Solution :

  Usage of brave tracer, solved this problem. This solves the below scenarios too

**Scenarios** :
  1. If a message is getting created synchronously as part of an API call, 
      a. the traceId from incoming request should get propagated if present
      b. new traceId should be generated if not present
  2. If a message is getting created in response to an incoming message, 
      a. the traceId from incoming message should get propagated if present
      b. new traceId should be generated if not present
  3. If a message is getting created in response to a scheduled job (or similar),
      a. the traceId from job should get propagated if present
      b. new traceId should be generated if not present

```

@Bean
  public Tracing tracing() {
    return Tracing.newBuilder()
      .currentTraceContext(ThreadLocalCurrentTraceContext.newBuilder()
        .addScopeDecorator(MDCScopeDecorator.get())
        .build())
      .build();
  }

  @Bean
  public MessagingTracing messagingTracing(Tracing tracing) {
    MessagingTracing messagingTracing = MessagingTracing.create(tracing);
    messagingTracing.tracing().setNoop(true);
    return messagingTracing;
  }

  @Bean
  public SpringRabbitTracing springRabbitTracing(MessagingTracing messagingTracing) {
    return SpringRabbitTracing.newBuilder(messagingTracing)
      .build();
  }

```

### Disabled extra log from barve trace library

```
messagingTracing.tracing().setNoop(true);
```

## Introduction :
RabbitMQ is an extremely popular open-source Message Broker that is used for building message-based systems. The key elements that constitute RabbitMQ — Producers, Consumers, Exchanges, Bindings and Queues.

![image](https://github.com/imvikki/RabbitMQ/assets/41563822/03dc89d9-edbe-4d29-bee9-978f20fa6b1e)


The applications that generate messages are called Producers. These messages are published to the RabbitMQ Exchange. The Exchange, depending on its configuration, forwards the messages to one or more Queues which act as buffers for storing messages. The link between the Exchange and the Queue is called Binding. The applications that consume from the Queues are called Consumers. It is important to note that once the Producer publishes a message to the Exchange, it does not wait for a response i.e, it is not blocked. This form of communication is facilitated by the Advanced Message Queueing Protocol or AMQP.


## Advanced Message Queueing Protocol (AMQP)
Though RabbitMQ supports several messaging protocols, the most popular one is the Advanced Message Queueing Protocol (AMQP). AMQP is a binary protocol. Contrary to text protocols such as HTTP, where the information exchanged between the client and the server is in human-readable text format, AMQP exchanges information (messages) in a binary format. Instead of relying on different fields, the participating entities agree upon a certain structure and find the required information at specific positions in the transmitted binary data.

The binary data is transmitted as frames, as shown in img-3. The first byte stores the type of the frame and can assume the value HEADER, METHOD, BODY or HEARTBEAT. The next byte stores the channel which identifies an independent thread of messages. Although the client establishes only a single TCP connection with the Message Broker, the connection is multiplexed — contrary to HTTP/1.1. This means that the client and the message-broker can utilise the same TCP connection to transmit multiple independent threads of messages. The next byte indicates the size of the Payload. The first three bytes are collectively called the frame header. The Payload holds data corresponding to the type of the frame. For example, if the type is METHOD, the payload will contain the name of the operation to be performed. If the type is BODY, it will contain the message data that the sender is transmitting.

## Major components in AQMP include :

**Queues:** A queue is a message buffer that stores messages until they can be processed by the receiving application. RabbitMQ supports different types of queues, such as durable, non-durable, and mirrored queues, which provide different levels of reliability and availability.

**Exchanges:** An exchange is a message-routing component that receives messages from producers and routes them to the appropriate queue(s) based on message properties and routing rules. RabbitMQ supports different types of exchanges, such as direct, fanout, etc. which help provide different routing patterns.

**Bindings:** A binding is a connection between an exchange and a queue, specifying the routing rules that determine how messages are routed from the exchange to the queue.

## Why use RabbitMQ?
**RabbitMQ is a popular message broker for several reasons:**

**Ease of Use:** RabbitMQ is easy to install, configure, and use. It has a simple and intuitive interface that allows developers to quickly set up and start using it. It provides a web-based interface that makes it easy to monitor queues, connections, exchanges, and other components. It also provides support for logging and monitoring tools such as Prometheus.

**Reliable Messaging:** RabbitMQ is a highly reliable message broker that provides features such as message persistence and delivery confirmation. It also has built-in mechanisms for handling message failures and ensuring message delivery. It is designed to be reliable and fault-tolerant.

**High Interoperability:** RabbitMQ is designed to work with a variety of programming languages, platforms, and messaging protocols, making it a flexible and interoperable solution for different environments. RabbitMQ supports multiple messaging protocols like AMQP, MQTT.

**Scalable:** It can handle high volumes of messages and can scale horizontally to accommodate additional message traffic.

**Powerful Routing mechanism:** It provides a powerful routing mechanism to route messages to different queues based on routing keys, message headers, and other attributes. It provides various other features, such as message transformation, routing, and filtering.
