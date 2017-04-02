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

![chrome 浏览器打印预览](http://lon.im/static/img/css-print.chrome-print-preview@2x.png)

使用 CSS 可以控制文档如何显示正确的在不同的媒介 ([Media]) 上。

其中分页媒介 ([Paged Media]) ，不同于连续媒介 ([Continuous Media])，它可以控制文档内容，将其分隔至一个或多个不相关联的页面 (如：书本、幻灯片)。

### 分页媒介的三种使用方式

#### 在 CSS 中使用 @media print

```css
@media print {
  body {
    /* 国家标准公文页边距 GB/T 9704-2012 */
    margin: 3.7cm 2.7cm 3.5cm;
  }
}
```

#### 在 CSS 中使用 `@import`

```css
@import url("my-print-style.css") print;
```

#### 在 HTML 中使用 `<link>` 标签

```html
<link rel="stylesheet" media="print" href="my-print-style.css">
```

### 基本概念

页面 (Page Sheet) 是物理介质 (如：纸张) 的表面，它包含可打印区域 (Printable Areas) 和不可打印区域 (Non-printable Areas)。用户代理可以调整文档内容的格式，使其显示在可以打印区域。

![页面打印区域和不可打印区域](https://www.w3.org/TR/css3-page/PageSheet.png)

页面盒子 (Page Box) 是一个由长边 (Long Edge) 和短边 (Short Edge) 组成的矩形。长边 (或短边) 的方向决定了页面朝向 (Page Orientation)，长边是垂直方向，则页面朝向为纵向 (Portrait Orientation)，反之为横向 (Landscape Orientation)。

CSS 打印无法指定文档是否为双面打印 (Duplex Printing)，是否双面打印应该通过用户代理指定。不管是否双面打印，CSS 打印总是包含左页和右页 (分别通过 `:left`, `:right` 指定) 的概念。（或者说 CSS 打印假定所有文档是双面打印）

### 页面模型 (Page Model)

和 CSS 盒子模型一样，页面盒子模型由外边距 (margin)、边框 (border)、内边距 (padding) 和 内容区域 (content area) 构成。

![页面模型](https://www.w3.org/TR/css3-page/PageBox.png)

其中内容区域和外边距有着特殊的功能：

- 内容区域也叫页面区域 (page area)，第一页的页面区域边界构成了文档的初始的包含块 ([Containing Block])。
- 外边距被分成了 16 个外边距盒子 (page-margin boxes)。每个外边距盒子都有自己的外边距、边框、内边距和内容区域。

页面进度 (Page Progression) 是文档被分隔后的页面的顺序方向。比如：现代中文页面进度是从左至右；而古代中文的页面进度是从右至左。可以通过设置根元素 (root element) 的 `driection` 和 `writing-mode` 属性来改变页面进度。

页面的“第一页”是左页还是右页，可以由页面进度的方向决定，当页面进度方向为从左至右时，第一页是右页；反之为左页。（事实上也可以通过设置根元素的 `break-before` 属性来强制改变第一页是左页还是右页）


// 待续……

参考链接：<https://www.w3.org/TR/css3-page/>

[Media]: https://www.w3.org/TR/CSS22/media.html
[Paged Media]: https://www.w3.org/TR/css3-page/#intro
[Continuous Media]: https://www.w3.org/TR/CSS22/media.html#continuous-media-group
[Containing Block]: http://www.w3.org/TR/CSS21/visudet.html#containing-block-details
