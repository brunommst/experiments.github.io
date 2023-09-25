---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Manual connection

One approach to manually interconnect NDI devices is to specify the IP address of the transmitter in the receiver.

![Screenshot of NDI Access Manager tool](<../../.gitbook/assets/2 (1).png>)

In Windows and MacOS, this can be achieved using the NDI Access Manager in the External Sources feature. Several NDI hardware decoders also support this functionality.

For Linux, the IP addresses of NDI senders can be added manually in the NDI configuration file called "ndi-config.v1.json." This file is in the home directory of the user currently logged in.

Specifying the IP address of an NDI source allows the receiver to receive NDI sources that are in a different subnet and may not be discoverable by mDNS (Multicast DNS). This method enables the reception of NDI sources that might be otherwise inaccessible due to network configurations or limitations.

In Linux manual connections can be added in the NDI configuration file located in the home directory of the effective user: "ndi-config.v1.json"

Here is the way to manually set up NDI sources in the configuration file:

"networks": {\
"ips": "192.168.123.200,10.10.123.22,",\
"discovery": "",
