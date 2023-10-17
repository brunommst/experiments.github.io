# NDI Send

A call to `NDIlib_send_create` will create an instance of the sender. This will return an instance of the type `NDIlib_send_instance_t` (or `NULL` if it fails) representing the sending instance.

The set of creation parameters applied to the sender is specified by filling out a structure called `NDIlib_send_create_t`. It is now possible to call `NDIlib_send_create` with a NULL parameter, in which case it will use default parameters for all values; the source name is selected using the current executable name, ensuring that there is a count that ensures sender names are unique (e.g. “My Application”, “My Application 2”, etc.)

### Parameters

| Supported Parameters                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`p_ndi_name (const char*)`**        | <p>This is the name of the NDI source to create. It is a <code>NULL</code>-terminated UTF-8 string. This will be the name of the NDI source on the network.</p><p></p><p>For instance, if your network machine name is called “MyMachine” and you specify this parameter as “My Video”, the NDI source on the network would be “MyMachine (My Video)”.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **`p_groups (const char*)`**          | <p>This parameter represents the groups that this NDI sender should place itself into. Groups are sets of NDI sources. Any source can be part of any number of groups, and groups are comma-separated. For instance, "cameras,studio 1,10am show" would place a source in the three groups named.</p><p></p><p>On the finding side, you can specify which groups to look for and look in multiple groups. If the group is <code>NULL</code> then the system default groups will be used.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **`clock_video, clock_audio (bool)`** | <p>These specify whether audio and video "clock" themselves. When they are clocked, video frames added will be rate-limited to match the current framerate at which they are submitted. The same is true for audio.</p><p></p><p>In general, if you are submitting video and audio of a single thread, you should only clock one of them (video is probably the better choice to clock off). If you are submitting audio and video of separate threads, then having both clocked can be useful.</p><p></p><p>A simplified view of how it works is that when you submit a frame, it will keep track of the time the next frame would be required. If you submit a frame before this time, the call will wait until that time. This ensures that if you sit in a tight loop and render frames as fast as you can go, they will be clocked at the framerate that you desire.</p><p></p><p>Note that combining clocked video and audio submission with asynchronous frame submission (<em>see below</em>) allows you to write very simple loops to render and submit NDI frames.</p> |

### Examples

**An example of creating an NDI sending instance is provided below**.

```
NDIlib_send_create_t send_create; 
send_create.p_ndi_name = "My Video"; 
send_create.p_groups = NULL; 
send_create.clock_video = true; 
send_create.clock_audio = true;

NDIlib_send_instance_t pSend = NDIlib_send_create(&send_create); 
if (!pSend)
    printf("Error creating NDI sender");
```

Once you have created a device, any NDI finders on the network will be able to see this source as available. You may now send audio, video, or metadata frames to the device – at any time, off any thread, and in any order.

There are no reasonable restrictions on video, audio, or metadata frames that can be sent or received. In general, video frames yield better compression ratios as resolution increases (although the size does increase). Note that all formats can be changed frame-to-frame.

The specific structures used to describe the different frame types are described in the section “Frame types” below. An important factor to understand is that video frames are “buffered” on an input; if you provide a video frame to the SDK when there are no current connections to it, the last video frame will automatically be sent when a new incoming connection is received. This is done without any need to recompress a frame (it is buffered in memory in compressed form).

**The following represents an example of how one might send a single 1080i59.94 white frame over an NDI sending connection.**



