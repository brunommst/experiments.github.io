# NDI-Send

### NDI-Send

A call to NDIlib\_send\_create will create an instance of the sender. This will return an instance of type NDIlib\_send\_instance\_t (or NULL if it fails) representing the sending instance.

The set of creation parameters applied to the sender are specified by filling out a structure called NDIlib\_send\_create\_t. It is now possible to call NDIlib\_send\_create with a NULL parameter, in which case it will use default parameters for all values; the source name is selected using the current executable name, ensuring that there is a count that ensures sender names are unique (e.g. “My Application”, “My Application 2”, etc.)

| Supported Parameters              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| p\_ndi\_name (const char\*)       | <p>This is the name of the NDI source to create. It is a NULL-terminated UTF-8 string. This will be the name of the NDI source on the network.</p><p>For instance, if your network machine name is called “MyMachine” and you specify this parameter as “My Video”, the NDI source on the network would be “MyMachine (My Video)”.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| p\_groups (const char\*)          | <p>This parameter represents the groups that this NDI sender should place itself into. Groups are sets of NDI sources. Any source can be part of any number of groups, and groups are comma-separated. For instance "cameras,studio 1,10am show" would place a source in the three groups named.</p><p>On the finding side, you can specify which groups to look for and look in multiple groups. If the group is NULL then the system default groups will be used.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| clock\_video, clock\_audio (bool) | <p>These specify whether audio and video "clock" themselves. When they are clocked, video frames added will be rate-limited to match the current framerate they are submitted at. The same is true for audio.</p><p><br>In general, if you are submitting video and audio off a single thread, you should only clock one of them (video is probably the better choice to clock off). If you are submitting audio and video of separate threads, then having both clocked can be useful.</p><p>A simplified view of the how works is that, when you submit a frame, it will keep track of the time the next frame would be required at. If you submit a frame before this time, the call will wait until that time. This ensures that, if you sit in a tight loop and render frames as fast as you can go, they will be clocked at the framerate that you desire.</p><p>Note that combining clocked video and audio submission combined with asynchronous frame submission (see below) allows you to write very simple loops to render and submit NDI frames.</p> |

An example of creating an NDI sending instance is provided below.

NDIlib\_send\_create\_t send\_create;

send\_create.p\_ndi\_name = "My Video";

send\_create.p\_groups = NULL;

send\_create.clock\_video = true;

send\_create.clock\_audio = true;

NDIlib\_send\_instance\_t pSend = NDIlib\_send\_create(\&send\_create);

if (!pSend)

printf("Error creating NDI sender");

Once you have created a device, any NDI finders on the network will be able to see this source as available. You may now send audio, video, or metadata frames to the device – at any time, off any thread, and in any order.

There are no reasonable restrictions on video, audio or metadata frames that can be sent or received. In general, video frames yield better compression ratios as resolution increases (although the size does increase). Note that all formats can be changed frame-to-frame.

The specific structures used to describe the different frame types are described under the section “Frame types” below. An important factor to understand is that video frames are “buffered” on an input; if you provide a video frame to the SDK when there are no current connections to it, the last video frame will automatically be sent when a new incoming connection is received. This is done without any need to recompress a frame (it is buffered in memory in compressed form).

The following represents an example of how one might send a single 1080i59.94 white frame over an NDI sending connection.

// Allocate a video frame (you would do something smarter than this!)

uint8\_t\* p\_frame = (uint8\_t\*)malloc(1920\*1080\*4);

memset(p\_frame, 255, 1920\*1080\*4);

// Now send it!

NDIlib\_video\_frame\_v2\_t video\_frame;

video\_frame.xres = 1920;

video\_frame.yres = 1080;

video\_frame.FourCC = NDIlib\_FourCC\_type\_BGRA;

video\_frame.frame\_rate\_N = 30000;

video\_frame.frame\_rate\_D = 1001;

video\_frame.picture\_aspect\_ratio = 16.0f/9.0f;

video\_frame.frame\_format\_type = NDIlib\_frame\_format\_type\_progressive;

video\_frame.timecode = 0;

video\_frame.p\_data = p\_frame;

video\_frame.line\_stride\_in\_bytes = 1920\*4;

video\_frame.p\_metadata = "\<Hello/>";

// Submit the buffer

NDIlib\_send\_send\_video\_v2(pSend, \&video\_frame);

// Free video memory

free(p\_frame);

