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

# mDNS

Sending and receiving video streams across an IP network requires applications that support video and can discover receiving applications that are looking for video.

NDI resolves host names to IP addresses over the LAN and does so automatically. When you start an application that sends NDI, the devices that can receive NDI become aware instantaneously. While this is a typical function on almost all networks, there are some cases where it is important to know how this works to properly configure networks utilizing managed data flow protocols.

By default, NDI utilizes mDNS (multicast Domain Name System)\[1] to create the zero-configuration environment for discovery. This service sends an IP multicast message that asks the host to identify itself. The target machine then multicasts a message that includes its own IP address. This multicast is seen by all NDI-receiving machines on the subnet, which then use the information in that message to update their own caches.

These multicast queries are sent to a multicast address, and thus, no single device is required to have global knowledge.

When a service or device sees a query for any service it recognizes, it provides a DNS response with the information from its cache. The primary benefit of using mDNS is that it requires little or no administration to set up. Unless the network is specifically configured not to allow mDNS, NDI sources will be discovered. This format works when no infrastructure is present and can span infrastructure failures.

**The mDNS Ethernet frame is a multicast UDP packet that broadcasts to\[2]:**

| MAC Adress   | 01:00:5E:00:00:FB (for IPv4) |
| ------------ | ---------------------------- |
| Ipv4 Address | 224.0.0.251                  |
| UDP Port     | 5353                         |

Choosing the network location type on Windows devices is critical for the successful discovery and registration of NDI. Typically, the first time a Windows machine is connected to a network, a dialog window appears that allows the user to choose the network location type: Private or Public. By default, Windows sets a new network location to Public.

This location is designed to keep machines from being visible and responding to broadcast pings. This location type also affects mDNS responses and keeps NDI video streams from being discovered and registered on the network.

**Network locations should be set to Private for successful discovery and registration of NDI.** The Domain network location is used for domain networks, such as those at enterprise workplaces. The network administrator controls this type of network location, and it cannot be selected or changed. In this type of configuration, mDNS discovery must be allowed at the domain level. Because mDNS uses a link-local multicast address, its capacity is limited to a single physical or logical LAN.
