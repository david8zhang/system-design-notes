# Monoliths and Microservices

## Monolithic Architecture

A **Monolithic architecture** is a singular system with one code base that couples all business concerns together. These are convenient early on in a project's life since they're easy to understand and manage. Testing and debugging is streamlined since we just have a single codebase or service, and we don't need to coordinate business logic across multiple endpoints which might give us some nice performance benefits.

However as a system grows, some major problems begin to arise: For one thing, we have a single point of failure so our system is less reliable. We also can't scale up individual parts of our system independently, or use different technologies for each part since it's one giant codebase. Finally, making changes requires updating and deploying the entire system, which can be slow and inefficient.

## Microservices

A **Microservices architecture** relies on coordinating a series of independently managed and deployed services. Each "microservice" is responsible for doing one thing and thus all development and testing is encapsulated. They solve for many of the problems we saw before with monolithic architectures in that they allow for faster, smaller scale updates to our system and the flexibility to choose different technologies which might be more suited for specific domains. They also give us higher reliability - we can isolate failures within our system now that logic is split across many different services.

There are some downsides as well - with more services comes more management complexity and costs. Communication and agreement on interfaces across service owner teams can bog down development speed. Allowing a variety of technologies can degrade standardization and present challenges in logging and monitoring. Furthermore, with each new service comes its own hosting infrastructure and tooling, which could mean exponentially increasing costs as the system scales.

Microservices architectures are widely adopted at large companies operating at high scale, so the next few sections will explore some methods of developing and maintaining them.
