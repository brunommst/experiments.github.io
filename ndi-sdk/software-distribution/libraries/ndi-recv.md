# NDI-Recv

### NDI-Recv

The NDI® Receive SDK is how frames are received over the network. It is important to be aware that it can connect to sources and remain “connected” to them even when they are no longer available on the network; it will automatically reconnect if the source becomes available again.

As with the other APIs, the starting point is to use the NDIlib\_recv\_create\_v3 function. This function may be initialized with NULL and default settings are used. This takes parameters defined by NDIlib\_recv\_create\_v3\_t_,_ as follows:

| Supported Parameters    |                                                                                                                                                                                                                                                                                                           |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| source\_to\_connect\_to | <p>This is the source name that should be connected too. This is in the exact format returned by NDIlib_find_get_sources<em>.</em></p><p>Note that you may specify the source as a NULL source if you wish to create a receiver that you desire to connect at a later point with NDIlib_recv_connect.</p> |
| p\_ndi\_name            | This is a name that is used for the receiver and will be used in future versions of the SDK to allow discovery of both senders and receivers on the network. This can be specified as NULL and a unique name based on the application executable name will be used.                                       |
| color\_format           | This parameter determines what color formats you are passed when a frame is received. In general, there are two color formats used in any scenario: one that exists when the source has an alpha channel, and another when it does not.                                                                   |

The following table lists the optional values that can be used to specify the color format to be returned.

| Optional color\_format values           | Frames without alpha            | Frames with alpha               |
| --------------------------------------- | ------------------------------- | ------------------------------- |
| NDIlib\_recv\_color\_format\_BGRX\_BGRA | BGRX                            | BGRA                            |
| NDIlib\_recv\_color\_format\_UYVY\_BGRA | UYVY                            | BGRA                            |
| NDIlib\_recv\_color\_format\_RGBX\_RGBA | RGBX                            | RGBA                            |
| NDIlib\_recv\_color\_format\_UYVY\_RGBA | UYVY                            | RGBA                            |
| NDIlib\_recv\_color\_format\_fastest    | Normally UYVY. See notes below. | Normally UYVA. See notes below. |
| NDIlib\_recv\_color\_format\_best       | Varies. See notes below.        | Varies. See notes below.        |

color\_format notes

If you specify the color option NDIlib\_recv\_color\_format\_fastest, the SDK will provide buffers in the format that it processes internally without performing any conversions before they are passed to you. This results in the best possible performance. This option also typically runs with lower latency than other options since it supports single-field format types.

The allow\_video\_fields option is assumed to be true in this mode. On most platforms this will return an 8-bit UYVY video buffer when there is no alpha channel, and an 8-bit UYVY+A buffer when there is. These formats are described in the description of the video layout.

If you specify the color option NDIlib\_recv\_color\_format\_best, the SDK will provide you buffers in the format closest to the native precision of the video codec being used. In many cases this is both high-performance and high-quality and results in the best quality.

Like the NDIlib\_recv\_color\_format\_fastest,this format will always deliver individual fields, implicitly assuming the allow\_video\_fields option as true.

On most platforms, when there is no alpha channel this will return either a 16-bpp Y+Cb,Cr (P216 FourCC) buffer when the underlying codec is native NDI, or a 8-bpp UYVY buffer when the native codec is an 8-bit codec like H.264. When there is alpha channel this will normally return a 16-bpp Y+Cb,Cr+A (PA16 FourCC) buffer.

You should support the NDIlib\_video\_frame\_v2\_t properties as widely as you possibly can in this mode, since there are very few restrictions on what you might be passed.

| Supported Parameters (Continued) |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| bandwidth                        | <p>This allows you to specify whether this connection is in high or low bandwidth mode. It is an enumeration because other alternatives may be available in future. For most uses you should specify NDIlib_recv_bandwidth_highest, which will result in the same stream that is being sent from the up-stream source to you.</p><p>You may specify NDIlib_recv_bandwidth_lowest, which will provide you with a medium quality stream that takes significantly reduced bandwidth.</p> |
| allow\_video\_fields             | <p>If your application does not like receiving fielded video data you can specify false to this value, and all video received will be de-interlaced before it is passed to you.</p><p>The default value should be considered true for most applications. The implied value is true when color_format is NDIlib_recv_color_format_fastest.</p>                                                                                                                                         |
| p\_ndi\_name                     | This is the name of the NDI receiver to create. It is a NULL-terminated UTF-8 string. Give your receiver a meaningful, descriptive, and unique name. This will be the name of the NDI receiver on the network. For instance, if your network machine name is called “MyMachine” and you specify this parameter as “Video Viewer”, then the NDI receiver on the network would be “MyMachine (Video Viewer)”.                                                                           |

