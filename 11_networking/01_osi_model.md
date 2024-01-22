# OSI Model

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
