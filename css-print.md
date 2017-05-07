<script type="application/ld+json">
{
  "@type": "Article",
  "headline": "CSS 打印样式",
  "name": "css-print",
  "url": "http://lon.im/post/css-print.html",
  "dateCreated": "2017-01-31",
  "dateModified": "2017-04-02",
  "datePublished": "2017-01-31"
}
</script>

本文主要讲解如何控制打印样式。

## 简介

![chrome 浏览器打印预览](http://lon.im/static/img/css-print.chrome-print-preview.png)

使用 CSS 可以控制文档如何正确的显示在不同的媒介 ([Media]) 上。

其中分页媒介 ([Paged Media]) ，不同于连续媒介 ([Continuous Media])，它可以控制文档内容，将其分隔至一个或多个不相关联的页面 (如：书、幻灯片)。

### 基本概念

页面 (Page Sheet) 是物理介质 (如：纸张) 的表面，它包含可打印区域 (Printable Areas) 和不可打印区域 (Non-printable Areas)。用户代理可以调整文档内容的格式，使其显示在可打印区域。

![页面打印区域和不可打印区域](https://www.w3.org/TR/css3-page/PageSheet.png)

页面盒子 (Page Box) 是一个由长边 (Long Edge) 和短边 (Short Edge) 组成的矩形。长边 (或短边) 的方向决定了页面朝向 (Page Orientation)，长边是垂直方向，则页面朝向为纵向 (Portrait Orientation)，反之为横向 (Landscape Orientation)。

CSS 打印无法指定文档是否为双面打印 (Duplex Printing)，是否双面打印应该通过用户代理指定。不管是否双面打印，CSS 打印总是包含左页和右页 (分别通过 `:left`, `:right` 指定) 。（或者说 CSS 打印假定所有文档是双面打印）

### 页面模型 (Page Model)

和 CSS 盒子模型一样，页面盒子模型由外边距 (margin)、边框 (border)、内边距 (padding) 和 内容区域 (content area) 构成。

![页面模型](https://www.w3.org/TR/css3-page/PageBox.png)

其中内容区域和外边距有着特殊的功能：

- 内容区域也叫页面区域 (page area)，第一页的页面区域边界构成了文档的初始的包含块 ([Containing Block])。
- 外边距被分成了 16 个外边距盒子 (page-margin boxes)。每个外边距盒子都有自己的外边距、边框、内边距和内容区域。



页面进度 (Page Progression)方向 是文档被分隔后的页面的排列方向。比如：现代中文页面进度是从左至右；而古代中文的页面进度是从右至左。可以通过设置根元素 (root element) 的 `driection` 和 `writing-mode` 属性来改变页面进度。

页面的“第一页”是左页还是右页，可以由页面进度的方向决定，当页面进度方向为从左至右时，第一页是右页；反之为左页。（事实上也可以通过设置根元素的 `break-before` 属性来强制改变第一页是左页还是右页）

## 使用 CSS 分页

### 引入分页样式

在 CSS 中使用 @media print

```css
@media print {
  body {
    /* 国家标准公文页边距 GB/T 9704-2012 */
    margin: 3.7cm 2.7cm 3.5cm;
  }
}
```

在 CSS 中使用 `@import`

```css
@import url("my-print-style.css") print;
```

在 HTML 中使用 `<link>` 标签

```html
<link rel="stylesheet" media="print" href="my-print-style.css">
```

### 使用 @page

当希望改变页面大小、方向和边距等，就需要用到 `@page` 了。

`@page` 语法如下：

```css
@page :pseudo-class {
    size: A4;
    margin: 3.7cm 2.6cm 3.5cm;
}
```

其中支持的伪类有：
`:left``:right``:first`
`:blank`

**伪类 `:left` 和 `:right`**

需要双面打印时，通常需要将左页和右页设置不同的样式。这时左页和右页可以分别用 `:left` 和 `:right` 来表示。再次强调，**通过 `:left` 和 `:right` 设置左右页面不同样式，并不能代表用户代理会将页面双面排版**

```css
/* 该例子通过分别设置左页和右页不同的左右页面距，为装订边留出更多的空间 */

@page :left {    margin-left: 2.5cm;    margin-right: 2.7cm;}@page :right {    margin-left: 2.7cm;    margin-right: 2.5cm;}
```

**伪类 `:first`**

伪类 `:first` 用于匹配到文档的第一页。

```css
@page {
    margin: 3.7cm 2.6cm 3.5cm; /* 所有页面边距设置为 3.7cm 2.6cm 3.5cm */
}

@page :first {
    margin-top: 10cm; /* 首页上页边距设置为 10cm */
}
```**伪类 `:blank`**

伪类 `:blank` 用于匹配文档的空白页。

```css
h1 {
    break-before: left; /* 一级标题强制分配到左页 */
}

@page :blank {
    @top-center {
        content: "这是空白页";
    }
}
```注意，空白页也可能是左页或右页，设置左页或右页的样式也会显示在空白页上，如果不希望显示在空白页上，可以清除这些样式。

```cssh1 {
    break-before: left;
}

@page :left {
    @left-center {
        content: "这是左页";
    }
}

@page :right {
    @right-center {
        content: "这是右页";
    }
}

@page :blank {
    @left-center,
    @right-center {
        content: none; /* 如果是空白页则不显示 */
    }
}```支持的属性有：

`size`
`marks`
`bleed`

// 待续……

参考链接：<https://www.w3.org/TR/css3-page/>

[Media]: https://www.w3.org/TR/CSS22/media.html
[Paged Media]: https://www.w3.org/TR/css3-page/#intro
[Continuous Media]: https://www.w3.org/TR/CSS22/media.html#continuous-media-group
[Containing Block]: http://www.w3.org/TR/CSS21/visudet.html#containing-block-details
