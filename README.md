# Introduction to Apache Kafka's Core Concepts

Kafka is a deceivingly simple piece of messaging infrastructure. In this article, I will explain how Kafka works aimed at new users of Kafka. Some understanding of messaging brokers will help most certainly help.

---

## ❓ Firstly, Why Even Use Kafka?

Kafka is a **persistent messaging broker** that is **highly customizable**. Some of the reasons why people choose Kafka is because they need:
* **Guaranteed delivery** of messages in order.
* **High availability** across regions.
* **Support** for distributing workloads.
* **Fast throughput** when handling large amounts of data.
* **Fault-tolerant** consumption of messages.

The nice thing about the Kafka infrastructure is that it has lots of configuration options. Meaning you can tune Kafka to your specific workloads.

---

## ⚙️ What Does Kafka Do?

There are two types of applications that use Kafka:
1. Applications that want to **consume** some information.
2. Applications that would like to **distribute** some information.

### 📤 Producing an Event to Kafka
Kafka uses **Topics** as its messaging concept to group related messages.

When a producer publishes something to Kafka, it provides a few pieces of information. This includes the:
* **Topic Name**
* **ID of the Message**
* **The Message itself**

If all three pieces of information are provided, then Kafka will guarantee that all messages with the same ID end up on the **same partition** of the topic.

---

## 🧩 Wait, What Are Partitions?

A partition is Kafka’s way of **parallelizing topics** so that we can distribute the processing of messages between consumers to handle various amounts of load. 

### 📐 Configuration & Sizing
* **Partition count is rigid:** The number of partitions is configurable for the topic. However, it is not easy to split partitions after they are created without causing potential inconsistencies in the data. 
* **Rule of thumb:** It’s best to create more partitions than you think you need. Think about how many consumers for a single application you expect to run and then **multiply that number by 2**.

### 🔍 Diving a Bit Deeper With an Example
Imagine an online shop that sells different Products. That product might go through a number of reprices in its lifecycle. 

Let’s say we have a system where other applications need to be made aware of when a product has changed in price, often Kafka is a good choice for this. If we have a producer that produces the following messages in order:
1. A **Product Created** event for *“White shirt”* | price `$15.00` at `07:00`
2. A **Product Reprice** event for *“White shirt”* | price `$35.00` at `08:00:00`
3. A **Product Reprice** event for *“White shirt”* | price `$13.00` at `08:00:01`

#### 🟢 Scenario A: The Producer Provides Message IDs
If the producer provides the ID’s for these messages then this works great. They will all end up on the exact same partition in chronological order.

#### 🔴 Scenario B: The Message ID is Missing
These two reprices happened within one second of each other. Without an ID to tell Kafka that these events are connected, they could end up on **two different partitions** and be handled by differing consumers. 

This results in our use of Kafka not guaranteeing the processing order of these events, meaning the final recorded price of the shirt could accidentally end up as either `$35.00` or `$13.00`.

---

## 📥 How Does an Application Consume Messages?

### 👥 Consumers and Their Groups
A **consumer group** is a group of consumers that will process every message on a given topic. You can have many consumer groups on the same topic, all processing its contents. This makes Kafka a common choice in **Microservice architectures** as it supports message broadcasting.

### ⏳ Message Lifespan & Data Retention
* **Messages persist after consumption:** Messages can have a short or long life on a Kafka topic and continue to exist even *after* they have been consumed. 
* **Time to Live (TTL):** When setting up a topic, it is possible to set the time to live (TTL) of the messages on a topic. 
* **Debugging advantage:** It is not uncommon for this TTL to be quite high. This helps developers to fix any issues they have with their consuming applications in the event of higher than expected load or un-processable messages, as they can re-read the data.

---

## 🤿 Consuming in Depth

We have covered the happy path of consuming messages which is relatively simple. However, there are some slightly more complicated topics concerning consuming from Kafka Topics.

### 🔄 Polling and Rebalancing
Kafka consumers are like sharks—**if they stop moving forward they die!** Even when there are no messages for a consumer to process, the consumer has to continually poll Kafka. 

Polling Kafka serves two critical purposes:
1. To tell Kafka the consumer is **alive** and what group it’s in.
2. To retrieve **more messages**.

If the time between polling is too long, then Kafka will believe the consumer is “dead” and reassign its partitions to another Consumer. This concept is called **rebalancing**.

#### ⚠️ Common Causes of Unexpected Rebalancing:
* **Bad network connectivity** between Kafka & the consumer.
* **A poorly configured consumer** with a higher poll time than expected by the topic.
* **Slow processing of messages** due to long synchronous tasks within the consuming thread.
* **Processing errors** that cause a failure to acknowledge the processing of messages.

### 📦 Processing Batches of Messages
For simplicity, everything above has been shown using one message at a time. But to have more efficient and lower latency integrations with Kafka, we would like to handle **batches of messages** for each poll while keeping the processing of the batch fast.

Our consumers then need to care even more about how they parallelize a batch of messages. These concerns include:
* How their implementation ensures that they **process messages in order**.
* Putting an even **higher emphasis on handling the message quickly**.

To manage this, the consumer has many options it can set to manage how it consumes messages. Some important ones include the **batch size** and the **time between polls**.

---

## 👋 Outro
Thanks for reading and I hope you now have a better understanding of what Kafka is doing under the hood. Please let me know if this helped with your understanding of Kafka or if you notice anything that you would like more information about.