// In a similar fashion, audio can be submitted for NDI audio sending,

// the following will submit 1920 quad-channel silent audio samples

// at 48 kHz

// Allocate an audio frame (you would do something smarter than this!)

float\* p\_frame = (float\*)malloc(sizeof(float)\*1920\*4)

memset(p\_frame, 0, sizeof(float)\*1920\*4);

// describe the buffer

NDIlib\_audio\_frame\_v3\_t audio\_frame;

audio\_frame.sample\_rate = 48000;

audio\_frame.no\_channels = 4;

audio\_frame.no\_samples = 1920;

audio\_frame.timecode = 0;

audio\_frame.p\_data = p\_frame;

audio\_frame.channel\_stride\_in\_bytes = sizeof(float)\*1920;

audio\_frame.p\_metadata = NULL; // No metadata on this example!

// Submit the buffer

NDIlib\_send\_send\_audio\_v3(pSend, \&audio\_frame);

// Free the audio memory

free(p\_frame);

Because many applications provide interleaved 16-bit audio, the NDI library includes utility functions to convert PCM 16-bit formats to and from floating-point formats.

Alternatively, there is a utility function (NDIlib\_util\_send\_send\_audio\_interleaved\_16s) for sending signed 16-bit audio. (Please refer to the example projects and also the header file Processing.NDI.utilities.h , which lists the functions available.) In general, we recommend using floating-point audio, since clamping is not possible and audio levels are well defined without a need to consider audio headroom.

Metadata is submitted in a very similar fashion. (We do not provide a code example as this is easily understood by referring to the audio and video examples.)

To receive metadata being sent from the receiving end of a connection (e.g., which can be used to select pages, change settings, etc.) we refer you to the way the receive device works.

The basic process involves calling NDIlib\_send\_capture with a time-out value. This can be used either to query whether a metadata message is available if the time-out is zero, or to efficiently wait for messages on a thread. The basic process is outlined below:

// Wait for 1 second to see if there is a metadata message available

NDIlib\_metadata\_frame\_t metadata;

if (NDIlib\_send\_capture(pSend, \&metadata, 1000) == NDIlib\_frame\_type\_metadata)

{

// Do something with the metadata here

// ...

// Free the metadata message

NDIlib\_recv\_free\_metadata(pSend, \&meta\_data);

}

Connection metadata, as specified in the NDI-Recv section of this documentation, is an important category of metadata that you will receive automatically as new connections to you are established. This allows an NDI receiver to provide up-stream details to a sender and can include hints as to the capabilities the receiver might offer. Examples include the resolution and framerate preferred by the receiver, its product name, etc. It is important that a sender is aware that it might be sending video data to more than one receiver at a time, and in consequence will receive connection metadata from each one of them.

Determining whether you are on program and/or preview output on a device such as a video mixer (i.e., ‘Tally’ information) is very similar to how metadata information is handled. You can ‘query’ it, or you can efficiently ‘wait’ and get tally notification changes. The following example will wait for one second and react to tally notifications:

// Wait for 1 second to see if there is a tally change notification.

NDIlib\_tally\_t tally\_data;

