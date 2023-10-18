---
description: >-
  NDI sending and receiving use common structures to define video, audio, and
  metadata types. The parameters of these structures are documented below.
---

# Frame Types

### Video Frames (NDILIB\_VIDEO\_FRAME\_V2\_T)

<table data-full-width="true"><thead><tr><th>Parameters</th><th>Description</th></tr></thead><tbody><tr><td><strong>xres, yres (int)</strong></td><td>This is the resolution of the frame expressed in pixels. Note that, because data is internally all considered in 4:2:2 formats, image width values should be divisible by two.</td></tr><tr><td><p><strong>FourCC</strong> </p><p><strong>(NDIlib_FourCC_video_type_e)</strong></p></td><td>This is the pixel format for this buffer. <strong>The supported formats are listed in the table below</strong>.</td></tr></tbody></table>

<table data-full-width="false"><thead><tr><th width="274">FourCC</th><th>Description</th></tr></thead><tbody><tr><td></td><td></td></tr><tr><td><strong>NDIlib_FourCC_type_UYVY</strong></td><td><p>This is a buffer in the “UYVY” FourCC and represents a 4:2:2 image in YUV color space. There is a Y sample at every pixel, and U and V sampled at every second pixel horizontally on each line. A macro-pixel contains 2 pixels in 1 DWORD. The ordering of these pixels is U0, Y0, V0, Y1.</p><p></p><p>Please see the notes below regarding the expected YUV color space for different resolutions.</p><p></p><p>Note that when using UYVY video, the color space is maintained end-to-end through the pipeline, which is consistent with how almost all video is created and displayed.</p></td></tr><tr><td><strong>NDIlib_FourCC_type_UYVA</strong></td><td><p>This is a buffer that represents a 4:2:2:4 image in YUV color space. There is a Y sample at every pixels with U,V sampled at every second pixel horizontally. There are two planes in memory, the first being the UYVY color plane, and the second the alpha plane that immediately follows the first.</p><p></p><p>For instance, if you have an image with <code>p_data</code> and <code>stride</code>, then the planes are located as follows:</p><pre><code>    uint8_t *p_uyvy = (uint8_t*)p_data; 
    uint8_t *p_alpha = p_uyvy + stride*yres;
</code></pre></td></tr><tr><td><strong>NDIlib_FourCC_type_P216</strong></td><td><p>This is a 4:2:2 buffer in semi-planar format with full 16bpp color precision. This is formed from two buffers in memory, the first is a 16bpp luminance buffer and the second is a buffer of U,V pairs in memory. This can be considered as a 16bpp version of NV12.</p><p></p><p>For instance, if you have an image with <code>p_data</code> and <code>stride</code>, then the planes are located as follows:</p><pre class="language-json"><code class="lang-json">    uint16_t *p_y = (uint16_t*)p_data;
    uint16_t *p_uv = (uint16_t*)(p_data + stride*yres);
</code></pre><p>As a matter of illustration, a completely packed image would have stride as <code>xres*sizeof(uint16_t)</code>.</p></td></tr><tr><td><strong>NDIlib_FourCC_type_PA16</strong></td><td><p>This is a 4:2:2:4 buffer in semi-planar format with full 16bpp color and alpha precision. This is formed from three buffers in memory. The first is a 16bpp luminance buffer, and the second is a buffer of U,V pairs in memory. A single plane alpha channel at 16bpp follows the U,V pairs.</p><p></p><p>For instance, if you have an image with p_data and stride, then the planes are located as follows:</p><pre class="language-json"><code class="lang-json">    uint16_t *p_y = (uint16_t*)p_data; 
    uint16_t *p_uv = p_y + stride*yres; 
    uint16_t *p_alpha = p_uv + stride*yres;
</code></pre><p>To illustrate, a completely packed image would have stride as</p><p><code>xres*sizeof(uint16_t)</code>.</p></td></tr><tr><td><strong>NDIlib_FourCC_type_YV12</strong></td><td><p>This is a planar 4:2:0 in Y, U, V planes in memory.</p><p></p><p>For instance, if you have an image with p_data and stride, then the planes are located as follows:</p><pre class="language-json"><code class="lang-json">    uint8_t *p_y = (uint8_t*)p_data;
    uint8_t *p_v = p_y + stride*yres; 
    uint8_t *p_u = p_v + (stride/2)*(yres/2);
</code></pre><p>As a matter of illustration, a completely packed image would have stride as <code>xres*sizeof(uint8_t)</code>.</p></td></tr><tr><td><strong>NDIlib_FourCC_type_I420</strong></td><td><p>This is a planar 4:2:0 in Y,U,V planes in memory with the U,V planes reversed from the YV12 format.</p><p></p><p>For instance, if you have an image with p_data and stride, then the planes are located as follows:</p><pre class="language-json"><code class="lang-json">    uint8_t *p_y = (uint8_t*)p_data;
    uint8_t *p_v = p_y + stride*yres; 
    uint8_t *p_u = p_v + (stride/2)*(yres/2);
