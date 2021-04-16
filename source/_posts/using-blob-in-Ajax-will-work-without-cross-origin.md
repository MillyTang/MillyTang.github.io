---
title: using-blob-in-Ajax-will-work-without-cross-origin
date: 2021-04-15 15:48:12
tags:
---

配合Ajax 使用 new Blob 下载文件不会导致跨域问题：

> Blob URL / Object URL是一种伪协议，可以让Blob和File对象用作图像和二进制数据下载链接等URL源

浏览器在内部通过URL.createObjectURL()创建一个对 Blob 或 File 对象的特殊引用，生成的 Blob URL 只能在浏览器本地的单个实例和同一会话中使用，并且这个 URL 对象会在页面退出的时候被浏览器释放。

因此 Blob URL 并不能指向一个服务器资源，你无法在其它页面中打开它。同时由于编码格式有所差别，Blob URL 比起 Data URLs 所占的空间资源更少，性能也更好。

Blob 为 Web 开发提供了一个非常有用的功能：创建 Blob URL。将二进制数据封装为 Blob 对象，然后使用URL.createObjectURL()生成 Blob URL，由于Blob URL本身就是一个同源URL，可以使用该 URL 配合download解决跨域资源的下载以及命名问题