if (NDIlib\_send\_get\_tally(pSend, \&tally\_data)

{

// The tally state changed and you can now

// read the new state from tally\_data.

}

An NDI send instance is destroyed by passing it into NDIlib\_send\_destroy.

Connection metadata is data that you can “register” with a sender; it will automatically be sent each time a new connection with the sender is established. The sender internally maintains a copy of any connection metadata messages and sends them automatically.

This is useful to allow a sender to provide downstream information whenever any device might want to connect to it (for instance, letting it know what the product name or preferred video format might be). Neither senders nor receivers are required to provide this functionality and may freely ignore any connection data strings.

Standard connection metadata strings are defined in a later section of this document. To add a metadata element, one can call NDIlib\_send\_add\_connection\_metadata. To clear all the registered elements, one can call NDIlib\_send\_clear\_connection\_metadata. An example that registers the name and details of your sender so that other sources that connect to you get information about what you are is provided below.

// Provide a metadata registration that allows people to know what we are.

NDIlib\_metadata\_frame\_t NDI\_product\_type;

NDI\_product\_type.p\_data = "\<ndi\_product long\_name=\\"NDILib Send Example.\\" "

" short\_name=\\"NDILib Send\\" "

" manufacturer=\\"CoolCo, inc.\\" "

" model\_name=\\"PBX-15M\\" "

" version=\\"1.000.000\\" "

" serial=\\"ABCDEFG\\" "

" session\_name=\\"My Midday Show\\" />";

NDIlib\_send\_add\_connection\_metadata(pSend, \&NDI\_product\_type);

Because NDI assumes that all senders must have a unique name and applies certain filtering to NDI names to make sure that they are network name-space compliant, at times the name of a source you created may be modified slightly. To assist you in getting the exact name of any sender (to ensure you use the same one) there is a function to receive this name.

const NDIlib\_source\_t\* NDIlib\_send\_get\_source\_name(NDIlib\_send\_instance\_t p\_instance);

The lifetime of the returned value is until the sender instance is destroyed.

### Asynchronous Sending <a href="#_toc131521299" id="_toc131521299"></a>

It is possible to send video frames asynchronously using NDI using the call NDIlib\_send\_send\_video\_v2\_async. This function will return immediately and will perform all required operations (including color conversion, any compression and network transmission) asynchronously with the call.

Because NDI takes full advantage of asynchronous OS behavior when available, this will normally result in improved performance (as compared to creating your own thread and submitting frames asynchronously with rendering).

The memory that you passed to the API through the NDIlib\_video\_frame\_v2\_t pointer will continue to be used until a synchronizing API call is made. Synchronizing calls are any of the following:

* Another call to NDIlib\_send\_send\_video\_v2\_async_._
* A call to NDIlib\_send\_send\_video\_v2\_async(pSend, NULL) will wait for any asynchronously scheduled frames to completed and then return. Obviously, you can also submit the next frame, whereupon it will wait for the previous frame to finish before asynchronously submitting the current one.
* Another call to NDIlib\_send\_send\_video\_v2.
* A call to NDIlib\_send\_destroy.

Using this in conjunction with a clocked video output results in a very efficient rendering loop where you do not need to use separate threads for timing or for frame submission. For example, the following is an efficient real-time processing system as long as rendering can always keep up with real-time:

while(!done())\
{

render\_frame();

NDIlib\_send\_send\_video\_v2\_async(pSend, \&frame\_data);

}

NDIlib\_send\_send\_video\_v2\_async(pSend, NULL); // Sync here

Note: User error involving asynchronous sending is most common SDK ‘bug report’. It is very important to understand that a call to NDIlib\_send\_send\_video\_v2\_async starts processing, then sending the video frame asynchronously with the calling application. If you call this and then free the pointer, your application will most likely crash in an NDI thread – because the SDK is still using the video frame that was passed to the call.

If you re-use the buffer immediately after calling this function, your video stream will likely exhibit tearing or other glitches, since you are writing to the buffer while the SDK is still compressing data it previously held. One possible solution is to “ping pong” between two buffers on alternating calls to NDIlib\_send\_send\_video\_v2\_async, and then call that same function with a NULL frame pointer before releasing these buffers at the end of your application. When working in this way you would generally render, compress, and send to the network, with each process being asynchronous to the others.

Note: If you are using the Advanced SDK, it is possible to assign a completion handler for asynchronous frame sending that more explicitly allows you to track buffer ownership with asynchronous sending.

### Timecode synthesis <a href="#_toc131521300" id="_toc131521300"></a>

It is possible to specify your own timecode for all data sent when sending video, audio, or metadata frames. You may also specify a value of NDIlib\_send\_timecode\_synthesize (defined as INT64\_MAX) to cause the SDK to generate timecode for you. When you specify this, the timecode is synthesized as UTC time since the Unix Epoch (1/1/1970 00:00) with 100 ns precision.

If you never specify a timecode at all (and instead ask for each to be synthesized) the current system clock time is used as the starting timecode (translated to UTC since the Unix Epoch), and synthetic values are generated. This keeping your streams exactly in sync, as long as the frames you are sending do does not deviate from the system time in any meaningful way. In practice this means that, if you never specify timecodes, they will always be generated correctly for you. Timecodes from different senders on the same machine will always be in sync with each other when working in this way. And if you have NTP installed on your local network, streams can be synchronized between multiple machines with very high precision.

If you specify a timecode at a particular frame (audio or video), then ask for all subsequent ones to be synthesized, the subsequent ones generated will continue this sequence. This maintains the correct relationship between the streams and samples generated, avoiding any meaningful deviation from the timecode you specified over time.

If you specify timecodes on one stream (e.g., video) and ask for the other stream (audio) to be synthesized, the timecodes generated for the other stream exactly match the correct sample positions; they are not quantized inter-stream. This ensures that you can specify just the timecodes on a single stream and have the system generate the others for you.

When you send metadata messages and ask for the timecode to be synthesized, it is chosen to match the closest audio or video frame timecode (so that it looks close to something you might want). If no sample looks sufficiently close, a timecode is synthesized from the last ones known and the time that has elapsed since it was sent.

Note that the algorithm to generate timecodes synthetically will correctly assign timestamps if frames are not submitted at the exact time.

For instance, if you submit a video frame and then an audio frame in sequential order, they will both have the same timecode even though the video frame may have taken a few milliseconds longer to encode.

That said, no per-frame error is ever accumulated. So – if you are submitting audio and video and they do not align over a period of more than a few frames – the timecodes will still be correctly synthesized without accumulated error.

### Failsafe <a href="#_toc131521301" id="_toc131521301"></a>

Failsafe is a capability of any NDI sender. If you specify a failsafe source on an NDI sender and the sender fails for any reason (even the machine failing completely), any receivers viewing that sender will automatically switch over to the failsafe sender. If the failed source comes back online, receivers will switch back to that source.

You can set the fail-over source on any video input with a call to:

void NDIlib\_send\_set\_failover(NDIlib\_send\_instance\_t p\_instance,\
const NDIlib\_source\_t\* p\_failover\_source);

The failover source can be any network source. If it is specified as NULL the failsafe source will be cleared.

### Capabilities <a href="#_toc131521302" id="_toc131521302"></a>

An NDI capabilities metadata message can be submitted to the NDI sender for communicating certain functionality that the downstream NDI receivers should know about upon connecting. For example, if you are providing PTZ type functionality, letting the NDI receiver know this would done through this type of metadata message. The following is an example of the NDI capabilities message:

\<ndi\_capabilities web\_control="http://ndi.video/" ntk\_ptz="true" ntk\_exposure\_v2="true" />

You would submit this message to the NDI sender for communication to current and future NDI receivers as follows:

NDIlib\_metadata\_frame\_t NDI\_capabilities;

NDI\_capabilities.p\_data = "\<ndi\_capabilities web\_control=\\"http://ndi.video/\\" "

" ntk\_ptz=\\"true\\" "

" ntk\_exposure\_v2=\\"true\\" />";

NDIlib\_send\_add\_connection\_metadata(pNDI\_send, \&NDI\_capabilities\_type);

Below is a table of XML attributes that can be used in this capabilities message:

| Supported Attributes |                                                                                                                                                                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| web\_control         | The URL to the local device webpage. If %IP% is present in the value, it will be replaced with the local IP of the NIC in which the NDI receiver is connected to.                                                                     |
| ntk\_ptz             | Signifies that this NDI sender is capable of processing PTZ commands sent from the NDI receiver. The NDI receiver will only assume the NDI sender can support PTZ commands if this attribute is received and set to the value “true”. |
| ntk\_pan\_tilt       | The NDI sender supports pan and tilt control.                                                                                                                                                                                         |
| ntk\_zoom            | The NDI sender supports zoom control.                                                                                                                                                                                                 |
| ntk\_iris            | The NDI sender supports iris control.                                                                                                                                                                                                 |
| ntk\_white\_balance  | The NDI sender supports white balance control.                                                                                                                                                                                        |
| ntk\_exposure        | The NDI sender supports exposure control.                                                                                                                                                                                             |
| ntk\_exposure\_v2    | The NDI sender supports detailed control over exposure such as iris, gain, and shutter speed.                                                                                                                                         |
| ntk\_focus           | The NDI sender supports manual focus control.                                                                                                                                                                                         |
| ntk\_autofocus       | The NDI sender supports setting auto focus.                                                                                                                                                                                           |
| ntk\_preset\_speed   | The NDI sender has preset speed support.                                                                                                                                                                                              |

### Apple iOS Notes <a href="#_toc131521303" id="_toc131521303"></a>

When an iOS app is sent to the background, most of the networking functionality is put into a suspended state. Sometimes resources associated with networking are released back to the operating system while in this state.

Apple recommends closing certain networking operations when the app is placed in the background, then restarted when put in the foreground again. Because of this, we recommend releasing an NDI sender instance within the app’s applicationDidEnterBackground method, then recreating the instance in the applicationDidBecomeActive method.
