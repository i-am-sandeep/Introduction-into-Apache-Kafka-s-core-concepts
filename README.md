# Introduction to Apache Kafka's Core Concepts

Kafka is a deceivingly simple piece of messaging infrastructure. In this article, I will explain how Kafka works aimed at new users of Kafka. Some understanding of messaging brokers will most certainly help.

---

## ❓ Why Even Use Kafka?

Kafka is a **persistent messaging broker** that is **highly customizable**. Some of the reasons why people choose Kafka is because they need:
* **Guaranteed delivery** of messages in order.
* **High availability** across regions.
* **Support** for distributing workloads.
* **Fast throughput** when handling large amounts of data.
* **Fault-tolerant** consumption of messages.

The nice thing about the Kafka infrastructure is that it has lots of configuration options, meaning you can tune Kafka to your specific workloads.

---

## ⚙️ What Does Kafka Do?

There are two types of applications that use Kafka:
1. Applications that want to **consume** some information.
2. Applications that would like to **distribute (produce)** some information.

### 📤 Producing an Event to Kafka
Kafka uses **Topics** as its messaging concept to group related messages.

When a producer publishes something to Kafka, it provides a few pieces of information:
* **Topic Name**
* **ID of the Message**
* **The Message Payload itself**

If all three pieces of information are provided, Kafka will **guarantee** that all messages with the same ID end up on the **same partition** of the topic.

---

## 🧩 Wait, What Are Partitions?

A partition is Kafka’s way of **parallelizing topics** so that we can distribute the processing of messages between consumers to handle various amounts of load. 

### 📐 Configuration Best Practices
* **Scale up early:** The number of partitions is configurable for the topic. However, it is not easy to split partitions after they are created without causing potential inconsistencies in the data. 
* **Rule of thumb:** It’s best to create more partitions than you think you need. Think about how many consumers for a single application you expect to run, and then **multiply that number by 2**.

### 🔍 Diving Deeper With an Example
Imagine an online shop that sells different products. A product might go through a number of reprices in its lifecycle. 

Let's say we have a system where other applications need to be made aware of when a product has changed in price. Kafka is a good choice for this. Consider a producer that generates the following messages in order:
1. A **Product Created** event for *"White shirt"* | price `$15.00` at `07:00`
2. A **Product Reprice** event for *"White shirt"* | price `$35.00` at `08:00:00`
3. A **Product Reprice** event for *"White shirt"* | price `$13.00` at `08:00:01`

#### 🟢 Scenario A: The Producer Provides Message IDs
This works perfectly. Because the messages share the same ID, Kafka ensures they all end up on the **same partition**, maintaining strict chronological order during processing.

#### 🔴 Scenario B: The Producer Missing Message IDs
These two reprices happened within one second of each other. Without an ID to tell Kafka that these events are connected, they could end up on **two different partitions** and be handled by differing consumers. 

This breaks Kafka's ordering guarantee, meaning the final state of the shirt could accidentally end up as either `$35.00` or `$13.00` depending on which consumer finished last.

---

## 📥 How Does an Application Consume Messages?

### 👥 Consumers and Consumer Groups
A **consumer group** is a group of consumers that will process every message on a given topic. You can have many consumer groups on the same topic, all processing its contents simultaneously. This makes Kafka a common choice in **Microservice architectures** because it supports message broadcasting.

### ⏳ Message Data Retention
Messages can have a short or long life on a Kafka topic and **continue to exist** even after they have been consumed. 
* When setting up a topic, you can configure the **Time to Live (TTL)** of the messages.
* Keeping the TTL high helps developers fix issues with consuming applications in the event of higher-than-expected loads or un-processable messages, as they can re-read past logs.

---

## 🤿 Consuming in Depth

While the "happy path" of consuming messages is relatively simple, there are slightly more complicated behaviors concerning Kafka consumer stability.

### 🔄 Polling and Rebalancing
Kafka consumers are like sharks—**if they stop moving forward, they die!** Even when there are no messages for a consumer to process, the consumer must continually poll Kafka. 

Polling Kafka serves two critical purposes:
1. It tells Kafka the consumer is **alive** and indicates what group it belongs to.
2. It retrieves the **next batch** of messages.

If the time between polls is too long, Kafka will believe the consumer is "dead" and reassign its partitions to another consumer. This concept is called **rebalancing**.

#### ⚠️ Common Causes of Unexpected Rebalancing:
* **Network issues:** Bad connectivity between Kafka and the consumer application.
* **Misconfiguration:** A poorly configured consumer with a max poll time limit lower than the actual execution requirements of the topic.
* **Slow processing:** Long synchronous tasks within the consuming thread stalling the next poll loop.
* **Acknowledge failures:** Processing errors causing a failure to acknowledge message delivery.

### 📦 Processing Batches of Messages
For simplicity, architectures are often visualized using one message at a time. However, to achieve more efficient and lower-latency integrations, Kafka consumers handle **batches of messages** during each poll while keeping batch processing fast.

Because of this, consumers must carefully manage how they parallelize a batch of messages. Core development concerns include:
* Ensuring the internal implementation processes messages **in strict order**.
* Placing an even higher emphasis on **handling each message quickly**.
* Tuning consumer configuration options to balance the **batch size** against the **time allowed between polls**.
