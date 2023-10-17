# NDI-RECV

The **NDI Receive SDK** is how frames are received over the network. It is important to be aware that it can connect to sources and remain “connected” to them even when they are no longer available on the network; it will automatically reconnect if the source becomes available again.

### Parameters

As with the other APIs, the starting point is to use the `NDIlib_recv_create_v3` function. This function may be initialized with `NULL` , and default settings are used. This takes parameters defined by `NDIlib_recv_create_v3_t`, as follows:

<table data-full-width="true"><thead><tr><th width="258">Supported Parameters</th><th>Description</th></tr></thead><tbody><tr><td><strong>source_to_connect_to</strong></td><td><p>This is the source name that should be connected to. This is in the exact format returned by <code>NDIlib_find_get_sources</code>.</p><p></p><p>Note that you may specify the source as a <code>NULL</code> source if you wish to create a receiver that you desire to connect at a later point with <code>NDIlib_recv_connect</code>.</p></td></tr><tr><td><strong>p_ndi_name</strong></td><td>This is a name that is used for the receiver and will be used in future versions of the SDK to allow discovery of both senders and receivers on the network. This can be specified as <code>NULL</code> and a unique name based on the application executable name will be used.</td></tr><tr><td><strong>color_format</strong></td><td>This parameter determines what color formats you are passed when a frame is received. In general, there are two color formats used in any scenario: one that exists when the source has an alpha channel and another when it does not.</td></tr></tbody></table>

> The **following table** lists the optional values that can be used to specify the color format to be returned.
>
>

<table data-full-width="false"><thead><tr><th width="363.3333333333333">Optional color_format values</th><th>Frames without alpha</th><th>Frames with alpha</th></tr></thead><tbody><tr><td><strong>NDIlib_recv_color_format_BGRX_BGRA</strong></td><td>BGRX</td><td>BGRA</td></tr><tr><td><strong>NDIlib_recv_color_format_UYVY_BGRA</strong></td><td>UYVY</td><td>BGRA</td></tr><tr><td><strong>NDIlib_recv_color_format_RGBX_RGBA</strong></td><td>RGBX</td><td>RGBA</td></tr><tr><td><strong>NDIlib_recv_color_format_UYVY_RGBA</strong></td><td>UYVY</td><td>RGBA</td></tr><tr><td><strong>NDIlib_recv_color_format_fastest</strong></td><td>Normally UYVY. See notes below.</td><td>Normally UYVA. See notes below.</td></tr><tr><td><strong>NDIlib_recv_color_format_best</strong></td><td>Varies. See notes below.</td><td>Varies. See notes below.</td></tr></tbody></table>

> **color\_format notes**
>
> If you specify the color option `NDIlib_recv_color_format_fastest`, the SDK will provide buffers in the format that it processes internally without performing any conversions before they are passed to you. This results in the best possible performance. This option also typically runs with lower latency than other options since it supports single-field format types.
>
> The `allow_video_fields` option is assumed to be true in this mode. On most platforms, this will return an 8-bit UYVY video buffer when there is no alpha channel and an 8-bit UYVY+A buffer when there is. These formats are described in the description of the video layout.
>
> If you specify the color option `NDIlib_recv_color_format_best`, the SDK will provide you buffers in the format closest to the native precision of the video codec being used. In many cases, this is both high-performance and high-quality and results in the best quality.
>
> Like the `NDIlib_recv_color_format_fastest`, this format will always deliver individual fields, implicitly assuming the allow\_video\_fields option as true.
>
> On most platforms, when there is no alpha channel, this will return either a 16-bpp Y+Cb,Cr (P216 FourCC) buffer when the underlying codec is native NDI, or an 8-bpp UYVY buffer when the native codec is an 8-bit codec like H.264. When there is an alpha channel, this will normally return a 16-bpp Y+Cb,Cr+A (PA16 FourCC) buffer.
>
> You should support the `NDIlib_video_frame_v2_t` properties as widely as you possibly can in this mode since there are very few restrictions on what you might be passed.
