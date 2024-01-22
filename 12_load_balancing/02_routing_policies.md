# Routing Policies

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
