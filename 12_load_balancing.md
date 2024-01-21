# Load Balancing

## Horizontal vs. Vertical Scaling

- **Vertical scaling** means improving the performance of our system by upgrading the hardware of our application server or database node.
- **Horizontal scaling** means improving performance by distributing computational load across more machines, which are usually commodity hardware.
  - This means we need something to route requests to each node to make sure that none get overwhelmed.

## Routing Policies

- **Weighted or Unweighted Round Robin:**

  - Distribute requests to each server sequentially and wrap around.
  - For weighted, follow the same methodology except we send more requests to a given node if it has a greater weight.

- **Lowest Response Time:**

  - Keep a running average of the response times of each node and route requests to the ones with the lowest response time.

- **Hashing:**
  - **L4:** Hash information based on information exposed at the networking layer, like IP, protocol, etc. (faster).
  - **L7:** Hash information available in the actual message content (slower but more flexible).
  - Need to use consistent hashing so that every request goes to the same partition.

## Fault Tolerance

### Active-Active

- Have more than 1 active load balancer.
- **Pros:**
  - Higher throughput since requests can be split across the two load balancers.
  - Fault tolerance since we essentially have replicas of our load balancers.
- **Cons:**
  - Might make local state maintained on the load balancer (like keeping track of average response time per server) a bit more complicated.
  - Complexity in configuration.
  - Potentially higher costs since we need to run multiple load balancers at full capacity.

### Active-Passive

- One active load balancer and one passive one.
- Use a coordinator service like Zookeeper, which will continuously send heartbeats to the active load balancer and swap over to the passive one if it detects that it’s down.
- **Pros:**
  - Simplicity - we just have one load balancer to manage.
  - Cost-effective - only one load balancer is run at full capacity.
- **Cons:**
  - Lower throughput.
  - Underutilization of resources in passive nodes.
