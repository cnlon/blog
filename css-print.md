<script type="application/ld+json">
{
  "@type": "Article",
  "headline": "CSS 打印",
  "name": "css-print",
  "url": "https://lon.im/post/css-print.html",
  "dateCreated": "2017-01-31",
  "datePublished": "2017-01-31",
  "dateModified": "2017-07-11"
}
</script>

## 简介

![Chrome 浏览器打印预览](//cdn.lon.im/static/img/css-print.chrome-print-preview.png)

本文主要讲解如何使用 CSS 控制打印样式。

### 基本概念

使用 CSS 可以控制文档如何正确的显示在不同的媒介 (Media) 上。其中分页媒介 (Paged Media) ，不同于连续媒介 (Continuous Media)，它可以控制文档内容，将其分隔至一个或多个不相关联的页面 (如：书、幻灯片)。

页面 (Page Sheet) 是物理介质 (如：纸张) 的表面，它包含可打印区域 (Printable Areas) 和不可打印区域 (Non-printable Areas)。用户代理可以调整文档内容的格式，使其显示在可打印区域。

![页面打印区域和不可打印区域](//cdn.lon.im/static/img/css-print.page-sheet.png)

页面盒子 (Page Box) 是一个由长边 (Long Edge) 和短边 (Short Edge) 组成的矩形。长边的方向决定了页面朝向 (Page Orientation)，长边是垂直方向，则页面朝向为纵向 (Portrait Orientation)，反之为横向 (Landscape Orientation)。

CSS 打印无法指定文档是否为双面打印 (Duplex Printing)，是否双面打印应该通过用户代理指定。不管是否双面打印，CSS 打印总是包含左页和右页 (分别通过 `:left`, `:right` 指定) 。（或者说 CSS 打印假定所有文档是双面打印）

### 页面模型 (Page Model)

和 CSS 盒子模型一样，页面盒子模型由外边距 (margin)、边框 (border)、内边距 (padding) 和 内容区域 (content area) 构成。

![页面模型](//cdn.lon.im/static/img/css-print.page-box.png)

其中内容区域和外边距有着特殊的功能：

- 内容区域也叫页面区域 (Page Area)，第一页的页面区域边界构成了文档的初始的包含块 (Containing Block)
- 页面外边距区域是透明的，环绕在页面区域周围。在 CSS3 中，可以用于创建页眉和页脚，详见下文 <a href="#page-margin-boxes">页面外边距盒子</a>

页面进度 (Page Progression)方向 是文档被分隔后的页面的排列方向。比如：现代中文页面进度多是从左至右；而古代中文的页面进度则相反。可以通过设置根元素 (root element) 的 `direction` 和 `writing-mode` 属性来改变页面进度。

页面的“第一页”是左页还是右页，可以由页面进度的方向决定，当页面进度方向为从左至右时，第一页是右页；反之为左页。（事实上也可以通过设置根元素的 `break-before` 属性来强制改变第一页是左页还是右页）

## 引入打印样式的三种方式

在 CSS 中使用 `@media print`

```css
@media print {
    body {
        background-color: white;
    }
    img {
        visibility: hidden;
    }
    a::after {
        content: "(" attr(href) ")"; /* 所有链接后显示链接地址 */
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

在 `@media print` 或 my-print-style.css 中，可以自由的修改大部分样式。

## 使用 @page

使用打印媒介查询可以自定义很多样式，当希望改变页面大小、边距等，就需要用到 `@page` 了。页面上下文 (Page Context) 中仅支持部分 CSS 属性，支持的属性有：`margin`、`size`、`marks`、`bleed` 以及页面外边距盒子等，不支持的属性将会被忽略。

<h4 id="page-margin-boxes">页面外边距盒子 (CSS3)</h4>

注：常见浏览器都不支持该属性，推荐使用 [Prince](https://www.princexml.com/)

页面的外边距被分成了 16 个页面外边距盒子。每个外边距盒子都有自己的外边距、边框、内边距和内容区域。页面外边距盒子用于创建页眉和页脚，页眉和页脚是页面的一部分，用于补充信息，如页码或标题。

![page-margin-boxes](//cdn.lon.im/static/img/css-print.page-margin-boxes.png)

页面外边距盒子需要在 `@page` 下使用，使用起来和伪类类似，也包含 `content` 属性。

```css
@page {
    /* 页面内容区域底部添加一条 1px 的灰线 */
    @bottom-left, @bottom-center, @bottom-right {
        border-top: 1px solid gray;
    }

    /* 页脚中间显示格式如 "第 3 页" 的页码 */
    @bottom-center {
        content: "第" counter(page) "页";
    }
}
```

#### 属性

##### `margin` (CSS2.1)

`margin` 系列属性（`margin-top`、`margin-right`、`margin-bottom`、`margin-left` 和 `margin`）用于指定页面外边距大小。

在 CSS2.1 中，页面上下文中只支持 `margin` 系列属性。而且因为 CSS2.1 的页面上下文中没有字体的概念，`margin` 系列属性的值的单位不支持 `em` 和 `ex`。

```css
@page {
    size: A4 portrait;
    margin: 3.7cm 2.6cm 3.5cm; /* 国家标准公文页边距 GB/T 9704-2012 */
}
```

##### `size` (CSS3)

`size` 属性支持 `auto`、`landscape`、`portrait`、`<length>{1,2}` 和 `<page-size>`。

- 默认值为 `auto`，表示页面大小和方向由用户代理决定
- `landscape` 指定页面为横向，如果 `<page-size>` 没有指定，大小则由用户代理决定
- `portrait` 指定页面为纵向，如果 `<page-size>` 没有指定，大小则由用户代理决定
- `<length>{1,2}` 表示指定页面大小，填写两个值则分别指定页面盒子的宽度和高度，填写一个值则同时指定宽度和高度。在 CSS3 中，值的单位支持 `em` 和 `ex`，大小相对于页面上下文中字体的大小
- `<page-size>` 也用于指定页面大小，等价于使用 `<length>{1,2}`。常用的值有：`A3`、`A4`、`A5`、`B4` 和 `B5` 等，详细尺寸请参考 [ISO 216]。`<page-size>` 可以与 `landscape` 或 `portrait` 组合同时指定页面方向。

#### 伪类

页面上下文也支持使用伪类，其中支持的伪类有：`:left`、`:right`、`:first` 和 `:blank`。

##### 伪类 `:left` 和 `:right`

需要双面打印时，通常需要将左页和右页设置不同的样式（如页边距、页码位置）。这时左页和右页可以分别用 `:left` 和 `:right` 表示。再次强调，**通过 `:left` 和 `:right` 设置左右页面不同样式，并不代表用户代理会将页面双面打印**

```css
/* 通过分别设置左页和右页不同的左右页面距，为装订边留出更多的空间 */

@page :left {
    margin-left: 2.5cm;
    margin-right: 2.7cm;
}

@page :right {
    margin-left: 2.7cm;
    margin-right: 2.5cm;
}
```

##### 伪类 `:first`

伪类 `:first` 用于匹配到文档的第一页。

```css
@page :first {
    margin-top: 10cm; /* 首页上页边距设置为 10cm */
}
```

##### 伪类 `:blank`

伪类 `:blank` 用于匹配文档的空白页。

```css
h1 {
    page-break-before: left; /* 一级标题强制分配到右页 */
}

@page :blank {
    @top-center {
        content: "这是空白页";
    }
}
```

注意，空白页既可能是左页，又可能是右页，设置左页或右页的样式也会显示在空白页上，如果不希望显示在空白页上，可以清除这些样式。

```css
h1 {
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
    @left-center, @right-center {
        content: none; /* 如果是空白页则不显示 */
    }
}
```

## 分页

##### `page-break-before`，`page-break-after`，`page-break-inside` (CSS 2.1)

用于控制元素之前、之后或之中是否分页，**没有生成盒子的块元素不会生效**。

`page-break-before`、`page-break-after` 属性支持 `auto`、`always`、`avoid`、`left`、`right`、`recto` 和 `verso`。

```css
h2 {
    page-break-before: always;
}
```

- `auto` 默认值，表示既不强制分页也不禁止分页
- `always`、`avoid` 表示在该元素之前（或之后）强制或禁止分页
- `left`、`right` 表示在该元素之前（或之后）强制分页，使得下一页出现在左页或右页
- `recto`、`verso` 页面进度从左至右时，分别与 `right` 和 `left` 一致；反之与 `left` 和 `right` 一致

`page-break-inside` 属性仅支持 `auto` 和 `avoid`，表示在元素内允许或禁止分页。

```css
thead, tfoot {
    display: table-row-group;
}
thead, tfoot, tr, th, td {
    page-break-inside: avoid;
}
```

##### `orphans`，`windows` (CSS 2.1)

`orphans` 和 `windows` 用于指定在页面的底部或顶部，元素中允许剩余的最少行数，默认为 2 行。

## 最佳实践

- “白纸黑字”--避免不必要的背景颜色、加深文字颜色等
- 避免打印次要的内容，比如导航栏、侧边栏等
- 链接后显示链接地址
- 做好分页，避免标题、表格单元格等换行

示例：

```css
@media print {
    @page {
        size: A4 portrait;
        margin: 3.7cm 2.6cm 3.5cm;
    }

    h1 {
        page-break-before: always;
    }

    h1, h2, h3, h4, h5, h6,
    thead, tfoot, tr, th, td,
    li {
        page-break-inside: avoid;
    }

    body {
        background-color: white;
        color: black;
    }

    nav, aside {
        display: none;
    }

    a::after {
        content: "(" attr(href) ")";
    }

    thead, tfoot {
        display: table-row-group;
    }
}
```


----

参考链接：

- [*Paged media (CSS 2.1)*](https://www.w3.org/TR/CSS21/page.html) W3C
- [*CSS Paged Media Module Level 3 (W3C Working Draft 14 March 2013)*](https://www.w3.org/TR/css3-page/) W3C
- [*CSS Paged Media Module Level 3 (Editor’s Draft, 22 June 2017)*](https://drafts.csswg.org/css-page-3/) W3C
- [*CSS Pages*](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Pages) MDN
- [*CSS Multi-column Layout Module Level 1 (Editor’s Draft, 22 June 2017)*](https://drafts.csswg.org/css-multicol-1/#break-after) W3C

[ISO 216]: https://en.wikipedia.org/wiki/ISO_216
