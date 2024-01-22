# Fault Tolerance

Let's look at 2 ways we can ensure fault tolerance in our load balancer setups.

## Active-Active

In an active-active setup, we have more than 1 active load balancer. That means requests come into both load balancers, and both load balancers actively redirect traffic to the appropriate downstream application servers.

**Pros**

- Higher throughput since requests can be split across the two load balancers.
- Fault tolerance since we essentially have replicas of our load balancers.

**Cons**

- Might make local state maintained on the load balancer (like keeping track of average response time per server) a bit more complicated.
- Complexity in configuration.
- Potentially higher costs since we need to run multiple load balancers at full capacity.

## Active-Passive

In an active-passive setup, we have an active load balancer that actually performs the request routing and one passive one that sits idly on standby. We utilize a coordinator service like Zookeeper, which will continuously send heartbeats to the active load balancer and swap over to the passive one in the event of a failure.

**Pros:**

- Simplicity - we just have one load balancer to manage.
- Cost-effective - only one load balancer is run at full capacity.

**Cons:**

- Lower throughput.
- Underutilization of resources in passive nodes.
