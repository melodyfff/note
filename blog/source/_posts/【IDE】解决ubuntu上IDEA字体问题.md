---
title:【IDE】解决ubuntu上IDEA字体问题
tags: [IDE]
date: 2018-2-18
---

# `IntelliJ IDEA`在`ubuntu16.04`上字体不正常

原本类似于微软雅黑的字体一下子成了宋体，方方正正就是感觉很别扭

这个原因是多安装了两个字体

```bash
sudo apt-get remove fonts-arphic-ukai fonts-arphic-uming
```

