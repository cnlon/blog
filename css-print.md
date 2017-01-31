本文主要讲解如何使用 CSS 控制打印样式

## @media

使用 @media print

```html
<style tyle="text/css">
  @media screen
  {
    p.bodyText {
      font-family: verdana, arial, sans-serif;
    }
  }
  @media print
  {
    p.bodyText {
      font-family: georgia, times, serif;
    }
  }
  @media screen, print
  {
    p.bodyText {
      font-size:10pt;
    }
  }
</style>
```

或将控制打印的样式单独写在一个文件中，然后：

使用 `@import`

```html
<style>
  @import url("mystyle.css") print;
</style>
```

或使用 `<link>` 标签

```html
<link rel="stylesheet" media="print" href="mystyle.css">
```

## @page

// 待续...

[CSS Printing - @media Rule](https://www.tutorialspoint.com/css/css_printing.htm)