</code></pre><p>To illustrate, a completely packed image would have stride as</p><p><code>xres*sizeof(uint8_t)</code>.</p></td></tr><tr><td><strong>NDIlib_FourCC_type_NV12</strong></td><td><p>This is a semi planar 4:2:0 in Y, UV planes in memory. The luminance plane is at the lowest memory address with the UV pairs immediately following them.</p><p>For instance, if you have an image with p_data and stride, then the planes are located as follows:</p><pre class="language-json"><code class="lang-json">      uint8_t *p_y = (uint8_t*)p_data;
      uint8_t *p_uv = p_y + stride*yres;
</code></pre><p>To illustrate, a completely packed image would have stride as</p><p>xres*sizeof(uint8_t).</p></td></tr><tr><td><strong>NDIlib_FourCC_type_BGRA</strong></td><td>A 4:4:4:4, 8-bit image of red, green, blue and alpha components, in memory order blue, green, red, alpha. This data is not pre-multiplied.</td></tr><tr><td><strong>NDIlib_FourCC_type_BGRX</strong></td><td><p>A 4:4:4, 8-bit image of red, green, blue components, in memory order blue, green, red, 255. This data is not pre-multiplied.</p><p>This is identical to BGRA, but is provided as a hint that all alpha channel values are 255, meaning that alpha compositing may be avoided. The lack of an alpha channel is used by the SDK to improve performance when possible.</p></td></tr><tr><td><strong>NDIlib_FourCC_type_RGBA</strong></td><td>A 4:4:4:4, 8-bit image of red, green, blue and alpha components, in memory order red, green, blue, alpha. This data is not pre-multiplied.</td></tr><tr><td><strong>NDIlib_FourCC_type_RGBX</strong></td><td><p>A 4:4:4, 8-bit image of red, green, blue components, in memory order red, green, blue, 255. This data is not pre-multiplied.</p><p>This is identical to RGBA, but is provided as a hint that all alpha channel values are 255, meaning that alpha compositing may be avoided. The lack of an alpha channel is used by the SDK to improve performance when possible.</p></td></tr></tbody></table>

When running in a YUV color space, the following standards are applied:

<table data-full-width="false"><thead><tr><th width="379">Resolution</th><th>Standard</th></tr></thead><tbody><tr><td><strong>SD resolutions</strong></td><td>BT.601</td></tr><tr><td><p><strong>HD resolutions</strong> </p><p><strong><code>xres>720 || yres>576</code></strong></p></td><td>Rec.709 </td></tr><tr><td><strong>UHD resolutions</strong><br><strong><code>xres>1920 || yres>1080</code></strong></td><td>Rec.2020</td></tr><tr><td><strong>Alpha channel</strong></td><td>Full range for data type <br>(0-255 range when running 8-bit and 0-65536 range when running 16-bit.)</td></tr></tbody></table>

For the sake of compatibility with standard system components, Windows APIs expose 8-bit UYVY and RGBA video (common FourCCs used in all media applications).

<table data-full-width="true"><thead><tr><th>Parameters (Cont.)</th><th>Description</th></tr></thead><tbody><tr><td><strong>frame_rate_N, frame_rate_D (int)</strong></td><td><p>This is the framerate of the current frame. The framerate is specified as a numerator and denominator, such that the following is valid:<br></p><p><code>frame_rate = (float)frame_rate_N / (float)frame_rate_D</code><br></p><p><strong>Some examples of common framerates are presented in the table below</strong>.</p></td></tr></tbody></table>

| Standard        | Framerate ratio | Framerate |
| --------------- | --------------- | --------- |
| NTSC 1080i59.94 | 30000 / 1001    | 29.97 Hz  |
| NTSC 720p59.94  | 60000 / 1001    | 59.94 Hz  |
| PAL 1080i50     | 30000 / 1200    | 25 Hz     |
| PAL 720p50      | 60000 / 1200    | 50 Hz     |
| NTSC 24fps      | 24000 / 1001    | 23.98 Hz  |

<table data-full-width="true"><thead><tr><th>Parameters (Cont.)</th><th>Description</th></tr></thead><tbody><tr><td><strong>picture_aspect_ratio (float)</strong></td><td>The SDK defines <em>picture</em> aspect ratio (as opposed to pixel aspect ratios). Some common aspect ratios are presented in the table below. When the aspect ratio is 0.0 it is interpreted as xres/yres, or square pixel; for most modern video types this is a default that can be used.</td></tr></tbody></table>

| Aspect Ratio | Calculated ad | image\_aspect\_ratio |
| ------------ | ------------- | -------------------- |
| 4:3          | 4.0/3.0       | 1.333...             |
| 16:9         | 16.0/9.0      | 1.667...             |
| 16:10        | 16.0/10.0     | 1.6                  |

