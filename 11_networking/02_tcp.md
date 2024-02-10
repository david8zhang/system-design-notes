# TCP: Transmission Control Protocol

TCP, or Transmission Control Protocol, is a network protocol which provides reliable 2-way 1:1 communication through the use of handshakes and acknowledgements. TCP is used for most applications where reliability is crucial.

The process for establishing a connection is via a 3-way handshake:

1. Send a SYN with a sequence number x
2. Respond with an ACK with a sequence number x + 1 and a SYN with sequence number y
3. Respond with ACK with a sequence number y + 1

TCP tries to guarantee reliable delivery using sequence numbers to confirm message receipt. This operates similarly to the handshake, send a message with a sequence number and expect an acknowledgment with the sequence number + 1

TCP also uses timeouts to retry whenever it doesn't receive an acknowledgement. For example, it sends a message with a sequence number, then waits for a set period before retrying the message. The amount of time it waits between retries increases exponentially; this is a strategy known as _exponential backoff_.

## Flow and Congestion control

**Flow control** deals with ensuring that the sender isn't sending more messages than a receiver can consume. TCP does this by looking at the receiver buffer to limit the number of messages in flight. It then resets the limit upon receiving an acknowledgment.

**Congestion control** deals with ensuring that message traffic does not degrade network response times. TCP handles this by setting a window of how many messages can be sent at once. It then uses _additive increase, multiplicative decrease_ (AIMD) to adjust the window size. AIMD dictates that the window should grow in size linearly when there's no congestion, but reduce in size exponentially when there is.