Once you have filled out this structure, calling NDIlib\_recv\_create\_v3 will create an instance for you. A full example is provided with the SDK that illustrates finding a network source and creating a receiver to view it (we will not reproduce that code here).

If you create a receiver with NULL as the settings, or if you wish to change the remote source that you are connected to, you may call NDIlib\_recv\_connect at any time with a NDIlib\_source\_t pointer. If the source pointer is NULL it will disconnect you from any sources to which you are connected.

Once you have a receiving instance you can query it for video, audio, or metadata frames by calling NDIlib\_recv\_capture\_v3. This function takes a pointer to the header for audio (NDIlib\_audio\_frame\_v3\_t), video (NDIlib\_video\_frame\_v2\_t), and metadata (NDIlib\_metadata\_frame\_t), any of which can be NULL. It can safely be called across many threads at once, allowing you to have one thread receiving video while another receives audio.

The NDIlib\_recv\_capture\_v3 function takes a timeout value specified in milliseconds. If a frame is available when you call NDIlib\_recv\_capture\_v3, it will be returned without any internal waiting or locking of any kind.  If the timeout is zero, it will return immediately with a frame if there is one. If the timeout is not zero, it will wait for a frame up to the timeout duration specified and return if it gets one (if there is already a frame waiting when the call is made it returns that frame immediately). If a frame of the type requested has been received before the timeout occurs, the function will return the data type received. Frames returned to you by this function must be freed.

The following code illustrates how one might receive audio and/or video based on what is available; it will wait one second before returning if no data was received;

NDIlib\_video\_frame\_v2\_t video\_frame;

NDIlib\_audio\_frame\_v3\_t audio\_frame;

NDIlib\_metadata\_frame\_t metadata\_frame;

switch (NDIlib\_recv\_capture\_v3(pRecv, \&video\_frame, \&audio\_frame, \&metadata\_frame, 1000))

{

// We received video.

case NDIlib\_frame\_type\_video:

// Process video here

// Free the video.

NDIlib\_recv\_free\_video\_v2(pRecv, \&video\_frame);

break;

// We received audio.

case NDIlib\_frame\_type\_audio:

// Process audio here

// Free the audio.

NDIlib\_recv\_free\_audio\_v3(pRecv, \&audio\_frame);

break;

// We received a metadata packet

case NDIlib\_frame\_type\_metadata:

// Do what you want with the metadata message here.

// Free the message

NDIlib\_recv\_free\_metadata(pRecv, \&metadata\_frame);

break;

// No audio or video has been received in the time-period.

case NDIlib\_frame\_type\_none:

break;

// The device has changed status in some way (see notes below)

case NDIlib\_frame\_type\_status\_change:

break;

}

You are able, if you wish, to take the received video, audio, or metadata frames and free them on another thread to ensure there is no chance of dropping frames while receiving them. A short queue is maintained on the receiver to allow you to process incoming data in the fashion most convenient for your application. If you always process buffers faster than real-time this queue will always be empty, and you will be running at the lowest possible latency.

NDIlib\_recv\_capture\_v3 may return the value NDIlib\_frame\_type\_status\_change, to indicate that the device’s properties have changed. Because connecting to a video source might take a few seconds, some of the properties of that device are not known immediately and might even change on the fly. For instance, when connecting to a PTZ camera, it might not be known for a few seconds that it supports the PTZ command set.

When this does become known, the value NDIlib\_frame\_type\_status\_change is returned to indicate that you should recheck device properties. This value is currently sent when a source changes PTZ type, recording capabilities or web user interface control.

If you wish to determine whether any audio, video or metadata frames have been dropped, you can call NDIlib\_recv\_get\_performance, which will supply the total frame count and the number of frames that have been dropped because they could not be de-queued fast enough.

If you wish to determine the current queue depths on audio, video, or metadata (to poll whether receiving a frame would immediately give a result), you can call NDIlib\_recv\_get\_queue.

