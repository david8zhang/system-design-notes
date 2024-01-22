# Load Balancing

Load balancing is the method of distributing network traffic equally across a pool of resources that support an application. It's very important in a distributed system to load balance intelligently so we don't overwhelm any of our key components whenever we have large amounts of traffic coming through.

## Horizontal vs. Vertical Scaling

First, let's look at the different ways we can scale a system

**Vertical scaling** means improving the performance of our system by upgrading the hardware of our application server or database node.

**Horizontal scaling** means improving performance by distributing computational load across more machines, which are usually commodity hardware. This is typically where load balancing comes into play

## Routing Policies

There are several ways to route traffic in our load balancers:

**Weighted or Unweighted Round Robin:**

A round robin policy is when we distribute requests to each server sequentially and wrap around. (you can imagine it's like dealing cards in Poker to each player).

An _unweighted_ round robin strategy just means we naively distribute traffic equally across all our nodes. In constrast, a _weighted_ strategy sends more requests to a given node if it has a greater weight.

**Lowest Response Time:**

In a lowest response time routing strategy, we keep a running average of the response times of each node. We then route requests to the ones with the lowest response time.

**Hashing:**

Hashing is where we take a hash of request content or headers and distribute load to each node corresponding to the hash value. We can do this at the L4 or L7 layer. (Recall in the OSI model of networking, L4 corresponds to the Transport Layer, like TCP, and L7 corresponds to the application layer, like HTTP)

- **L4:** Hash information based on information exposed at the networking layer, like IP, protocol, etc. (faster).
- **L7:** Hash information available in the actual message content (slower but more flexible).

It's important that we use consistent hashing so that every request goes to the same partition.

## Fault Tolerance

Let's look at 2 ways we can ensure fault tolerance in our load balancer setups.

### Active-Active

In an active-active setup, we have more than 1 active load balancer. That means requests come into both load balancers, and both load balancers actively redirect traffic to the appropriate downstream application servers.

**Pros**

- Higher throughput since requests can be split across the two load balancers.
- Fault tolerance since we essentially have replicas of our load balancers.

**Cons**

- Might make local state maintained on the load balancer (like keeping track of average response time per server) a bit more complicated.
- Complexity in configuration.
- Potentially higher costs since we need to run multiple load balancers at full capacity.

### Active-Passive

In an active-passive setup, we have an active load balancer that actually performs the request routing and one passive one that sits idly on standby. We utilize a coordinator service like Zookeeper, which will continuously send heartbeats to the active load balancer and swap over to the passive one in the event of a failure.

**Pros:**

- Simplicity - we just have one load balancer to manage.
- Cost-effective - only one load balancer is run at full capacity.

**Cons:**

- Lower throughput.
- Underutilization of resources in passive nodes.

## Additional Reading / Material

**jordanhasnolife System Design 2.0 Playlist** ["Load Balancing - The Right Way to Do It"](https://www.youtube.com/watch?v=PERKHUJYotM&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=57)
