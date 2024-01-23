# Server-Sent Events

**Server-Sent Events** provide a persistent, unidirectional communication channel that reconnects automatically.

Similar to Long Polling, communication is unidrectional from server to client. But then like with Websockets, we avoid the overhead of dealing with headers / time spent establishing connections since we create persistent connections. Of course, that also means we need to ensure we don't maintain too many inactive connections and waste resources

Automatic connection re-establishment is a convenient feature, however it might not necessarily be good. The **thundering herd problem** could occurr if the server goes down. When it comes back online it will try to re-establish connections with all the clients _at once_, causing a huge strain and potentially knocking it out again

A way to mitigate this is to use some random jitter similar to the leader candidate proposal process in the Raft distributed consensus algorithm.
