# Libraries

Components included in the SDK provide support for finding (find), receiving (recv), and sending (send). These share common structures and conventions to facilitate development and may all be used together.

#### NDI-Send <a href="#_toc131521269" id="_toc131521269"></a>

This is used to send video, audio, and metadata over the network. You establish yourself as a named source on the network, and then anyone may see and use the media that you are providing.

Video can be sent at any resolution and framerate in RGB(+A) and YCbCr color spaces, and any number of receivers can connect to an individual NDI-Send.

{% content-ref url="ndi-send.md" %}
[ndi-send.md](ndi-send.md)
{% endcontent-ref %}

#### NDI-Find <a href="#_toc131521270" id="_toc131521270"></a>

This is used to locate all of the sources on the local network that are serving media capabilities for use with NDI.

{% content-ref url="ndi-find.md" %}
[ndi-find.md](ndi-find.md)
{% endcontent-ref %}

#### NDI-Receive <a href="#_toc131521271" id="_toc131521271"></a>

This allows you to take sources on the network and receive them. The SDK internally includes all the requisite codecs and handles all the complexities of reliably receiving high-performance network video.

{% content-ref url="ndi-recv.md" %}
[ndi-recv.md](ndi-recv.md)
{% endcontent-ref %}
