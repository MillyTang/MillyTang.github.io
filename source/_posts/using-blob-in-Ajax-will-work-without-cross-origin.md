---
title: 配合Ajax 使用 new Blob 下载文件不会导致跨域问题
date: 2021-04-15 15:48:12
tags: Blob, URL.createObjectURL 
---

配合 Ajax 使用 new Blob 下载文件不会导致跨域问题：

> Blob URL / Object URL 是一种伪协议，可以让 Blob 和 File 对象用作图像和二进制数据下载链接等 URL 源

当你需要在 HTML 中通过 URL 来引用一个 File 对象时，你可以创建一个对象 URL，这个对象 URL 是一个标识 File 对象的字符串，当文档关闭时，它们会自动被释放或者通过 URL.revokeObjectURL 来主动释放。

因此 Blob URL 并不能指向一个服务器资源，你无法在其它页面中打开它，只能在当前页面打开。

Blob 为 Web 开发提供了一个非常有用的功能：创建 Blob URL。将二进制数据封装为 Blob 对象，然后使用 URL.createObjectURL()生成 Blob URL，由于 Blob URL 本身就是一个同源 URL，可以使用该 URL 配合 download 解决**跨域资源的下载（配合 XMLHttpRequest 使用）**

> 将文件名放到返回头`Content-Disposition`时，要注意在服务端设置`Access-Control-Expose-Headers: Content-Disposition`, 否则返回头拿不到Content-Disposition头信息 

## URL.createObjectURL 的适用对象

1. 对象缩略图
2. 显示 PDF：注意在 Firefox 中，要让 PDF 嵌入式地显示在 iframe 中（而不是作为下载的文件弹出），必须将 pdfjs.disabled 设为 false

    ```js
      const obj_url = window.URL.createObjectURL(blob);
      const iframe = document.getElementById('viewer');
      iframe.setAttribute('src', obj_url);
      window.URL.revokeObjectURL(obj_url);
    ```

3. 上传视频

    ```js
    const video = document.getElementById('video');
    const obj_url = window.URL.createObjectURL(blob);
    video.src = obj_url;
    video.play()
    window.URL.revokeObjectURL(obj_url);
    ```
