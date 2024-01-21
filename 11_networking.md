# Networking

## TCP vs. UDP

- **TCP: Transmission Control Protocol**

  - Reliable 2-way 1:1 communication
    - 3-way handshake:
      1. Send a SYN with a sequence number x
      2. Respond with an ACK with a sequence number x + 1 and a SYN with sequence number y
      3. Respond with ACK with a sequence number y + 1
    - Reliable delivery
      - Use sequence numbers to confirm
        - Similar to the handshake, send a message with a sequence number and expect an acknowledgment with the sequence number + 1
      - Using timeouts
        - Send a message with a sequence number, then wait for a set period before retrying the message
        - Use exponential backoff to prevent spamming the network
  - Flow and Congestion control
    - Flow control - look at the receiver buffer to limit the number of messages in flight. Reset the limit upon receiving an acknowledgment.
    - Congestion control - increase the window of how many messages can be sent at once. Use Additive Increase, Multiplicative Decrease (AIMD) to adjust the window size.
  - Used for most applications where reliability is crucial.

- **UDP: User Datagram Protocol**
  - Super bare bones, just send packets from node 1 to node 2
  - **Pros:**
    - Super fast
    - Checksums with every packet to avoid corruption
    - Multicast - send a single packet routed to multiple places
  - **Cons:**
    - Unreliable and super spammy
  - Used for low-latency scenarios where dropped data is acceptable.
    - Examples: Video calls, video games, stock prices

## Long Polling

- HTTP request that only completes when data is available
- Upon completion, create another request
  - Expensive to make new connections or send headers
  - Good if data is rarely needed
- Unidirectional

## Websockets

- Persistent, bidirectional connection between client and server
  - No overhead of headers or making new connections since there is a persistent connection
  - If largely inactive, too many connections waste resources
  - If a connection goes down, re-establishing the connection is required

## Server-Sent Events

- Persistent, unidirectional connection that reconnects automatically
  - Less headers / time spent establishing connections since it’s persistent
  - Wastes resources if there are many persistent connections
  - Re-establishing connections might not necessarily be good
    - **Thundering herd problem:** if the server goes down and comes back online, it will try to re-establish connections with all the clients at once, causing a huge strain and potentially knocking it out again
      - Use some random jitter similar to the leader candidate proposal process in the Raft distributed consensus algorithm
