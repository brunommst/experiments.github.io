# Reliable UDP – NDI v5

In NDI version 5 the default communication mechanism is a Reliable UDP protocol that represents the state-of-the-art communication protocol that is implemented by building upon all the experience we have seen in the real world with NDI across a massive variety of different installations

Reliable UDP, also known as RUDP, is a transport protocol that combines the advantages of UDP's low latency and simplicity with the reliability of TCP (Transmission Control Protocol). It is designed specifically for real-time multimedia applications, where maintaining the timeliness of data is crucial.

In the context of NDI, Reliable UDP is employed to ensure that video and audio streams are delivered reliably and with minimal delay. It achieves this by implementing several mechanisms:

Sequencing: Reliable UDP assigns a sequence number to each packet it sends. This allows the receiving end to detect missing or out-of-order packets and request retransmissions if necessary.

Retransmissions: If a packet is lost or arrives out of order, the receiving end can request a retransmission of the missing packet(s) using the sequence number information.

Flow control: Reliable UDP incorporates flow control mechanisms to manage the rate of data transmission. This prevents overwhelming the network or the receiving device with more data than it can handle, ensuring a smoother streaming experience.

Congestion control: RUDP also includes congestion control algorithms to prevent network congestion and avoid unnecessary packet loss. It dynamically adjusts the transmission rate based on network conditions, maintaining optimal throughput without overwhelming the network.
