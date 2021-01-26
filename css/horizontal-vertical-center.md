# 水平垂直居中

> 居中的场景在业务中是很常见的

### absolute 负margin

```markup
<div class="width-hr">
  <div class="child">width-hr</div>
</div>
```

```css
.width-hr {
  position: relative;
  width: 400px;
  height: 400px;
}
.child {
  position: absolute;
  width: 200px;
  height: 200px;
  top: 50%;
  left: 50%;
  margin-top: -100px;
  margin-left: -100px;
  text-align: center;
  line-height: 200px;
  background: red;
}
```

### absolute transform

```markup
<div class="top all-hr">
  <div class="all-child">all-hr</div>
</div>
```

```css
.all-hr {
  position: relative;
  width: 400px;
  height: 400px;
}

.all-child {
  position: absolute;
  width: 200px;
  height: 200px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  line-height: 200px;
  background: red;
}
```

### flex

```markup
<div class="top flex-hr">
  <div class="flex-child">flex-hr</div>
</div>
```

```css
.flex-hr {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 400px;
  height: 400px;
}

.flex-child {
  width: 200px;
  height: 200px;
  background: red;
  text-align: center;
  line-height: 200px;
}
```

### table

```markup
<div class="top table-hr">
  <div class="table-child">flex-hr</div>
</div>
```

```css
.table-hr {
  display: table-cell;
  vertical-align: middle;
  text-align: center;
  width: 400px;
  height: 400px;
}

.table-child {
  display: inline-block;
  width: 200px;
  height: 200px;
  background: red;
}
```

