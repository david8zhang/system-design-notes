## Websockets

**WebSocket** is a communications protocol which provides persistent two-way communication between client and server. It's implemented over a single TCP connection, utilizes a special handshake incorporating the _HTTP Upgrade Header_ to change from the HTTP protocol to WebSocket protocol, and has its own custom URI scheme: (`ws://` for WebSocket) and (`wss://` for Websocket Secure).

Since it's _persistent_, we avoid the overhead of creating new connections and dealing with request headers. However, if these connections are largely inactive, we might waste resources. In addition, if a connection goes down, the client will need to manually re-establish it.
