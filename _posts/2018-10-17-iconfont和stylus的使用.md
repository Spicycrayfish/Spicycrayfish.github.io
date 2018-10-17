---
layout:     post
title:      iconfont和stylus的使用
subtitle:   前端项目开发过程中可能使用到的iconfont和stylus的简单介绍
date:       2018-10-17
author:     Tank
header-img: img/post-bg-miui6.jpg
catalog: true
tags:
    - 前端
---

> 本文首次发布于 [Tank's Blog](https://spicycrayfish.github.io/), 作者 [@谭轲](http://github.com/Spicycrayfish) ,转载请保留原文链接.



# iconfont

登录  [iconfont](http://www.iconfont.cn/)  官网，将想要使用的图标添加到 **我的项目** 中，下载至本地，下载后会有多个文件，其中我们需要使用到项目中的文件有：
* `iconfont.css`
* `iconfont.eot`
* `iconfont.svg`
* `iconfont.ttf`
* `iconfont.woff`

在需要使用到图标的组件或 `main.js` 文件中引入 `iconfont.css` 文件；

> 注意：将文件添加到项目中后，`iconfont.css` 文件中对 `iconfont.eot` `iconfont.svg` `iconfont.ttf` 文件的引用路径可能需要改变。

之后即可通过 `iconfont.css` 中定义的类名来使用对应的图标，如：

`<i class="iconfont icon-xxx"></i>`
或者使用十六进制的 **unicode** 来使用，浏览器打开 `demo_unicode.html` 文件可查看图标对应的 **unicode**，如：
`<i class="iconfont">&#x33;</i>`



如果后期需要添加新的图标，在 [iconfont](http://www.iconfont.cn/) 官网上将新的图标添加到我的项目中的对应项目里，再重新下载所有图标。下载完成后，用新的字体文件（`iconfont.eot` `iconfont.svg` `iconfont.ttf` `iconfont.woff`）替换掉原来的字体文件，另外还需要替换 `iconfont.css` 文件中有关 **base64** 数据的 **url** 代码。



# stylus

#### stylus 的安装

```
npm install stylus --save
npm install stylus-loader --save
```

安装完成后，将 `<style>` 标签的 `lang` 属性设置为 `stylus`，即可在标签中使用stylus

> 注意：使用 `scoped` 属性可限制样式的作用域



> tips：在 `reset.css` 中将 `html` 标签的 `font-size` 样式设置为 **50px**，则有：
>
> 1rem = html font-size = 50px，这样方便以后开发中 **rem** 数值的计算。



####stylus变量

定义变量：
创建 `varibles.styl` 文件，定义想使用的变量：

```stylus
$bgColor = #9900ff
```

使用变量：

引入 `stylus` 变量文件，并使用定义好的变量：

```stylus
@import '../../../assets/styles/varibles.styl'

.back
  background-color: $bgColor
```



#### stylus代码封装

stylus 可封装一段样式代码（如省略号样式），方便各个地方使用。首先创建封装样式文件如 `mixins.styl`，在其中定义代码块：

```stylus
ellipsis()
  white-space: nowrap
  overflow: hidden
  text-overflow: ellipsis
```

在其它地方使用该样式代码块，引入封装文件并使用，实例代码：

```stylus
@import '../../../assets/styles/mixins.styl'

.ellipsis
  ellipsis()
```



#### 样式穿透

有时候会需要 `<style>` 标签中的部分样式不受 `scoped` 限制（比如要对引入的第三方组件的部分样式进行修改），可使用样式穿透解决：

```stylus
.wrapper >>> .swiper-pagination-bullet-active
  background: #fff
```

这样可以不受 `scoped` 的限制。

