# Timestamp, NDI Advanced SDK – AV Sync API

NDI transmitters and receivers utilize the system clock as a point of reference. This system clock can be aligned with robust sources like NTP or PTP. The timing details from these sources are integrated into the NDI Stream through timestamps embedded in each video and audio frame (with audio frames being smaller than video frames).

Subsequently, the receiver can realign audio and video sources using this timestamp information. This method provides an ingenious approach to achieving synchronization, making it an ideal solution for remote or cloud-based production scenarios.
