# Routing Policies

There are several ways to route traffic in our load balancers. These ways can be broadly divided into _static_ and _dynamic_ load balancing strategies.

## Static Load Balancing

_Static_ load balancing distributes traffic without adjusting for the current state of the system or servers.

**Weighted or Unweighted Round Robin:**

A round robin policy is when we distribute requests to each server sequentially and wrap around. (you can imagine it's like dealing cards in Poker to each player).

An _unweighted_ round robin strategy naively distributes traffic equally across all our nodes. In constrast, a _weighted_ strategy sends more requests to a given node if it has a greater weight.

In practice, round robin is implemented at the [DNS](/concepts/11_networking?subtopic=08_dns) level. An authoritative nameserver will have a list of different A records for a domain, providing a different one in response to each DNS query.

![DNS-based-load-balancing](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/DNS-based-load-balancing.png?alt=media&token=bf518146-c0ba-42fe-8af8-3cdacc610131)

For a weighted round robin strategy, server administrators can configure weights within DNS records.

**Hashing:**

Hashing is where we take a hash of request content or headers and distribute load to each node corresponding to the hash value. We can do this at the L4 or L7 layer. (Recall in the OSI model of networking, L4 corresponds to the Transport Layer, like TCP, and L7 corresponds to the application layer, like HTTP).

- **L4:** Hash information based on information exposed at the networking layer, like IP, protocol, etc. (faster).
- **L7:** Hash information available in the actual message content, lke URL, type of data or cookie information. (slower but more flexible).

Due to advances in CPU and memory speed and cost, L4 load balancing performance advantages are no longer as relevant today as they were before, when commodity hardware was not as powerful.

## Dynamic Load Balancing

_Dynamic_ load balancing takes into account the current capacity of servers in the system when determining how to distribute load, which may change over time.

**Lowest Response Time**

In a lowest response time routing strategy, we keep a running average of the response times of each node. We then route requests to the ones with the lowest response time.

**Least Connection**

In a least connection routing strategy, we check which servers have the fewest connections open and send traffic to those servers. This assumes all connections require roughly equal processing power.

**Resource-Based**

A resource-based load balancing strategy distributes load based on what resources each server has available. Specialized software (called an "agent") runs on each server and continuously monitors that server's available CPU and memory. The load balancer queries a given server's agent before distributing traffic to it.
