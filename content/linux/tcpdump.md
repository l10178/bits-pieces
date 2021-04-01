---
title: "tcpdump"
date: 2021-01-29T23:54:37+08:00
draft: false
---

tcpdump 捕捉网卡eth0流量，并存入到out.pcap文件中。

```sh
sudo tcpdump -vv -s0 -i eth0 -w out.pcap
```
