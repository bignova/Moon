---
layout: post
title: "实时数据可视化平台"
date: 2018-06-13
excerpt: "Real-time Data Visualization Solutions"
tags: [Tech, Analysis, 技术, 分析]
comments: true
---

实时可视化框架 - <a href="https://bignova.github.io/viztool/">Viztool</a>, 是我在本科的Software Engineering课上的小组project。一个学期下来，与之相关的研究也做了不少。

## 概念

在这篇文章里，我们将数据可视化方案分为“后端实现”与“前端实现”两种。

后端可视化通常将采集到的数据通过脚本或者人工的方式，使用python，R，或matlab的数据可视化库生成图表，然后再通过server向用户终端发送图像。

前端数据可视化方案将可视化的最终实现放在前端（或用户终端，即web app，mobile app）完成。在web的框架之中，用户终端通常为浏览器。而Javascript（JS）作为浏览器可执行语言，由其开发的相关数据可视化工具非常适合在用户终端上进行图表绘制。

## 前端实现相比于后端实现有哪些好处？

* 由于前端为用户终端，因此前端可视化更容易实现**实时**数据可视化的效果。
* 在前端实现的方案中，数据仅在用户终端被转换为视图，因此节省了之前阶段的传输流量。
* 最为重要的一点，前端通过JS实现数据可视化，更容易调整视图风格，使其与网站整体设计风格相适应。

## 数据可视化流程

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/41342508-fc9dafce-6f2e-11e8-8d20-cd7723673719.png" width="610px" height="193px">
</figure>

前端实时数据可视化通常被分为以上三个阶段。而目前市面上常见的数据可视化解决方案正是将对于三个阶段相应的服务整合而成。

在第一阶段“数据源”中，相关服务一般根据具体的数据源使用平台，提供不同的SDK，如下图所示。

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/41342656-57b7eed8-6f2f-11e8-8381-3e5239955c8e.png" width="358px" height="230px">
</figure>

第二阶段“数据传送”则通常由“云平台”实现，并提供相关的数据存储，预处理服务。

第三阶段“前端”则如上文所说，通过实现一些常见的JS可视化库完成最终的数据可视化工作。

而各阶段之间数据的发送接收则依赖不同的标准通信协议，例如MQTT（基于TCP/IP，物联网为其常见使用场景）。具体协议视相关服务所支持的标准而定。

## 常见前端实时数据可视化方案
