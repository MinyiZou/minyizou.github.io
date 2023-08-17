---
layout:     post
title:      Architecture of Kafka
subtitle:   Message Queue-Kafka
date:       2023-08-13
author:     Minyi
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Kafka
    - Message Queue
    - Distribute System
    - Golang
---

# Introduction 

## Concept

Apache Kafka is a distributed event streaming platform that is widely used for building real-time data pipelines and streaming applications.
![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1cdceadc-df79-47ca-8ba9-b6bb3dadf8d3_1994x960.png)

1.  **Producer**:

    -   **Role**: Sends or publishes data to Kafka topics.
    -   **Functionality**: Producers push data to topics. The data being sent by a producer is typically called a "message", and each message has a key and a value.
    -   **Responsibility**: Producers decide which topic partition the data will be written to, typically based on the message key.
    -   **Fault Tolerance**: Can be configured for various levels of acknowledgement to ensure data durability.

1.  **Leader**:

    -   **Role**: The primary replica for a partition and serves client requests for that partition.
    -   **Functionality**: Every partition has one leader. When messages are written to a topic, they are written to the partition's leader.
    -   **Responsibility**: Leaders handle all reads and writes for the respective partition. If a leader fails, a new leader is elected from the existing followers for that partition.
    -   **Fault Tolerance**: Plays a crucial role in Kafka's ability to tolerate failures. If the leader of a partition fails, one of its followers will automatically become the new leader.

1.  **Follower**:

    -   **Role**: Replicas of the leader that passively replicate the leader's data.
    -   **Functionality**: Followers consume messages from the leader just like a typical Kafka consumer. However, they don’t serve client requests.
    -   **Responsibility**: Their primary duty is to replicate the data and to step in as the leader if the current leader fails.
    -   **Fault Tolerance**: Helps in providing data redundancy. If a broker (or multiple brokers) containing leaders fail, the followers can ensure data isn't lost and can step in as leaders.

1.  **Consumer**:

    -   **Role**: Reads or subscribes to data from Kafka topics.
    -   **Functionality**: Consumers read data from topics, starting from a specific offset (i.e., position) and can read data from multiple topics.
    -   **Responsibility**: Keeps track of what has been read by storing the offset of messages. If a consumer dies or if a new consumer is added, it can start reading from where the last consumer left off.
    -   **Consumer Groups**: Multiple consumers can work together as part of a consumer group. When they do so, they cooperate to consume data from one or more topics, with each consumer in the group reading from a unique set of partitions.

1.  **Broker**:

    -   **Role**: A Kafka server that stores data and serves client requests.
    -   **Functionality**: Kafka messages are organized into topics, which are split into partitions. Each broker manages a subset of these partitions. A Kafka cluster is composed of multiple brokers.
    -   **Scalability**: Adding more brokers allows Kafka to handle more total data and requests, offering a way to scale out the system.
    -   **Fault Tolerance**: Brokers support data replication. Each topic can be configured to have its data replicated across multiple brokers, ensuring data is still available even if some brokers fail.
![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F95522c11-3973-4446-909b-29287afe5122_2220x1196.png)

#### Hypothetical scenario

Sending one message at a time and waiting for an acknowledgement before sending the next, the implications would be **Latency Increase**.

Therefore, you can use a batch and send several messages once a time:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbea6163a-1596-4fff-936c-21daed304533_310x512.png)

However, f an individual message is too large and multiple such messages are sent concurrently, it could saturate the available bandwidth.

**Message Compression**: Kafka supports message compression out of the box. Producers can compress messages before sending, and consumers can decompress these messages upon receiving. Kafka supports several compression codecs including GZIP, Snappy, LZ4, and Zstd. By using compression, you can significantly reduce the size of the messages and hence the required bandwidth.
![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F54ac9194-9d94-44e2-8594-be315e5df008_2190x996.png)

#### Broker

**Broker file structure:**

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F02741429-d8e3-40fd-aec7-bfc42f4f4c2e_1744x904.png

**How to get the message from a broker?**

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdfaff41b-bd73-4aee-8778-0292876bc64b_2380x1174.png)

In Kafka, topics are divided into partitions, and each message within a partition has an offset which serves as a unique identifier. These messages are also usually sequenced in order of their arrival time. Given this ordered nature, if you want to find a message based on a specific timestamp, you can utilize a binary search approach for efficiency.

1.  Binary Search for Timestamp
1.  Retrieving the Largest Offset Less Than Target Timestamp
1.  Fetching Data

##### Broker Optimization - Zero Copy

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F614395b2-645d-4b31-b62f-ab749ef16832_2000x1252.png)

"Zero copy" is a method that, as the name suggests, enables data to be transferred between buffers without being copied multiple times between application address space and the kernel address space. When applied to systems like Kafka's brokers, zero copy can significantly improve efficiency and performance, especially when dealing with large volumes of data.

1.  **Traditional Data Transfer**:

    -   In a conventional data transfer mechanism, data sent from a source to a destination usually undergoes several copies: first from the application to the OS kernel, then to the system's I/O buffers, and finally to the destination application. Each of these copy operations consumes CPU, memory, and I/O bandwidth.

1.  **Zero Copy in Kafka**:

    -   Kafka brokers are designed to handle a lot of read and write operations, especially when serving many producers and consumers. One key performance optimization Kafka uses to achieve high throughput is leveraging the zero copy technology.
    -   When a Kafka broker sends a message to a consumer or replicates data to another broker, it can use the `sendfile` system call (available in many modern operating systems) to transfer data directly from the file system cache to the network socket, bypassing the need to copy data to the application layer entirely.
    -   This means that when reading data from a Kafka topic partition log (which is essentially a file on disk), the data doesn't need to be copied into the broker's JVM memory before being sent out. Instead, it goes directly from disk cache to the network, saving both CPU and memory resources.

#### Consumer

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbd6e0b97-a9cc-434c-9dc5-05a4143b399a_1700x1000.png)

**How does consumer1 in group 1 select a partition in Topic?**

1.  Manual configuration when starting kafka service. But if this partition crashed accidentally, it can not complete disaster discovery automatically.
1.  Use coordinator to configure consumers and partitions automatically.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa134146e-fe78-40d3-b64f-7995536c98fb_2250x1042.png)

The steps of consumer rebalance:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbd0d8f84-044b-450e-a294-8c426081668e_2078x1116.png)

#### Restart kafka

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1b85afde-f4c9-492c-be55-2b9f8c80ded5_1888x1304.png)

#### Load imbalance

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe9a1bb10-eb03-4b10-8195-f31f2bad95b8_1878x842.png)

For instance, consider a scenario where broker 1 is overloaded with more partitions and data, necessitating the transfer of partition 3 to broker 2. This transfer can introduce IO challenges, requiring intricate solutions to address the issues :

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F286ac777-1679-4aff-b2db-584c2f5af07e_2024x900.png)
