# Networking

## OSI Model

The OSI model is a logical and conceptual framework that divides network communications functions into seven layers. If you've ever heard the terms "L7" or "L4" with regards to networking, this is what that's referring to.

It encapsulates every type of network communication across both software and hardware components and ensures that two standalone systems can communicate via standardizd interfaces or protocols based on the current layer of operation.

The seven layers are as following:

1. **Physical** - the _physical_ cables and devices that deal with data transfer
2. **Data Link** - the technologies used to connect two machines across a network where the physical layer exists (Ethernet, MAC)
3. **Network** - concerned with routing, forwarding, and addressing across a dispersed network or multiple connected networks (IPv4, IPv6)
4. **Transport** - focuses on ensuring data packets arrive in the right order without losses or errors and can recover seamlessly (TCP, UDP)
5. **Session** - responsible for network coordination between two separate applications in a system (Network File System, or NFS, and Server Message Block, or SMB)
6. **Presentation** - Concerned with the syntax of the data itself for applications to send and consume. (HTML, JSON)
7. **Application** - Concerned with the specific type of application itself and its standardized communication methods (HTTPS, SMTP)

## TCP: Transmission Control Protocol

TCP, or Transmission Control Protocol, is a network protocol which provides reliable 2-way 1:1 communication through the use of handshakes and acknowledgements. TCP is used for most applications where reliability is crucial.

The process for establishing a connection is via a 3-way handshake:

1. Send a SYN with a sequence number x
2. Respond with an ACK with a sequence number x + 1 and a SYN with sequence number y
3. Respond with ACK with a sequence number y + 1

TCP tries to guarantee reliable delivery using sequence numbers to confirm message receipt. This operates similarly to the handshake, send a message with a sequence number and expect an acknowledgment with the sequence number + 1

TCP also uses timeouts to retry whenever it doesn't receive an acknowledgement. For example, it sends a message with a sequence number, then wait for a set period before retrying the message. It uses a strategy called _exponential backoff_ to prevent spamming the network. Exponential backoff

### Flow and Congestion control

**Flow control** deals with ensuring that the sender isn't sending more messages than a receiver can consume. TCP does this by looking at the receiver buffer to limit the number of messages in flight. It then resets the limit upon receiving an acknowledgment.

**Congestion control** deals with ensuring that message traffic does not degrade network response times. TCP handles this by setting a window of how many messages can be sent at once. It then uses _additive increase, multiplicative decrease_ (AIMD) to adjust the window size. AIMD dictates that the window should grow in size linearly when there's no congestion, but reduce in size exponentially when there is.

## UDP: User Datagram Protocol

UDP, or User Datagram Protocol, is a simple, connectionless communication model without any of the reliability mechanisms that TCP provides. It's fairly bare bones - you just send packets from one node to another with some checksums for data integrity.

For that reason, it's super fast, and is used for low-latency scenarios where dropped data is acceptable. (e.g. video calls, video games, real-time tracking of stock prices). It also supports **multicast**, which is when a single packet is routed to multiple places.

### Checksums

**Checksums** are sent with every UDP header to ensure that packets aren't corrupted. Before sending off the segment, the sender:

1. Computes the checksum based on the data in the segment
2. Stores the computed checksum in the final two bytes of the UDP header

Upon receiving the data, the recipient

1. Computes the checksum based on the received segment
2. Compares it to the checksum in the header it received. If they aren't equal, it knows data was corrupted.

## Long Polling

In traditional HTTP communication, the client sends requests to the server and waits for a response. Once received, the client needs to make a brand new requests to receive new data. This is "short polling", which might not be efficient for real time scenarios since we'd need to keep sending requests to the server.

Long Polling, on the other hand, keeps the requests open until new data is available. Once it receives that data, it completes the request and creates a new one. Since creating new connections is expensive, long polling is best used for situations where data is rarely needed. In addition, it's **unidirectional**, since data is only sent from the server to the client.

## Websockets

**WebSocket** is a communications protocol which provides persistent two-way communication between client and server. It's implemented over a single TCP connection.

Since it's _persistent_, we avoid the overhead of creating new connections. However, if these connections are largely inactive, we might waste resources. In addition, if a connection goes down, the client will need to manually re-establish it.

## Server-Sent Events

**Server-Sent Events** provide a persistent, unidirectional communication channel that reconnects automatically.

Similar to Long Polling, communication is unidrectional from server to client. But then like with Websockets, we avoid the overhead of dealing with headers / time spent establishing connections since we create persistent connections. Of course, that also means we need to ensure we don't maintain too many inactive connections and waste resources

Automatic connection re-establishment is a convenient feature, however it might not necessarily be good. The **thundering herd problem** could occurr if the server goes down. When it comes back online it will try to re-establish connections with all the clients _at once_, causing a huge strain and potentially knocking it out again

A way to mitigate this is to use some random jitter similar to the leader candidate proposal process in the Raft distributed consensus algorithm

## Additional Reading / Material

- KhanAcademy ["Computers and the Internet: User Datagram Protocol"](https://www.khanacademy.org/computing/computers-and-internet/xcae6f4a7ff015e7d:the-internet/xcae6f4a7ff015e7d:transporting-packets/a/user-datagram-protocol-udp)
- PubNub ["What is Long Polling?"](https://www.pubnub.com/guides/long-polling/)
- AWS Cloud Computing Concepts Hub ["What is OSI Model?"](https://aws.amazon.com/what-is/osi-model/)
- **jordanhasnolife System Design 2.0 Playlist**:
  - ["TCP vs. UDP in 12 minutes"](https://www.youtube.com/watch?v=hPSsPCNxta4&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=58)
  - ["Long Polling, Websockets, Server-Sent Events"](https://www.youtube.com/watch?v=fIwOd4PToAY&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=59)
