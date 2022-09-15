<h1 align="center">Apache Kafka</h1>
<p align="center">This is a POC (proof of concept) to understand better the behavior of Apache Kafka technology.</p>

See: https://kafka.apache.org/

## World of events

Everything is an event in a digital modern world. Apache Kafka can deal with a huge throughput of events and it can store then for someone else to consume then. That is, Kafka is able to get events from a producer and save them for others (consumers) that are interested to use them.

#### Key points that Kafka answers:
* Where can I save the events?
* How can I quickly (real time) recover each event and send an ACK of reading, even between different systems?
* How can I scale the throughput?
* How can I have resilience and high availability?
* How can I guarantee no messages losses?

## Concepts of Apache Kafka

Kafka is compound with a set of machines and each one is called as "broker". Each broker has its own database which the events are stored. A important thing to know is that Kafka doesn't send messages to consumers, but only put the events to be available for someone else to consume them.

![image](https://user-images.githubusercontent.com/9732874/190032601-a9eea95e-484f-4e7d-bb2a-80e1f6221afe.png)

The brokers exchange message among them all the time to know who is part of the group. Olders versions of Kafka make use of Zookeeper as Service Discovery, but newest versions of Kafka will built its own solution. The best practices says that for productions environment put at least 3 brokers to oparate the Kafka.

### Topics
Events are organized and durably stored in **topics**. Very simplified, a topic is similar to a folder in a filesystem, and the events are the files in that folder. An example topic name could be "payments". That is, a topic is the channel where producers send events and consumres read then and we can have many consumers reading events from the same topic and reanding the same event. This is different from RabbitMQ which once a message got read from a consumer no one can read it again.

Topics in Kafka are like a _log_ which is a sequence of events. Each events receive a number (from 0 to n) thats is called _offset_ (a kind of an ID). Each consumer can read the _offset_ that it likes even olders _offsets_ (i.e. an offset that failed can be read again for a new try). This is possible as all events are stored in disk even if that event was already read by a consumer (they are still there).

### Partitions
Topics are partitioned, meaning a topic is spread over a number of "buckets" located on different Kafka brokers (_Don't Put All your Eggs in One Basket_ - giving us resilience). This distributed placement of your data is very important for scalability because it allows client applications to both read and write the data from/to many brokers at the same time. When a new event is published to a topic, it is actually appended to one of the topic's partitions (_round robin_ algorithm when the "_key_" of the event is null).

![image](https://user-images.githubusercontent.com/9732874/190252253-cb86d6ae-a148-4363-972a-169258315d4f.png)

### Anatomy of a offset
![image](https://user-images.githubusercontent.com/9732874/190244353-98b05af6-7da4-4aa3-a743-bd2654f1ce50.png)

* Headers
  * Optional metadata.

* Key
  * When a new event is published to a topic, it is actually appended to one of the topic's partitions. Events with the same event **key** are written to the same partition, and Kafka guarantees that any consumer of a given topic-partition will always read that partition's events in **exactly the same order as they were written**. "Key" is important when the order of the events is important, like a _payment order_ in an account (offset 5) and then a _payment reversal_ (offset 7). That is, a consumer B (fast machine on partition 2) cannot read the _payment reversal_ before the consumer A (slow machine on partition 1) that is about to read the order of payment.

* Value
  * It is the payload of the event. Example: a JSON.

