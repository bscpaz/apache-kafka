<h1 align="center">Apache Kafka</h1>
<p align="center">This is a POC (proof of concept) to understand better the behavior of Apache Kafka technology.</p>

See: https://kafka.apache.org/

## World of events

Everything is an event in a digitally modern world. Apache Kafka can handle a huge throughput of events and can store them for someone else to consume later. In other words, Kafka is able to receive events from a producer and save them for others (consumers) who are interested in using them.

#### Key points addressed by Kafka include:
* Where can I store events?
* How can I efficiently retrieve each event in real-time and send an acknowledgment of reading (ACK), even across different systems?
* How can I scale the throughput?
* How can I ensure resilience and high availability?
* How can I guarantee no message losses?

## Concepts of Apache Kafka

Kafka is composed of a set of machines, and each one is referred to as a 'broker.' Each broker has its own database where events are stored. Importantly, Kafka doesn't directly send messages to consumers; instead, it makes events available for others to consume.

![image](https://user-images.githubusercontent.com/9732874/190032601-a9eea95e-484f-4e7d-bb2a-80e1f6221afe.png)

The brokers continuously exchange messages among themselves to maintain awareness of the group members. Older versions of Kafka utilized Zookeeper for Service Discovery, but upcoming Kafka versions will incorporate an internally developed solution. Best practices recommend deploying a minimum of three brokers for Kafka to operate in a production environment.

### Topics
Events are systematically organized and durably stored in **topics**. In a simplified analogy, a topic can be compared to a folder in a filesystem, with the events acting as files within that folder. For instance, a topic might be named 'payments.' Essentially, a topic serves as the channel through which producers send events, and consumers read them. Multiple consumers can read events from the same topic, and it's possible for multiple consumers to read the same event. This differs from RabbitMQ, where once a message is read by a consumer, it cannot be read by any other consumer.

Topics in Kafka resemble a _log_, representing a sequence of events. Each event is assigned a number, ranging from 0 to n, known as an _offset_, essentially serving as an ID. Consumers have the flexibility to read any _offset_ of their choice, even revisiting older _offsets_. For example, an _offset_ that previously failed can be re-read for a new attempt. This is feasible because all events are persistently stored on disk, even if an event has already been consumed by a consumer—they remain accessible.

### Partitions
Topics in Kafka are partitioned, implying that a topic is distributed across a number of 'buckets' located on different Kafka brokers (_Don't Put All your Eggs in One Basket_ - giving us resilience). This strategic distribution of data is crucial for scalability, enabling client applications to concurrently read and write data from/to multiple brokers. When a new event is published to a topic, it is appended to one of the topic's partitions, with a _round robin_ algorithm employed when the "_key_" of the event is null.

![image](https://user-images.githubusercontent.com/9732874/192921645-1eb7e140-212f-40fa-8076-098505190350.png)

### Anatomy of a offset
![image](https://user-images.githubusercontent.com/9732874/190244353-98b05af6-7da4-4aa3-a743-bd2654f1ce50.png)

* Headers
  * Optional metadata.

* Key
  * When a new event is published to a topic, it is actually appended to one of the topic's partitions. Events with the same event **key** are written to the same partition, and Kafka guarantees that any consumer of a given topic-partition will always read that partition's events in **exactly the same order as they were written**. "Key" is important when the order of the events is important, like a _payment order_ in an account (offset 5) and then a _payment reversal_ (offset 7). That is, a consumer B (fast machine on partition 2) cannot read the _payment reversal_ before the consumer A (slow machine on partition 1) that is about to read the order of payment.

* Value
  * It is the payload of the event. Example: a JSON.

### Replication factor
Kafka replicates the log for each topic's partitions across a configurable number of servers (you can set this replication factor on a topic-by-topic basis). This allows automatic failover to these replicas when a server in the cluster fails so messages remain available in the presence of failures.

The unit of replication is the topic partition. Under non-failure conditions, each partition in Kafka has a single leader and zero or more followers. The total number of replicas including the leader constitute the **replication factor**. 

All writes are directed to the leader of the partition, while reads can be served by either the leader or the followers of the partition. Typically, there are many more partitions than brokers, and the leaders are evenly distributed among brokers. The logs on the followers are identical to the leader's log—they all have the same offsets and messages in the same order. However, it's worth noting that, at any given time, the leader may have a few as-yet unreplicated messages at the end of its log. Followers consume messages from the leader just as a normal Kafka consumer would and apply them to their own log.

![image](https://user-images.githubusercontent.com/9732874/192921136-f467b9ef-670a-4856-a7e9-845f4b843c60.png)

When the leader does die Kafka will choose a new leader from among the followers and a unique broker may have two leaders.  If a follower dies, gets stuck, or falls behind, the leader will remove it from the list of in sync replicas.

### Availability and Durability Guarantees
When writing to Kafka, producers can choose whether they wait for the message to be acknowledged by 0,1 or all (-1) replicas.

* ack=0: If set to zero then the producer will **not** wait for any acknowledgment from the server at all. The record will be immediately added to the socket buffer and considered sent. No guarantee can be made that the server has received the record in this case, and the retries configuration will not take effect (as the client won't generally know of any failures). The offset given back for each record will always be set to -1. This scenario is good when you need to process a huge of data and it's not critical if some of messages get loss. Example: position of car (GPS) in a Uber application.

* acks=1: This will mean the leader will write the record to its local log but will respond without awaiting full acknowledgement from all followers. In this case should the leader fail immediately after acknowledging the record but before the followers have replicated it then the record will be lost.

* acks=-1 (or acks=all): This means the leader will wait for the full set of in-sync replicas to acknowledge the record. This guarantees that the record will not be lost as long as at least one in-sync replica remains alive. This is the strongest available guarantee but it is slowest option.

### Message Delivery Guarantees
The semantic guarantees that Kafka provides between producer and consumer has multiple possible message delivery guarantees:

* _At most once_: Messages may be lost but are never redelivered (non-duplicated delivers). It has the better performance.
![image](https://user-images.githubusercontent.com/9732874/191397472-8a00438f-534a-4a40-b799-07189d429b16.png)

* _At least once_: Messages are never lost but may be redelivered (duplicated delivers). It has a mid perform and consumers may handle duplicated messages if it want.
![image](https://user-images.githubusercontent.com/9732874/191397993-a9f2d1dc-850b-4d61-b464-d0a36a652aa3.png)

* _Exactly once_: This is what people actually want, each message is delivered once and only once. Worst perform.
![image](https://user-images.githubusercontent.com/9732874/191398375-05c5efd1-3119-4e84-921f-60d66116ca31.png)

Kafka includes support for **idempotent** and transactional capabilities in the producer. Idempotent delivery ensures that messages are delivered exactly once to a particular topic partition during the lifetime of a single producer. That is, once the idempotent option is enabled, Kafka will discarts duplicated messages from producers but it may occurs more slowness. By the other hand, if the idempotent option is disabled, consumers may read duplicated messages but Kafka will be faster.

### Consumer's group

A consumer group is a set of consumers that share the same group id. When a topic is consumed by consumers in the same group, every partition will be read by only one consumer in that group. That is, "If all consumers instances have the same consumer group, then the records will effectively be load-balanced over the consumer instances". There is no way for two or more consumers in the same group reading the same partition.

This is an example that there is one consumer overloaded.
![image](https://user-images.githubusercontent.com/9732874/192923463-e0d4f9c5-02e4-4eb5-8678-5443a1de2bed.png)

It's a good practice to have the same amount of consumers and partitions, like below:
![image](https://user-images.githubusercontent.com/9732874/192924262-2634a51a-6b88-407b-a165-bb671d667b1f.png)

If there are more consumers in the same group than partitions, the excess consumers will remain idle:

![image](https://user-images.githubusercontent.com/9732874/192928613-433a3abb-c865-4d79-b97d-0221b5ee77f2.png)

However, if some consumer dies or falls ("Consumer 2"), the idle consumer ("Consumer 4") will replace the first one.
