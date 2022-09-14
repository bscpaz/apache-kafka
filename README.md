<h1 align="center">Apache Kafka</h1>
<p align="center">This is a POC (proof of concept) to understand better the behavior of Apache Kafka technology.</p>

See: https://kafka.apache.org/

#### World of events

Everything is an event in a digital modern world. Apache Kafka can deal with a huge throughput of event and it can store then for someone else to consume it. That is, Kafka is able to get events from a producer and save them for others (consumers) that are interested to use them.

##### Key points that Kafka answers:
* Where can I save the events?
* How can I quickly (real time) recover each event and send an ACK of reading, even between different systems?
* How can I scale the throughput?
* How can I have resilience and high availability?
* How can I guarantee no messages losses?

#### Concepts of Apache Kafka

Kafka is compound with a set of machines and each one is called as "broker". Each broker has its own database wich the events are storage. A important thing to know is that Kafka doesn't send messages to consumers, but only put the events available for someone to consume them.

![image](https://user-images.githubusercontent.com/9732874/190032601-a9eea95e-484f-4e7d-bb2a-80e1f6221afe.png)

The brokers exchange message among them all the time to know who is part of the group. Olders version of Kafka make use of Zookeeper as Service Discovery, but newest version of Kafka will built its own solution. The best practices says that for productions put at least 3 brokers to oparate the Kafka.

#### Topics
Topics, in Kafka, is the channel that producers send events and consumres read then and we can have many consumers reading events from the same topic and reanding the same event. This is different from RabbitMQ which once a message got read from a consumer no one can read it again.

Topics in Kafka are like a _log_ which is a sequence of events. Each events receive a number (from 0 to n) thats is called _offset_ (a kind of an ID). Each consumer can read the _offset_ that it likes even olders _offsets_ (i.e. an offset that failed can be read again for a new try). This is possible as all events are stored in disk even if that event was already reading by a consumer.
