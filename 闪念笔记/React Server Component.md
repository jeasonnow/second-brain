---
aliases: 
date created: 2023-05-11
date modified: 2023-05-11
tags:
  - React
title: React Server Component
---

## 闪念

`React Server Component` 的 `RFC` 相当于是提供了一种让页面渲染可以离散到客户端 + 服务端两种端的场景，通过将部分重依赖低交互的组件作为 `Server Component` 放到服务端完成渲染并通过特定协议流式传输到客户端和客户端混合渲染成完整页面从而减少客户端和服务端的通信，以及部分巨大依赖的加载，从而提升性能。

## 来源

[初探 React Server Components - 知乎](https://zhuanlan.zhihu.com/p/352848874)
