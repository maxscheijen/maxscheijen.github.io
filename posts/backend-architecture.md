---
title: "Backend Architecture: Monolith, Microservices, and Serverless"
subtitle: "What are common backend architecture"
description: "The pro's and con's of common backend architectures"
date: 2023-05-17
author: "Max Scheijen"
categories:
  - backend
  - architecture
draft: true
---

In recent years, **microservices** and **serverless** architectures have gained popularity due to their scalability and flexibility. Microservices have become a popular choice for building large-scale applications with complex requirements, such as machine learning platforms, as they allow for modular development and easier maintenance.

Serverless architectures have also gained traction in the machine learning community due to their cost-efficiency and scalability. By only paying for the resources used, serverless architectures can be more cost-effective than traditional server-based architectures. Additionally, serverless architectures can automatically scale up or down in response to traffic spikes, making them an attractive option for machine learning applications with unpredictable workloads.

That being said, **monolithic** architectures can still be a suitable choice for small to medium-sized machine learning applications that don't require a high degree of scalability and can benefit from the simplicity and reduced operational overhead of a monolithic architecture. Ultimately, the choice of architecture should be based on a careful evaluation of the application's requirements and constraints.

I will discuss the pros and cons of each backend architecture and their impact on development experience, scalability, response time, reliability, and cost.

## Monolith

The Monolith architecture is the **simplest** of the three. This architecture is where the entire application is built as a single, self-contained unit. Meaning that all of the application's components, including the machine learning models, APIs, and databases, are housed within a single codebase.

This architecture is **easy to run** and test locally, making it the fastest way to build, test, and deploy.

![](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fcodesteps.com%2Fwp-content%2Fuploads%2F2020%2F09%2Fmonolithic-architecture.png&f=1&nofb=1&ipt=831194c821229d3d32c9d4d02c6d03e664aa6f7e962cbdeb4e1574be41c78ad2&ipo=images){width=50% fig-align="center"}

However, Monolith is not suitable for large-scale applications due to its **limited scalability**. It can scale vertically by adding memory and CPUs, but horizontal scaling is not feasible due to **memory overhead and caching**.

Additionally, load balancers add **infrastructure complexity**. Monolith architecture also has a **single point of failure**, and if the process crashes, the entire application goes down.

Renting a virtual server, such as [AWS EC2](https://aws.amazon.com/ec2/),  [Azure Virtual Machines](https://azure.microsoft.com/en-us/products/virtual-machines/), or [Google Compute Engine](https://cloud.google.com/compute/), is a common way to deploy Monolith architecture. It has **fixed pricing**, however the processing power of these virtual services can be **unutilized**.

## Microservices

In a microservices architecture, the application is broken down into smaller, independent services, each with its own API and database. Each service can be developed, tested, deployed, and scaled independently, making it easier to maintain and scale the application over time. Services communicate through internal network calls.

Microservices architecture offers **better scalability** than Monolith architecture, as services can scale vertically and horizontally. Specific services can be optimized, and **failures are isolated** per service, meaning the app remains partially functional.

![](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fmiro.medium.com%2Fmax%2F1400%2F1*LkHA1vkfdcG6Zko2mj-PAQ.png&f=1&nofb=1&ipt=08e7e9e2158efa14b2a7db50de17aa0114af61f02d2020b02c86be554f49eeba&ipo=images){width=50% fig-align="center"}

However, Microservices architecture requires jumping around the codebase, and **integration testing** can be difficult. Additionally, service communication via internal network calls can have a **performance penalty** because a (de)serialization, especially when dealing with bigger payloads.

Microservices architecture has **additional costs** such as deploying a Managed Kubernetes Cluster, container registry fees, and load balancers.

## Serverless

A serverless architecture is where the application's code is executed on-demand, in response to specific events or triggers, rather than being hosted on a dedicated server. In this architecture, the application is broken down into smaller functions, which are executed in a serverless environment.

Serverless architecture offers **simple code without any infrastructure** or architecture to maintain. It allows you to configure memory and CPU per function, **scaling horizontally** from zero to "infinite."

Serverless architecture has **runtime limitations**, such as native dependencies, image/PDF conversion, and websockets/SSE (server-side events) that require paying for long-running sessions.

![](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fmiro.medium.com%2Fmax%2F1280%2F1*5C4EkjRtMlKOObYfHX4vCA.png&f=1&nofb=1&ipt=19282576a8e8b65ab8f602657220400f02686c7f07da731ae808815a58ae7712&ipo=images){width=50% fig-align="center"}

**Cold starts** can also cause a delay in response time, ranging from 100ms to over 1000ms. However, failures are **isolated per function**, and there is no infrastructure to maintain which reduces the possibilities for creating bugs.

::: {.column-margin}
> AWS Lambda $0.20 per 1M requests &  400,000 GB-seconds per month free, up to 3.2 million seconds of compute time. $0.00001667 for every GB-second used thereafter.
:::

Deploying functions to a provider, such as [AWS Lambda](https://aws.amazon.com/lambda/), [Azure Functions](https://azure.microsoft.com/en-us/products/functions/), or [GCP Cloud Functions](https://cloud.google.com/functions/), is a common way to implement Serverless architecture and overall it is probably the most cost efficient option for most use cases.


## Conclusion

In conclusion, each architecture has its pros and cons. Monolith architecture is simple and easy to test but lacks scalability. Microservices architecture offers better scalability but requires more infrastructure and can be costly. Serverless architecture is simple, scalable, and cost-effective, but with runtime limitations and occasional cold starts. The choice of architecture depends on the requirements and constraints of the application.