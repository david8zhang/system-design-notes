# Long Polling

In traditional HTTP communication, the client sends requests to the server and waits for a response. Once received, the client needs to make a brand new requests to receive new data. This is "short polling", which might not be efficient for real time scenarios since we'd need to keep sending requests to the server.

Long Polling, on the other hand, keeps the requests open until new data is available. Once it receives that data, it completes the request and creates a new one. Since creating new connections is expensive, long polling is best used for situations where data is rarely needed. In addition, it's **unidirectional**, since data is only sent from the server to the client.