NDIlib\_recv\_get\_no\_connections will return the number of connections that are currently active and can also be used to detect whether the video source you are connected to is currently online or not. Additional functions provided by the receive SDK allow metadata to be passed upstream to connected sources via NDIlib\_recv\_send\_metadata. Much like the sending of metadata frames in the NDI Send SDK, this is passed as an NDIlib\_metadata\_frame\_t structure that is to be sent.

Tally information is handled via NDIlib\_recv\_set\_tally_._ This will take a NDIlib\_tally\_t structure that can be used to define the program and preview visibility status. The tally status is retained within the receiver so that, even if a connection is lost, the tally state is correctly set when it is subsequently restored.

Connection metadata is an important concept that allows you to “register” certain metadata messages so that each time a new connection is established the up-stream source (normally an NDI Send user) receives those strings. Note that there are many reasons that connections might be lost and established at run time.

For instance, if an NDI-Sender goes offline the connection is lost; if it comes back online at a later time, the connection would be re-established, and the connection metadata would be resent. Some standard connection strings are specified for connection metadata, as outlined in the next section.

Connection metadata strings are added with NDIlib\_recv\_add\_connection\_metadata that takes an NDIlib\_metadata\_frame\_t structure. To clear all connection metadata strings, allowing them to be replaced, call NDIlib\_recv\_clear\_connection\_metadata.

An example that illustrates how you can provide your product name to anyone who ever connects to you is provided below.

// Provide a metadata registration that allows people to know what we are.

NDIlib\_metadata\_frame\_t NDI\_product\_type;

NDI\_product\_type.p\_data = "\<ndi\_product long\_name=\\"NDILib Receive Example.\\" "

" short\_name=\\"NDILib Receive\\" "

" manufacturer=\\"CoolCo, inc.\\" "

" version=\\"1.000.000\\" "

" model\_name=\\"PBX-42Q\\" "

" session\_name=\\"My Midday Show\\" "

" serial=\\"ABCDEFG\\" />";

NDIlib\_recv\_add\_connection\_metadata(pRecv, \&NDI\_product\_type);

Note: When using the Advanced SDK, it is possible to assign custom memory allocators for receiving that will allow you to provide user-controlled buffers that are decompressed into. In some cases, this might improve performance or allow you to receive frames into GPU accessible buffers.

### Receiver User interfaces <a href="#_toc131521306" id="_toc131521306"></a>

A sender might provide an interface that allows configuration. For instance, an NDI-converter device might offer an interface that allows its settings to be changed; or a PTZ camera might provide an interface that provides access to specific setting and mode values. These interfaces are provided via a web URL that you can host.

