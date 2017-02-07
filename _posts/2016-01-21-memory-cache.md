---
layout: post
title: "Memory Cache"
category: [NOTE, TODO]
tags: [Cache]
---
* TOC
{:toc}
# Memory Cache
> 此处简述内存缓存
> 传统的服务，每次请求都访问DB，但是访问DB的成本很高，而且连接有限。在千万级的PV中，根本无法支撑

## Less database access
> 尽量避免DB的访问，磁盘、连接

### Local cache
> 第一阶段可以将热点数据缓存在本地cache中
