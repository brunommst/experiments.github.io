# UDP with Forward Error Correction – NDI v3

This alternative protocol to TCP is used when reliable delivery of data packets is not required. UDP is typically used for applications where timeliness is of higher priority than accuracy, such as streaming media, teleconferencing, and voice-over-IP (VoIP). Forward error correction (FEC) is a method of obtaining error control in data transmission in which the source (transmitter) sends redundant data and the destination (receiver).

UDP (User Datagram Protocol) with Forward Error Correction (FEC) is a beneficial approach when the network is prone to errors or not entirely reliable. It provides a solution for error correction when data packets get lost or corrupted during transmission.

However, it's important to note that using UDP with FEC requires additional computational processing on the receiver side. The receiver needs to implement algorithms and mechanisms to manage the error correction process. This involves decoding the received data and applying error correction techniques to recover any lost or corrupted packets.
