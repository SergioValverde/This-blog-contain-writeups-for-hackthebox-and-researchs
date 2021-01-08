---
title: Data Exfiltration - Mistica
date: 2021-01-08 19:17:00 Z
---

![Mistica.png]({{site.baseurl}}/images/Mistica/Mistica.png)

[https://github.com/IncideDigital/Mistica](https://github.com/IncideDigital/Mistica)


`Mistica is a tool that allows to embed data into application layer protocol fields, with the goal of establishing a bi-directional channel for arbitrary communications. Currently, encapsulation into HTTP, DNS and ICMP protocols has been implemented, but more protocols are expected to be introduced in the near future.


Mística has a modular design, built around a custom transport protocol, called SOTP: Simple Overlay Transport Protocol. Data is encrypted, chunked and put into SOTP packets. SOTP packets are encoded and embedded into the desired field of the application protocol, and sent to the other end.
`