---
title: 危险的边缘试探--parcel
date: 2018-07-10 12:30
categories: 打包工具
---

> 零配置的打包工具

在我们日常过工作中会用到webpack或gulp等打包工具，这些得力的打包工具带来工作的流程化和便捷的同时还有复杂繁琐的各种配置，是我快速开始工作之前的必经之路。

parcel打造零配置，只需要你把它加入到你的项目中即可。
```
    parcel index.html
    
```
file-watch这个插件，我们在项目开发中基本上都会用，它会帮助我们监听文件的变化从而重启项目。但是如果项目大了的话，重启1次是一件很耗时的事情。parcel利用worker进程并行编译你的代码，利用现代多核处理器。这是因为它在初始化构建的时候有一个巨大的加速度，它还有一个文件系统缓存，它为更快的后续启动节省了每个文件的编译结果。
<!--more-->
> parcel's transforms work on ASTs so that there is one parse, many transforms, and one code generation per file.

### parcel是怎么工作的
Parcel 将 资源 树转换成 bundle 树。许多其它的打包工具基本上是基于 JavaScript 资源，还有附加在其上的其它格式的资源。例如，在 JS 文件中内联成字符串。 Parcel 是对文件类型无感知的，它能按你所期待的方式那样与任意类型的资源工作，且毋须配置。Parcel 的打包流程共有三个步骤。

- 构建资源树
Parcel 接受单个入口资源作为输入，可以是任意类型：一个 JS 文件、HTML、CSS 和图片等等。有许多不同的资源类型在 Parcel 中被定义，它知道如何去处理特定的文件类型。资源会被解析，资源的依赖会被提取，资源会被转换成最终编译好的形态。此过程创建了一个资源树。

- 构建 bundle 树
一旦资源树被构建好，资源会被放置在 bundle 树中。首先一个入口资源会被创建成一个 bundle，然后动态的 import() 会被创建成子 bundle ，这引发了代码的拆分。
当不同类型的文件资源被引入，兄弟 bundle 就会被创建。例如你在 JavaScript 中引入了 CSS 文件，那它会被放置在一个与 JavaScript 文件对应的兄弟 bundle 中。
如果资源被多于一个 bundle 引用，它会被提升到 bundle 树中最近的公共祖先中，这样该资源就不会被多次打包。

- 打包
在 bundle 树被构建之后，每个 bundle 都会被 packager 写到一个特定文件类型的文件中。packagers 知道如何从每个资源中将代码合并起来，生成到最终被浏览器加载的文件中。

[parceljs中文网][1]

> 看完完整的介绍后，感觉parcel像是一个搅碎机，什么东西丢进去，都能去解析、分析依赖、转换及生成代码。而开发者只需要丢东西进去就好了。


  [1]: https://parceljs.org/