For example, a converter device might have an Advanced SDK web page that is served at a URL such as [http://192.168.1.156/control/index.html](http://192.168.1.156/control/index.html). In order to get this address, you simply call the function:

const char\* NDIlib\_recv\_get\_web\_control(NDIlib\_recv\_instance\_t p\_instance);

This will return a string representing the URL, or NULL if there is no current URL associated with the sender in question. Because connections might take a few seconds, this string might not be available immediately after having called connect. To avoid the need to poll this setting, note that NDIlib\_recv\_capture\_v3 returns a value of NDIlib\_frame\_type\_status\_change when this setting is known (or when it has changed).

The string returned is owned by your application until you call NDIlib\_recv\_free\_string. An example to recover this is illustrated below:

const char\* p\_url = NDIlib\_recv\_get\_web\_control(pRecv);

if (p\_url)

{

// You now have a URL that you can embed in your user interface if you want!

// Do what you want with it here and when done, call:

NDIlib\_recv\_free\_string(pRecv, p\_url);

}

else

{

// This device does not currently support a configuration user interface.

}

You can then store this URL and provide it to an end user as the options for that device. For instance, a PTZ camera or an NDI conversion box might allow its settings to be configured using a hosted web interface.

For sources indicating they support the ability to be configured, NDI Studio Monitor application includes this capability as shown in the bottom-right corner of the image below.

![](<../../../.gitbook/assets/0 (1).png>)

When you click this gear gadget, the application opens the web page specified by the sender.

### Receiver PTZ Control <a href="#_toc131521307" id="_toc131521307"></a>

![](../../../.gitbook/assets/1.png)NDI standardizes the control of PTZ cameras. An NDI receiver will automatically sense whether the device it is connected to is a PTZ camera, and whether it may be controlled automatically.

When controlling a camera via NDI, all configuration of the camera is completely transparent to the NDI client, which will respond to a uniform set of standard commands with well-defined parameter ranges. For instance, NDI Studio Monitor application uses these commands to display on-screen PTZ controls when the current source is reported to be a camera that supports control.

To determine whether the connection that you are on would respond to PTZ messages, you may simply ask the receiver whether this property is supported using the following call:

bool NDIlib\_recv\_ptz\_is\_supported(NDIlib\_recv\_instance\_t p\_instance);

This will return true when the video source is a PTZ system, and false otherwise. Note that connections are not instantaneous, so you might need to wait a few seconds after connection before the source indicates that it supports PTZ control. To avoid the need to poll this setting, note that NDIlib\_recv\_capture\_v3 returns a value of NDIlib\_frame\_type\_status\_change when this setting is known (or when it has changed).

#### PTZ Control <a href="#_toc131521308" id="_toc131521308"></a>

There are standard API functions to execute the standard set of PTZ commands. This list is not designed to be exhaustive and may be expanded in the future.

It is generally recommended that PTZ cameras provide a web interface to give access to the full set of capabilities of the camera and that the host application control the basic messages listed below.

**Zoom Level**

bool NDIlib\_recv\_ptz\_zoom(NDIlib\_recv\_instance\_t p\_instance, const float zoom\_value);

Set the camera zoom level. The zoom value ranges from 0.0 to 1.0.

**Zoom Speed**

bool NDIlib\_recv\_ptz\_zoom\_speed(NDIlib\_recv\_instance\_t p\_instance, const float zoom\_speed);

Control the zoom level as a speed value. The zoom speed value is in the range \[-1.0, +1.0] with zero indicating no motion.

**Pan and Tilt**

bool NDIlib\_recv\_ptz\_pan\_tilt\_speed(NDIlib\_recv\_instance\_t p\_instance, const float pan\_speed, const float tilt\_speed);

This will tell the camera to move with a specific speed toward a direction. The speed is specified in a range \[-1.0, 1.0], with 0.0 meaning no motion.

bool NDIlib\_recv\_ptz\_pan\_tilt(NDIlib\_recv\_instance\_t p\_instance, const float pan\_value,

const float tilt\_value);

This will set the absolute values for pan and tilt. The range of these values is \[-1.0, +1.0], with 0.0 representing center.

**Presets**

bool NDIlib\_recv\_ptz\_store\_preset(NDIlib\_recv\_instance\_t p\_instance, const int preset\_no);

Store the current camera position as a preset. The preset number is in the range 0 to 99.

bool NDIlib\_recv\_ptz\_recall\_preset(NDIlib\_recv\_instance\_t p\_instance,

const int preset\_no, const float speed);

Recall a PTZ preset. The preset number is in the range 0 to 99. The speed value is in the range 0.0 to 1.0, and controls how fast it will move to the preset.

**Focus**

Focus on cameras can either be in auto-focus mode or in manual focus mode. The following are examples of these commands:

bool NDIlib\_recv\_ptz\_auto\_focus(NDIlib\_recv\_instance\_t p\_instance);

bool NDIlib\_recv\_ptz\_focus(NDIlib\_recv\_instance\_t p\_instance, const float focus\_value);

If the mode is auto, then there are no other settings. If the mode is manual, then the value is the focus distance, specified in the range 0.0 to 1.0.

If you wish to control the focus by speed instead of absolute value, you may do this as follows:

bool NDIlib\_recv\_ptz\_focus\_speed(NDIlib\_recv\_instance\_t p\_instance,

const float focus\_speed);

The focus speed is in the range -1.0 to +1.0, with 0.0 indicating no change in focus value.

**White Balance**

White balance can be in a variety of modes, including the following:

bool NDIlib\_recv\_ptz\_white\_balance\_auto(NDIlib\_recv\_instance\_t p\_instance);

This will place the camera in auto-white balance mode.

bool NDIlib\_recv\_ptz\_white\_balance\_indoor(NDIlib\_recv\_instance\_t p\_instance);

This will place the camera in auto-white balance mode, but with a preference for indoor settings.

bool NDIlib\_recv\_ptz\_white\_balance\_outdoor(NDIlib\_recv\_instance\_t p\_instance);

This will place the camera in auto-white balance mode, but with a preference for outdoor settings.

bool NDIlib\_recv\_ptz\_white\_balance\_manual(NDIlib\_recv\_instance\_t p\_instance,

const float red, const float blue);

This allows for manual white-balancing, with the red and blue values in the range 0.0 to 1.0.

bool NDIlib\_recv\_ptz\_white\_balance\_oneshot(NDIlib\_recv\_instance\_t p\_instance);

This allows you to setup the white-balance automatically using the current center of the camera position. It will then store that value as the white-balance setting.

**Exposure Control**

Exposure can either be automatic or manual.

bool NDIlib\_recv\_ptz\_exposure\_auto(NDIlib\_recv\_instance\_t p\_instance);

This will place the camera in auto exposure mode.

bool NDIlib\_recv\_ptz\_exposure\_manual\_v2(NDIlib\_recv\_instance\_t p\_instance,

const float iris,

const float gain,

const float shutter\_speed);

This will place the camera in manual exposure mode with values in the range \[0.0, 1.0].

### Receivers and Tally Messages <a href="#_toc131521309" id="_toc131521309"></a>

Video receivers can specify whether the source is visible on a video switcher’s program or preview row. This is communicated up-stream to the source’s sender, which then indicates its state (see the section on the sender SDK within this document). The sender takes its state and echoes it to all receivers as a metadata message of the form:

\<ndi\_tally\_echo on\_program="true" on\_preview="false"/>

This message is very useful, allowing every receiver to ‘know’ whether its source is on program output.

To illustrate, consider a sender named “My Source A” sending to two destinations, “Switcher” and “Multi-viewer”. When “Switcher” places “My Source A” on program out, a tally message is sent from “Switcher” to “My Source A”. Thus, the source ‘knows’ it is visible on program output. At this point, it will echo its tally state to “Multi-viewer” (and “Switcher”), so that the receiver is aware that “My Source A” is on program out. This functionality is used in the NDI tools Studio Monitor application to display an indicator when the source monitored has its tally state set.

![](../../../.gitbook/assets/2.png)

### Frame Synchronization <a href="#_toc131521310" id="_toc131521310"></a>

When using video, it is important to realize that different clocks are often used by different parts of the signal chain.

Within NDI, the sender can send at the clock rate it wants, and the receiver will receive it at that rate. In many cases, however, the sender and receiver are extremely unlikely to share the _exact same_ clock rate. Bear in mind that computer clocks rely on crystals which – while notionally rated for the same frequency – are seldom truly identical. For example, your sending computer might have an audio clock rated to operate at 48000 Hz. However, it might well actually run at 48001 Hz, or perhaps 47998 Hz.

Similar variances also affect receivers. While the differences appear miniscule, they can accumulate to cause significant audio sync drift over time. A receiver may receive more samples than it plays back; or audible glitches can occur because too few audio samples are sent in a given timespan. Naturally, the same problem affects video sources.

It is very common to address these timing discrepancies by having a "frame buffer", and displaying the most recently received video frame. Unfortunately, the deviations in clock-timing prevent this from being a perfect solution. Frequently, for example, video will appear to ‘jitter’ when the sending and receiving clocks are _almost_ aligned (which is actually the most common case).

A "time base corrector" (TBC) or frame-synchronizer for the video clock provides another mechanism to handle these issues. This approach uses hysteresis to determine the best time to either drop or insert a video frame to achieve smooth video playback (audio should be dynamically sampled with a high order resampling filter to adaptively track clocking differences).

It’s quite difficult to develop something that is correct for all of the diverse scenarios that can arise, so the NDI SDK provides an implementation to help you develop real time audio/video applications without assuming responsibility for the significant complexity involved.

Another way to view what this component of the SDK does is to think of it as transforming ‘push’ sources (i.e., NDI sources in which the data is pushed from the sender to the receiver) into ‘pull’ sources, wherein the host application pulls the data down-stream. The frame-sync automatically tracks all clocks to achieve the best video and audio performance while doing so.

In addition to time-base correction operations, NDI’s frame sync will also automatically detect and correct for timing jitter that might occur. This internally handles timing anomalies such as those caused by network, sender or receiver side timing errors related to CPU limitations, network bandwidth fluctuations, etc.

A very common application of the frame-synchronizer is to display video on screen timed to the GPU v-sync, in which case you should convert the incoming time-base to the time-base of the GPU. The following table lists some are common scenarios in which you might want to use frame-synchronization:

| Scenario                                  | Recommendation                                                                                                                                                                                                                                                                   |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Video playback on a screen or multiviewer | Yes – you want the clock to be synced with vertical refresh. On a multiviewer you would have a frame-sync for every video source, then call all of them on each v-sync and redraw all sources at that time.                                                                      |
| Audio playback through sound card         | Yes – the clock should be synced with your sound card clock.                                                                                                                                                                                                                     |
| Video mixing of sources                   | Yes – all video input clocks need to be synced to your output video clock. You can take each of the video inputs and frame-synchronize them together.                                                                                                                            |
| Audio mixing                              | Yes – you want all input audio clocks to be brought into sync with your output audio clock. You would create a frame-synchronizer for each audio source and – when driving the output – call each one, asking for the correct number of samples and sample-rate for your output. |
| Recording a single channel                | No – you should record the signal in the raw form without any re-clocking.                                                                                                                                                                                                       |
| Recording multiple channels               | Maybe – If you want to sync some input channels to match a master clock so that they can be ISO-edited, you might want a frame sync for all sources _except one_ (allowing them all to be synchronized with a single channel).                                                   |

To create a frame synchronizer object, you will call the function below (that is based an already instantiated NDI receiver from which it will get frames). Once this receiver has been bound to a frame-sync, you should use it in order to recover video frames.

You can continue to use the underlying receiver for other operations, such as tally, PTZ, metadata, etc. Remember, it remains your responsibility to destroy the receiver – even when a frame-sync is using it (you should always destroy the receiver _after_ the framesync has been destroyed).

NDIlib\_framesync\_instance\_t NDIlib\_framesync\_create(NDIlib\_recv\_instance\_t p\_receiver);

The frame-sync is destroyed with the corresponding call:

void NDIlib\_framesync\_destroy(NDIlib\_framesync\_instance\_t p\_instance);

In order to recover audio, the following function will pull audio samples from the frame-sync queue. This function will always return data immediately, inserting silence if no current audio data is present. You should call this at the rate that you want audio, and it will automatically use dynamic audio sampling to conform the incoming audio signal to the rate at which you are calling.

Note that you have no obligation to ensure that your requested sample rate, channel count and number of samples match the incoming signal, and all combinations of conversions are supported.

Audio resampling is done with high order audio filters. Timecode and per frame metadata are inserted into the best possible audio samples.

Also, if you specify the desired sample-rate as zero it will fill in the buffer (and audio data descriptor) with the original audio sample rate. And if you specify the channel count as zero, it will fill in the buffer (and audio data descriptor) with the original audio channel count.

void NDIlib\_framesync\_capture\_audio(

NDIlib\_framesync\_instance\_t p\_instance, // The frame sync instance

NDIlib\_audio\_frame\_v2\_t\* p\_audio\_data, // The destination audio buffer

int sample\_rate, // Your desired sample rate. 0 for “use source”.

int no\_channels, // Your desired channel count. 0 for “use source”.

int no\_samples); // The number of audio samples that you wish to get.

The buffer returned is freed using the corresponding function:

void NDIlib\_framesync\_free\_audio(NDIlib\_framesync\_instance\_t p\_instance,

NDIlib\_audio\_frame\_v2\_t\* p\_audio\_data);

This function will pull video samples from the frame-sync queue. It will always immediately return a video sample by using time-base correction. You can specify the desired field type, which is then used to return the best possible frame.

Note that:

* Field-based frame sync means that the frame synchronizer attempts to match the fielded input phase with the frame requests to provide the most correct possible field ordering on output.
* The same frame can be returned multiple times if duplication is needed to match the timing criteria.

It is assumed that progressive video sources can i) correctly display either a field 0 or field 1, ii) that fielded sources can correctly display progressive sources, and iii) that the display of field 1 on a field 0 (or vice versa) should be avoided at all costs.

If no video frame has ever been received, this will return NDIlib\_video\_frame\_v2\_t as an empty (all zero) structure. This allows you to determine that there has not yet been any video, and act accordingly (for instance you might want to display a constant frame output at a particular video format, or black).

void NDIlib\_framesync\_capture\_video(

NDIlib\_framesync\_instance\_t p\_instance, // The frame-sync instance

NDIlib\_video\_frame\_v2\_t\* p\_video\_data, // The destination video frame

NDIlib\_frame\_format\_type\_e field\_type); // The frame type that you prefer

The buffer returned is freed using the corresponding function:

void NDIlib\_framesync\_free\_video(NDIlib\_framesync\_instance\_t p\_instance,

NDIlib\_video\_frame\_v2\_t\* p\_video\_data);
