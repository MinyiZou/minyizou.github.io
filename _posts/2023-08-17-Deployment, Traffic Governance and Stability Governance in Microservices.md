---
layout:     post
title:      Deployment, Traffic Governance and Stability Governance in Microservices
subtitle:   Microservices concept
date:       2023-08-17
author:     Minyi
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Microservices
    - Deployment
    - Distribute System
    - Golang
---

# Concept

Service deployment refers to the process of making a software service available for use in a production environment.

## Deployment

### Blue-Green Deployment

In this strategy, there are two independent, identical production environments, one called the "blue" environment and the other called the "green" environment.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb15d9c31-2712-43a0-a08d-34d357b35a76_740x914.png)

### Grayscale Deployment

Gray Deployment is a strategy where a new software version is gradually released to a small user subset before full rollout. It allows developers to test the version in a live setting without risking bugs or performance issues affecting all users. If problems arise, the version can be quickly rolled back, minimizing user impact. Once tested and refined, the version can be fully released.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3804c1b6-5676-43a9-b4a5-8c65419ad48b_2036x962.png)

## Traffic Governance

In a microservices architecture, we can precisely control end-to-end traffic routing based on dimensions such as region, cluster, instance, and request.

![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8753db2e-c653-420e-bcf5-4ce345370e21_1874x798.png)

## Stability Governance

1.  Rate Limiting (限流): This refers to the practice of controlling the number and frequency of requests a client can make to a server within a specified time period

    ![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5497c840-8839-4618-b278-cf6d30c67058_788x374.png)

2.  Degradation (降级): This is a strategy where non-essential features of a service are disabled or limited when the system is under heavy load or experiencing faults, in order to preserve core functionality.

    ![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F103612c0-d1a9-4497-b6ec-a85e5adb8b6e_784x376.png)

3.  Overload Protection (过载保护): This involves implementing measures to prevent a system from being overwhelmed by too much demand, which could lead to performance issues or system failure

    [![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0da9a4db-0cc7-45d6-8641-7192334d3dd2_792x376.png)

    .

4.  Circuit Breaker (熔断): This is a design pattern that detects failures and prevents the application from performing an action that is likely to fail, allowing the application to continue operating without waiting for the failed action to be fixed

    ![](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5a232d70-2a6f-4904-970e-a9413807efbe_792x376.png)

