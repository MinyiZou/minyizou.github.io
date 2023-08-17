---
layout:     post
title:      Microservices Architecture Principles
subtitle:   History of Architecture
date:       2023-08-16
author:     Minyi
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Microservices
    - Distribute System
    - Golang
---

# History

## Monolithic architecture

Monolithic architecture refers to a software design pattern where all components and services are combined into a single codebase and run as a single application.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fff70cf2a-f8ca-4aa4-b0f5-4a1a18cc7cfe_1404x720.png)

> refer to: https://medium.com/design-microservices-architecture-with-patterns/when-to-use-monolithic-architecture-57c0653e245e

## Vertical Application Architecture

Vertical Application Architecture refers to a design approach where different functionalities or components of a system are separated based on specific business or domain capabilities, rather than technical layers.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Faa21e698-e649-406c-8f6e-d427d788a8b6_1048x848.png)

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F75beb902-3326-4a2f-8dcb-e0dfdf5500a9_1600x786.png)

## Distributed Architecture

Distributed Architecture refers to a system where components and services are run on different machines or servers, typically interconnected by a network, to achieve a common goal. Within such an architecture, it's common practice to extract independent services that are unrelated to the core business logic.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F528487f5-3040-443d-943f-2fd5edd60e10_1476x870.png)

## Service oriented architecture (SOA)

Service-Oriented Architecture (SOA) is a design pattern in which software applications are organized around services rather than monolithic components or modules. This architectural style encourages the decomposition of a business process or function into a set of interconnected services that can be reused to build a flexible and scalable system.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9f4a7cb4-1176-492f-94ee-f62e04076e07_1302x888.png)

## Microservices architecture

Microservices architecture is a software development approach that breaks down a large application into multiple small, independent services. Each service can run and be deployed independently and typically has its own database and API.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F23b05ca0-888f-4a51-bb85-4279800a0fb2_1808x682.png)

1.  Service Governance: This refers to the set of practices, policies, and standards that help manage and control the microservices in a system. Service governance includes aspects such as service discovery (finding available services), service versioning (managing different versions of a service), and load balancing (distributing requests across multiple instances of a service). It helps ensure that services can effectively communicate with each other and that the overall system operates smoothly.
2.  Observability: Observability in microservices architecture involves monitoring and logging the behavior of services to understand their performance and health. This is essential for detecting and diagnosing issues, as well as optimizing the system. Observability tools provide insights into metrics such as response times, error rates, and resource usage. They also allow for tracing the flow of requests through services to identify bottlenecks or failures.
3.  Security: Security is a critical element in microservices architecture, as each service can be a potential attack vector. Security practices in microservices include securing communication between services (e.g., using HTTPS), implementing authentication and authorization (verifying the identity of users and services and controlling their access), and protecting data (e.g., encrypting sensitive data). Ensuring security helps prevent unauthorized access and data breaches, safeguarding the integrity and confidentiality of the system.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F37519412-7090-4154-9721-d29e8241dfd8_912x538.png)

If we think of HDFS as Microservice, it would be like this:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0ace12bc-9eee-4d60-a852-ce41c7c3021d_996x640.png)

The communications between different services in Microservice:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9e62432d-ba2a-4d49-82eb-b40b34fb3411_1138x692.png)

#### Registration

When service A needs to communicate with service B, using a hardcoded IP address and port number is not an ideal approach (as service B could have multiple IP addresses). Instead, you might want to consider using a NameServer. However, there could be issues if the instance in Service B becomes unavailable.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F36daa1be-ce74-4b55-8139-dc4fc6b66ee4_940x714.png)

Therefore, you can add a service registry to solve the problem:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb705d86e-6507-477e-aec1-0e61a1f0573d_978x874.png)

Meanwhile, it is much easy to control Service online and service offline:

Procedure of service offline:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F87d79006-f311-4fe6-9000-9572b22b633b_1130x884.png)

Procedure of service online:

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc9420802-3bc7-4cf2-ba1c-a4c13261fb0f_1220x882.png)

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F827bacbd-f75e-40d3-bffd-fede6288338c_1222x838.png)

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2307245e-a936-4ebb-833e-a45b62650015_1276x882.png)

  
Microservices Traffic Patterns:

**Internet uses HTTP, intranet uses RPC.** Why?

RPC can be more efficient than HTTP, as it does not require the overhead of HTTP headers and can use binary data formats that are more compact and faster to parse than textual data formats such as JSON or XML.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9764fc2e-0a68-4703-b593-11d4facdfc2b_1156x654.png)
