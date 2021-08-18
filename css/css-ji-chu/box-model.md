# 盒子模型



> 当对一个文档进行布局（lay out）的时候，浏览器的渲染引擎会根据标准之一的 CSS 基础框盒模型（CSS basic box model），将所有元素表示为一个个矩形的盒子（box）。CSS 决定这些盒子的大小、位置以及属性（例如颜色、背景、边框尺寸…）

每个盒子都有四部分组成，content（内容宽度），内边距（padding），边框（border），外边距（margin）

### 盒子的宽度

* 标准的盒子宽度 width = 内容宽度（content）+ padding + border + margin
* IE低版本盒子宽度 width = 内容宽度（content + padding + border）+ margin

**box-sizing** box-sizing 可以指定一个盒子的总宽度和总高度是什么方式

1. content-box \(默认\): 指定的宽度就是这个元素的宽度
2. border-box: 设置的border和padding会在元素的宽度里面

### 外边距合并

块级元素的上外边距和下外边距有时会合并（或折叠）为一个外边距，其大小取其中的最大者，这种行为称为外边距折叠（margin collapsing），有时也翻译为外边距合并。 注意浮动元素和绝对定位元素的外边距不会折叠。

1. 相邻元素之间合并 

   毗邻的两个元素之间的外边距会折叠

```markup
<p>this is first p element</p>
<p>this is second p element</p>
```

```css
p {
  margin: 5px 0 10px 0px;
}
```

1. 父元素与其第一个或最后一个子元素之间

如果在父元素与其第一个子元素之间不存在边框、内边距、行内内容，也没有创建块格式化上下文、或者清除浮动将两者的 margin-top 分开；或者在父元素与其最后一个子元素之间不存在边框、内边距、行内内容、height、min-height、max-height将两者的 margin-bottom 分开，那么这两对外边距之间会产生折叠。此时子元素的外边距会“溢出”到父元素的外面

```markup
<div>
  <p>this is first p element</p>
  <p>this is second p element</p>
</div>
```

```css
div {
  margin: 20px 0;
}
p {
  margin: 5px 0 10px 0px;
}
```

1. 空的块级元素

   如果一个块级元素中不包含任何内容，并且在其 margin-top 与 margin-bottom 之间没有边框、内边距、行内内容、height、min-height 将两者分开，则该元素的上下外边距会折叠

