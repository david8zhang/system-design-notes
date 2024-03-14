# Horizontal vs. Vertical Scaling

First, let's look at the different ways we can scale a system and examine their pros and cons.

**Vertical scaling** means improving the performance of our system by upgrading the hardware of our application server or database node. This is referred to as _scaling up_.

**Pros**

- **Lower costs** since upgrading a pre-existing server is cheaper than purchasing a new one
- **Less complex process communication** as there's just a single node handling all traffic, removing the need to synchronize and communicate with other machines
- **Less complicated maintenance** since you just need to manage and maintain a single node

**Cons**

- **Single point of failure** - one node = a single point of failure. If it goes down, your whole application goes down. This can be mitigated with a backup machine.
- **Upgrade limitiation**, since there's only so much you can upgrade

**Horizontal scaling** means improving performance by distributing computational load across more machines, which are usually commodity hardware. This is sometimes referred to as _scaling out_, and is typically where load balancing comes into play.

**Pros**

- **Scaling is easier from a hardware perspective**, since you don't need to analyze your specific system and determine which parts to upgrade. You simply add more machines and distribute the load
- **Less downtime** - you don't need to shut off the machine when scaling since you're adding more machines rather than upgrading a single one
- **Increased resilience and fault tolerance** due to distributing load across more machines. If one goes down, the others can continue handling traffic
- **Increased performance** due to more endpoints for connections, enabling greater traffic throughput.

**Cons**

- **Increased complexity of maintenance and operation** since you have more servers to maintain, and you'll need to figure out how to load balance traffic between them
- **Increased initial costs** since adding new servers is far more expensive than upgrading old ones
