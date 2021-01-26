# 页面布局

1. 单列布局
2. 两列自适应布局
3. 三栏布局
4. 等高布局
5. 粘连布局

### 单列布局

常见的单列布局

* header content footer 等宽
* header footer 等宽全屏 content 略窄中间

#### 单列布局（等宽）

```markup
<div class="header"></div>
<div class="content"></div>
<div class="footer"></div>
```

```css
.header{
  max-width: 960px;
  width: 500px;
  margin: 0 auto;
  height: 40px;
  border: 1px solid #333
}

.content {
  max-width: 960px;
  width: 500px;
  margin: 0 auto;
  height:180px;
  border: 1px solid #333;
}
.footer {
  max-width: 960px;
  width: 500px;
  margin: 0 auto;
  height:40px;
  border: 1px solid #333
}
```

result prefer [jsRun](http://jsrun.pro/XQbKp)

#### 单列布局 （content 略窄）

```css
.header {
  min-width: 500px;
  height: 40px;
  border: 1px solid #333
}
.content {
  max-width: 600px;
  height: 180px;
  border: 1px solid #333;
  margin: 0 auto;
}
.footer {
  min-width: 500px;
  height: 40px;
  border: 1px solid #333
}
```

result prefer [jsrun](http://jsrun.pro/bQbKp)

### 两列自适应布局

两列自适应布局是指一列由内容撑开，另一列撑满剩余宽度的布局方式

#### 使用float overflow

```markup
<div class="content">
  <div class="left">
    left
  </div>
  <div class="right">
    right
    <p>right</p>
    right
  </div>
</div>
```

```css
.content {
  overflow: hidden;
  zoom: 1;
  margin-bottom: 20px;
}
.left {
  float: left;
  border: 1px solid #333
}
.right {
  overflow: hidden;
  zoom: 1;
  border: 1px solid #333
}
```

result prefer [jsRun](http://jsrun.pro/2QbKp)

#### 使用flex

```css
.content {
  display: flex;
}
.left{
  width: 50px;
}
.right {
  flex: 1
}
```

### 三栏布局

::: tip 中间列自适应宽度，旁边两侧固定宽度 :::

1. float 基本布局 BFC 布局
2. 圣杯布局 基于float 
3. 双飞翼布局
4. 绝对定位布局 
5. flex 布局
6. table 布局
7. grid 布局

#### 使用 float BFC

```markup
<article class="left-center-right">
  <div class="left"></div>
  <div class="right"></div>
  <!-- 疑问 why center 放到最后 -->
  <div class="center">
      <p>Float solution </p>
      <p>this is a float layout</p>
      <p>this is a float layout</p>
  </div>
</article>
```

```css
.left {
  float: left;
  width: 50px;
  height: 200px;
  border: 1px solid #333
}
.right {
  float: right;
  width: 50px;
  height: 200px;
  border: 1px solid #333
}
.center {
  overflow: hidden;
  border: 1px solid #333;
  height: 200px;
}
```

result prefer [js Run](http://jsrun.pro/IQbKp)

#### 使用圣杯布局

比较特殊的三栏布局，同样也是两边固定宽度，中间自适应，唯一区别是dom结构必须是先写中间列部分，这样实现中间列可以优先加载。

```markup
<article class="container">
  <div class="center">
      <p>Float solution </p>
      <p>this is a float layout</p>
      <p>this is a float layout</p>
  </div>
  <div class="left">left</div>
  <div class="right">right</div>  
</article>
```

```css
.container {
  padding-left: 220px;
  padding-right: 220px;
}

.left {
  float: left;
  width: 200px;
  height: 400px;
  background: red;
  margin-left: -100%;
  position: relative;
  left: -220px;
}
.center {
  float: left;
  width: 100%;
  height: 400px;
  background: yellow;
}
.right {
  float: left;
  width: 200px;
  height: 400px;
  margin-left: -200px;
  background: blue;
  position: relative;
  right: -220px;
}
```

result prefer [JS run](http://jsrun.pro/NQbKp)

#### 双飞翼布局

::: tip 同样也是三栏布局，在圣杯布局基础上进一步优化，解决了圣杯布局错乱问题，实现了内容与布局的分离。而且任何一栏都可以是最高栏，不会出问题 :::

```markup
<article class="container">
  <div class="center">
    <div class="inner">
      <p>Float solution </p>
      <p>this is a float layout</p>
      <p>this is a float layout</p>
    </div>
  </div>
  <div class="left">left</div>
  <div class="right">right</div>  
</article>
```

```css
.container {
  min-width: 600px
}

.left {
  float: left;
  width: 200px;
  height: 400px;
  margin-left: -100%;
  background: red;
}
.center {
  float: left;
  width: 100%;
  height: 400px;
  background: blue;
}
.center .inner {
  margin: 0 200px;
}
.right {
  float: left;
  width: 200px;
  height: 400px;
  margin-left: -200px;
  background: yellow
}
```

result prefer [JS run](http://jsrun.pro/FQbKp)

#### 使用绝对定位

```markup
<article class="content">
  <div class="left"></div>
  <div class="center">
      <p>Absolute solution</p>
      <p>this is a absolute layout</p>
  </div>
  <div class="right"></div>
</article>
```

```css
.content {
  position: relative
}
.content div {
  position: absolute;
}

.left {
  left: 0;
  width: 100px;
  height: 400px;
  background: red
}

.right {
  right: 0;
  width: 100px;
  height: 400px;
  background: blue
}

.center {
  left: 100px;
  right: 100px;
  height: 400px;
  background: yellow;
}
```

result prefer [JS run](http://jsrun.pro/xQbKp)

#### 使用flex

```css
.centent {
  display: flex;
}

.left {
  width: 200px;
  height: 400px;
  background: red;
}

.center {
  flex: 1;
  background: blue;
}

.right {
  width: 200px;
  height: 400px;
  background: yellow
}
```

result prefer [JS run](http://jsrun.pro/GQbKp)

#### 使用table

```css
.content {
  display: table;
  width: 100%;
}
.content div {
  display: table-cell;
  min-height: 400px;
}

.left {
  width: 200px;
  background: red;
}
.center {
  background: blue
}
.right {
  width: 200px;
  background: yellow
}
```

result prefer [JS run](http://jsrun.pro/5QbKp)

#### 使用grid

```css
.content {
  display: grid;
  grid-template-rows: auto;
  grid-template-columns: 200px auto 200px;
}
.left {
  background: red;
}
.center {
  background: blue
}
.right {
  background: yellow
}
```

result prefer [JS run](http://jsrun.pro/DQbKp)

