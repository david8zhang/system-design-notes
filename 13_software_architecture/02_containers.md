# Containers

A **container** is a standard unit of software that packages up code and all its dependencies for reliable, efficient execution and deployment. In the context of microservices, each service might be using different technologies and runtimes. Containers allow us to bundle these up in a nice, scalable way.

A widely used platform for building and maintaning software containers is [Docker](https://www.docker.com/)

## Virtual Machines v.s Containers

**Virtual Machines** are instances of operating systems co-located on the same physical machine. These are coordinated through the use of a hypervisor, which abstracts the physical host hardware from the operating system environment. Each Virtual Machine has its own OS, memory and other resources, which are isolated from other VMs.

**Containers**, on the other hand, are lightweight, portable, executable images that only concern themselves with application software and their dependencies. Unlike VMs, they share the same operating system kernel so they use fewer resources and can start and stop faster. Containers also allow us to dynamically scale resources, unlike VMs, where we need to specify our resource requirements in advance.

Virtual Machines and containers aren't mutually exclusive. A container could run within a VM to reap the benefits of both worlds, namely the portability and speed afforded by containers and the security afforded by isolated VM environments.

## Kubernetes & Container Management Systems

A few problems arise when we start using a whole bunch of containers together in our microservices architecture. How do we know which host to put each container on? And what should we do in the event of a host failure?

Kubernetes and other container management systems provide a "control plane" to deal with these kinds of issues. In Kubernetes specifically, we have a control plane node that runs some kind of coordination service like Zookeeper or Etcd. It talks to all of our hosts through "Kubelets", which are agents that run on each host. These Kubelets coordinate and provision "pods", which correspond to individual Docker containers. If a container fails, the Kubelet can restart it, and if a host goes down, the control plane can replicate it over to another host.
