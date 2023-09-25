# Multipath TCP – NDI v4

This protocol permits transport across multiple NICs and all network paths, it is intended to use hardware-accelerated network adapters with adaptive bandwidth sharing across NICs.

Multipath TCP is a transmission protocol that offers advantages such as maximizing throughput, optimizing resource usage, and enhancing network redundancy. It can seamlessly integrate multiple network pathways, including wireless and mobile networks. It is especially efficient when used with NDI (equipment that utilizes multiple Gigabit connections to exchange a large number of NDI streams.

However, in scenarios where 10Gbit interfaces are connected with 1Gbit interfaces, Multipath TCP's efficiency is compromised. This is primarily due to network switches being unable to effectively manage network congestion in such situations. As a result, the protocol may not perform optimally in these specific network configurations.
