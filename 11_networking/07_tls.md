# TLS: Transport Layer Security

TLS is a cryptographic protocol used to protect communications over a computer network. You probably know it as providing the "S" in "HTTPS". TLS ensures that data sent between clients and servers can't be eavesdropped on or tampered with.

TLS is typically thought of as "running on top of some reliable transport protocol", given that the name literally stands for "Transport layer Security". However, it's not a strictly L4 level protocol since it serves encryption to higher layers. Applications also need to actively initiate _TLS handshakes_ and handle digital certificates.

## TLS Handshake

When a client and server agree to use TLS, they use a handshake procedure to agree on the keys that they'll both use to encrypt/decrypt their data. The handshake goes like this:

1. A client connects to a TLS enabled server and presents a list of supported _cipher suites_ and hash functions. These include algorithms for exchanging keys, encrypting data, and verifying data integrity.

2. The server picks a cipher and hash function and notifies the client

3. The server then provides identification in the form of a digital certificate. The certificate contains the server name, trusted certificate authority that vouches for its authenticity, and a public encryption key.

4. The client confirms the validity of the certificate

5. The client and server now generate session keys in one of two ways:

   a. The client encrypts a random number with the server's public encryption key, which the server then decrypts with its private key. Both then use this random number to generate a unique session key for subsequent encryption/decryption.

   b. The client and server use the [Diffie-Helman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) key exchange algorithm to securely generate a random and unique session key. This gives us the nice benefit of ensuring that the session cannot be decrypted even if the private key leaks on the server, which is known as _forward secrecy_

## TLS Termination

Performing this key exchange and subsequent encryption/decryption of data can be CPU intensive on our application servers. A common pattern to mitigate this is to _terminate_ TLS using a proxy server (like a load balancer), and use regular HTTP when communicating between servers internally in our system.

Doing this comes with all the added complexity of managing proxy load-balancer servers - we'd want to do this in a fault-tolerant way and without bottlenecking our system, which we talk about a bit more in the [load balancers](/topic/12_load_balancing) section. Furthermore, if the proxy is compromised, then all our data throughout our system is available since it's unencrypted.
