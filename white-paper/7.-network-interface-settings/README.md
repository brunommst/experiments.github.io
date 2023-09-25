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
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 7. Network Interface Settings

NDI is designed to enable successful video transport using the default configuration of network interface drivers; however, most recent network interface drivers do support the configuration of advanced properties that can help optimize NDI transmission.

Consider the following adjustments but note that adjusting individual adapters can significantly affect performance and reliability positively and negatively. It is important to consider testing performance with a network analyzer before and after each setting change. The following adjustments are intended to help; however, performance will depend on network and usage (names and available settings vary between vendors, adapter models, and even between different driver versions):
