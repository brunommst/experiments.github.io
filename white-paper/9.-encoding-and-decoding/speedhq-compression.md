# SpeedHQ Compression

NDI uses compression to enable the transmission of many video streams across existing infrastructure, specifically discrete cosine transform (DCT), which converts video signals into elementary frequency components. This method of compression is commonly used in encoding formats and mezzanine codecs within the industry.

One of the most efficient codecs in existence, NDI achieves significantly better compression than many codecs that have been accepted for professional broadcast use. On a typical, modern Intel-based i7 processor, the codec can compress a video stream to the following benchmarks:

**The NDI codec's peak signal-to-noise ratio (PSNR) exceeds 70dB for typical video content.**

Uniquely and importantly, NDI is the first ever codec to provide multi-generational stability.\
This means there is no further loss once a video signal is compressed. As a practical example, generation 2 and generation 1000 of a decode-to-encode sequence would be identical.

The NDI codec is designed to run very fast and is largely implemented in hand-written assembly to ensure that the process of compressing video frames occurs as quickly as possible. Latency is both a factor of the network connection and the endpoint products. NDI has a technical latency of 16 video scan lines, although in practice, most implementations would be one field of latency. Hardware implementations can provide full end-to-end latency of within 8 scan lines.