<table data-full-width="true"><thead><tr><th>Parameters (Cont.)</th><th>Description</th></tr></thead><tbody><tr><td><strong>frame_format_type (NDIlib_frame_format_type_e)</strong></td><td>This is used to determine the frame type. Possible values are listed in the next table.</td></tr></tbody></table>

<table data-full-width="false"><thead><tr><th>Value</th><th>Description</th></tr></thead><tbody><tr><td><strong>NDIlib_frame_format_type_progressive</strong></td><td>This is a progressive video frame</td></tr><tr><td><strong>NDIlib_frame_format_type_interleaved</strong></td><td>This is a frame of video that is comprised of two fields. The upper field comes first, and the lower comes second (see note below)</td></tr><tr><td><strong>NDIlib_frame_format_type_field_0</strong></td><td>This is an individual field 0 from a fielded video frame. This is the first temporal, upper field (see note below).</td></tr><tr><td><strong>NDIlib_frame_format_type_field_1</strong></td><td>This is an individual field 1 from a fielded video frame. This is the second temporal, lower field (see note below).</td></tr></tbody></table>

{% hint style="info" %}
To make everything as easy to use as possible, the SDK always assumes that fields are ‘**top field first**.’
{% endhint %}

This is, in fact, the case for every modern format, but does create a problem for two specific older video formats as discussed below:

#### NTSC 486 Lines

The best way to handle this format is simply to offset the image vertically by one line `(p_uyvy_data + uyvy_stride_in_bytes)` and reduce the vertical resolution to 480 lines. This can all be done without modification of the data being passed in at all; simply change the data and resolution pointers.

#### DV NTSC

This format is a relatively rare these days, although still used from time to time. There is no entirely trivial way to handle this other than to move the image down one line and add a black line at the bottom.

<table data-full-width="true"><thead><tr><th width="388">Parameters (Cont.)</th><th>Description</th></tr></thead><tbody><tr><td><strong>timecode (int64_t, 64-bit signed integer)</strong></td><td>This is the timecode of this frame in 100 ns intervals. This is generally not used internally by the SDK but is passed through to applications, which may interpret it as they wish. When sending data, a value of <code>NDIlib_send_timecode_synthesize</code> can be specified (and should be the default). The operation of this value is documented in the sending section of this documentation.</td></tr><tr><td><strong>p_data (const uint8_t*)</strong></td><td>This is the video data itself laid out linearly in memory in the FourCC format defined above. The number of bytes defined between lines is specified in <code>line_stride_in_bytes</code>. No specific alignment requirements are needed, although larger data alignments might result in higher performance (and the internal SDK codecs will take advantage of this where needed).</td></tr><tr><td><strong>line_stride_in_bytes (int)</strong></td><td>This is the inter-line stride of the video data, in bytes.</td></tr><tr><td><strong>p_metadata (const char*)</strong></td><td>This is a per-frame metadata stream that should be in UTF-8 formatted XML and <code>NULL</code>-terminated. It is sent and received with the frame.</td></tr><tr><td><strong>timestamp (int64_t, 64-bit signed integer)</strong></td><td><p>This is a per-frame timestamp filled in by the NDI SDK using a high precision clock. It represents the time (in 100 ns intervals measured in UTC time, since the Unix Time Epoch 1/1/1970 00:00) when the frame was submitted to the SDK.</p><p>On modern sender systems this will have ~1 μs accuracy; this can be used to synchronize streams on the same connection, between connections, and between machines. For inter-machine synchronization, it is important to use external clock locking capability with high precision (such as NTP).</p></td></tr></tbody></table>

### Audio Frames (NDILIB\_AUDIO\_FRAME\_V3\_T)

NDI Audio is passed to the SDK in floating-point and has a dynamic range without practical limits (without clipping). To define how floating-point values map into real-world audio levels, a sinewave that is 2.0 floating-point units peak-to-peak (i.e., -1.0 to +1.0) is assumed to represent an audio level of +4 dBU, corresponding to a nominal level of 1.228 V RMS.

**Two tables are provided below** that explain the relationship between NDI audio values for the SMPTE and EBU audio standards.

**SMPTE Audio levels - reference Level**

<table data-header-hidden data-full-width="true"><thead><tr><th width="202"></th><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td>NDI</td><td>0.0</td><td>0.063</td><td>0.1</td><td>0.63</td><td>1.0</td><td>10.0</td></tr><tr><td>dBu</td><td>-∞</td><td>-20 dB</td><td>-16 dB</td><td>+0 dB</td><td>+4 dB</td><td>+24 dB</td></tr><tr><td>dBVU</td><td>-∞</td><td>-24 dB</td><td>-20 dB</td><td>-4 dB</td><td>+0 dB</td><td>+20 dB</td></tr><tr><td>SMPTE dBFS</td><td>-∞</td><td>-44 dB</td><td>-40 dB</td><td>-24 dB</td><td>-20 dB</td><td>+0 dB</td></tr></tbody></table>

