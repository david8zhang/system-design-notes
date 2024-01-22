# UDP: User Datagram Protocol

UDP, or User Datagram Protocol, is a simple, connectionless communication model without any of the reliability mechanisms that TCP provides. It's fairly bare bones - you just send packets from one node to another with some checksums for data integrity.

For that reason, it's super fast, and is used for low-latency scenarios where dropped data is acceptable. (e.g. video calls, video games, real-time tracking of stock prices). It also supports **multicast**, which is when a single packet is routed to multiple places.

## Checksums

**Checksums** are sent with every UDP header to ensure that packets aren't corrupted. Before sending off the segment, the sender:

1. Computes the checksum based on the data in the segment
2. Stores the computed checksum in the final two bytes of the UDP header

Upon receiving the data, the recipient

1. Computes the checksum based on the received segment
2. Compares it to the checksum in the header it received. If they aren't equal, it knows data was corrupted.
