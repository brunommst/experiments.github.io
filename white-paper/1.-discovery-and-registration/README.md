---
cover: ../../.gitbook/assets/ndi sdk background light (1).png
coverY: 0
layout:
  cover:
    visible: true
    size: hero
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

# 1. Discovery and Registration

### Zero configuration in AV signal distribution <a href="#_toc144291436" id="_toc144291436"></a>

One of the biggest issues in AV distribution in the IP world is that equipment is not identifiable by its physical connection. In networking, every connected device needs to have a unique address so another device, hardware, and applications can reach it.&#x20;

However, the network physical connection is dynamic and not related at all to the equipment address. For that reason, in a large network with hundreds (or thousands) of devices with addresses, it becomes difficult to find and interconnect equipment. NDI offers two different options for a zero-configuration discovery and registration: **mDNS** and **Discovery Service**.
