# standard 系列 - 解析HTML文档

> HTML 解析过程的输入由一系列代码点组成，这些代码点通过一个标记化阶段，然后是树构造阶段。输出是一个 Document 对象。

### 解析模型概述

解析HTML也是浏览器页面渲染的一部分，而这个过程是怎么解析的呢？又是怎么产生最终的**Document**对象的呢？

在浏览器工作原理中，浏览器通过网络请求获取HTML资源（文件字节流）是没有办法让渲染引擎理解的，因此要将它转换为渲染引擎能够理解的结构，而这个结构就是**DOM**

**DOM 层面的作用**

1. 页面视角，DOM是生成页面的基本数据结构
2. 脚本视角，DOM提供给JS操作DOM的接口，从而可以对节点进行处理
3. 安全视角，一段不安全的DOM在解析的时候就会被拦截

**DOM 树的生成**

DOM树的生成大概可以看下面的图。

\*\*HTML parsing 不是等HTML文档加载完成之后再解析，而是边加载变解析。

首先网络进程接收到响应头中的`Content-type`，如果是`text/html`，就会判断是`html`文件，会选择创建一个渲染进程，_网络进程和渲染进程之间会有一个共享的管道_，网络进程获取到数据之后，会通过这管道传输到渲染进程，渲染进程拿到数据之后就会进行`解析`，也就是通过**HTML Parsing**，结束之后就会产生DOM对象。

![DOM-parsing](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/browser/dom_parse.png)

### 解析HTML文档

网络进程获取到HTML文档后，是怎么一步一步的将HTML文档解析成DOM树的。

**首先，通过分词器将字节流转换成Token**

解析的过程是按照HTML语法将字节流转换成不同类型的**token**，例如`startTag`, `endTag`，`Text`等。

**之后，就是将token解析成DOM节点，以及将DOM节点添加到DOM树中**

在分词阶段，HTML Parsing 会维护一个`Token栈`，之前生成token会按顺序进行入栈，入栈的过程中，会有以下的操作：

1. 如果判断当前**token**是`startTag`类型的话，解析器会创建一个DOM节点，添加到DOM树中，父节点就是相邻的节点
2. 如果解析的**token**是`Text`文本类型的话，解析器会创建一个文本节点，添加到栈中相邻的的DOM节点，也就是父节点
3. 如果解析的**token**是`endTag`类型的话，解析器就会查看栈顶的元素是不是对应的`startTag`，如果是，将`startTag`从栈中弹出，表明此节点解析完成

在解析的时候，解析器会默认的创建一个根为`Document`的DOM结构，同时将document`startTag`的token添加到栈中。

实现HTML Parsing大概有以下几个步骤:

1. 初始化FSM（状态机）
2. 解析标签（parseTag）
3. emitToken
4. attribute
5. 构造树
6. combineText

#### 初始化FSM（状态机）

这里简单的描述了[状态机](zh/webApi/browser/posts/statusMachine.html)的实现方式。接下来用状态机简单实现ParseHTML。

在解析HTML中，常见的state有:

* `Data state` - 主要接受下一个输入的字符
* `tagOpen state` - 开始标签，主要是`<`之后的字符
* `tagname state` - 标签名称
* `endTagOpen state` - 结束标签，主要`>`
* `beforeAtributeName state` - 处理属性名之前的操作
* `afterAttributeName state` - 处理属性名之后的操作状态

输入字节流来自网络进程，例如我们编写的HTML 文档，以下面的HTML为输入流来进行解析。

```markup
<html maaa=a >
  <head>
    <style>
      #container{
          width:500px;
          height:300px;
          display:flex;
          background-color:rgb(255,255,255);
      }
      #container #myid{
        width:200px;
        height:100px;
        background-color:rgb(255,0,0)
      }
      #container .c1{
        flex:1;
        background-color:rgb(0,255,0)
      }
    </style>
  </head>
  <body>
      <div id="container">
        <div id="myid"></div>
        <div class="c1"></div>
      </div>
  </body>
</html>
```

```javascript
// end of file
let EOF = Symbol('EOF');
// 状态机入口 
function dataState() {

}
// parseHtml.js
module.exports = function parseHtml(html) {
  // 状态机初始处理状态
  let state = dataState;
  // 循环处理每个字节流
  for(let c of html) {
    state = state(c)
  }
  // 结束状态机
  state = state(EOF)
}
```

这里初始化了标识`EOF`（其实是文件结束的token）、`dataState`（状态机入口），以及遍历字节流字符，处理每个不同的_token_会返回下一个处理对应类型的_token_，最后结束状态机。

#### 解析标签

在这一步，主要实现解析标签的状态机，比如`startTag`、 `endTag`、`tagName`、`selfCloseStartTag`以及`beforeAttributeName`等状态。

在这里先完善`dataState`入口状态机，如果接受到的字符是`<`，则就会被认为是一个**startTag**\(开始标签\)，之后就会进入到`tagOpenState`状态机，如果`c`是**EOF**，则就_emit_一个类型为**EOF**，表明是结束状态机。

```javascript
// end of file
let EOF = Symbol('EOF');
// 当前token
let currentToken = null;
// 当前token属性
let currentAttribute = null;
// 当前token文本节点
let currentTextNode = null;


/**
 * 初始状态
 * @params c 
 * @return state 
 **/ 
function dataState(c) {
  if(c == '<') {
    return tagOpenState
  }else if(c == 'EOF') {
    return emit({
      type: 'EOF'
    })
  }else {
    return emit({
      type: 'text',
      content: c
    })
  }
}
// parseHtml.js
module.exports = function parseHtml(html) {
  // 状态机初始处理状态
  let state = dataState;
  // 循环处理每个字节流
  for(let c of html) {
    state = state(c)
  }
  // 结束状态机
  state = state(EOF)
}
```

上述代码中，定义了状态机入口、token栈、以及记录当前节点的信息。接下来就实现不同类型的**state**，首先实现的`tagOpenState`，这个状态是实现**startTag**起始标签的。

```javascript
/**
 * 起始标签
 * @param c 
 * @return state
 **/ 
function tagOpenState(c) {
  if(c == '!') {
    // 处理doctype 
    return markUpOpenState
  }else if(c == '/') {
    // 结束标签打开状态
    return endTagOpenState
  }else if(c.match(/^[a-xA-Z]$/)) {
    // 处理标签名称, 将当前字节流代理到tagName
    currentToken = {
      type: 'startTag',
      tagName: ''  
    }
    return tagNameState(c)
  }else if(c == 'EOF') {
    emit({
      type: 'EOF'
    })
  }else {
    emit({
      type: 'text',
      content: c
    })
  }
}
```

第一个状态`tagOpenState`是解析开始标签的，例如`<div>`，如果消费的字符是`d`，就会进入到解析标签name名称状态`tagNameState`，下面看如何解析标签名称的。

```javascript
function tagNameState(c) {
  // 匹配到 Tab LF FF whiteSpace
  // 处理标签的属性
  if(c.match(/^[\t\n\f ]$/)) {
    return beforeAttributeName
  }else if(c == '/') {
    // 处理自闭合标签
    return selfClosingStartTag
  }else if(c == '>') {
    // 开始标签处理结束，重新进入入口状态，处理标签内容
    emit(currentToken);
    return dataState
  }else if(c.match(/^[a-xA-Z]$/)) {
    // 处理标签字符名称
    currentToken.tagName += c;
    return tagNameState
  }else if(c == 'EOF') {
    emit({
      type: 'EOF'
    })
  }else {
    currentToken.tagName += c;
    return tagNameState
  }
}
// 处理自闭合标签
function selfClosingStartTag(c) {
  if(c == ">") {
    currentToken.isSelfClosing = true;
    emit(currentToken)
    return dataState
  }else if(c == "EOF") {

  }else {

  }
}
```

标签名的解析主要包括，tagName 标签名称，attributeName 属性名称以及自闭合标签。根据上述的用来测试的HTML文档为例子，首先接收的标签为`<html maaa=a>`，它包含了`tagName`、`beforeAttributeName`和`attributeName`状态

紧接着看`beforeAttributeName`状态的处理

```javascript
function beforeAttributeName(c) {
  // 如果是 Tab LF FF whiteSpace 则忽略字符
  if(c.match(/^[\t\n\f ]$/)) {
    return beforeAtributeName
  }else if(c == "=") {
    // 这是一个意外的先等号后属性的分析错误，在当前标记中启动一个新属性，将该属性的名称设置为当前输入字符，将其值设置为空字符串，切换到属性名(attributeName)状态
    currentAttribute = {
      name: "",
      value: ""
    }
    return attributeName(c)

  }else if(c == ">" || c == "/" || c == EOF) {
    // 当前标签 新增一个属性
    return afterAttributeName(c)
  }else {
    currentAttribute = {
      name: "",
      value: ""
    }
    return attributeName(c)
  }
}
// 处理属性名称
function attributeName(c) {
  if(c.match(/^[\t\n\f ]$/) || c == "/" || c == EOF || c == ">") {
    return afterAttributeName(c)
  }else if(c == "=") {
    // 处理属性值
    return beforeAtributeValue
  }else if(c == '\u0000') {
    currentAttribute.name += '\ufffd'
  }else if(c == "\"" || c == "'" || c == "<") {
    //  Treat it as per the "anything else" entry below.
    currentAttribute.name += c
    return attributeName
  }else {
    c.name += c
    return attributeName
  }
}
// 处理属性之后的操作
function afterAttributeName(c) {
  if(c.match(/^[\t\n\f ]$/)) {
    return afterAttributeName
  }else if(c == "/") {
    return selfClosingStartTag;
  }else if(c == "=") {
    return beforeAtributeValue
  }else if(c == ">") {
    // 遇到字符 > , emit 当前token 添加到DOM中
    currentToken[currentAttribute.name] = currentAttribute.value;
    emit(currentToken)
    return data
  }else if(c == EOF) {
    emit({
      type: "EOF"
    })
  }else {
    currentToken[currentAttribute.name] = currentAttribute.value;
    currentAttribute = {
      name: "",
      value: ""
    }
    return attributeName(c)
  }
}
```

到此为止，已经解析到了`<html maaa=a>`中的标签名称以及其属性名称。

```javascript
// currentToken
currentToken = {
  type: 'startTag',
  tagName: 'html'
}
currentAttribute = {
  name: 'maaa',
  value: ''
}
```

属性名称解析结束状态机就进入到了属性值解析，接下来看属性值解析的状态机。

```javascript
function beforeAttributeValue(c) {
  if(c.match(/^[\t\n\f ]$/) || c == "/" || c == ">" || c == EOF) {
    return beforeAttributeValue
  }else if(c == "\"") {
    return doubleQuotedAttributeValue;
  }else if(c == "\'") {
    return singleQuotedAttributeValue
  }else if(c == ">") {
    // 标签解析结束，重新进入入口状态机解析下一个字节流
    emit(currentToken)
    return dataState
  }else {
    // 解析属性值
    return UnquotedAttibutedValue(c)
  }
}
// 属性值解析
function UnquotedAttibutedValue(c) {
  // Tab LF FF whitespace 
  if(c.match(/^[\t\n\f ]$/)) {
    currentToken[currentAttribute.name] = currentAttribute.value;
    return beforeAttributeName
  }else if(c == "/") {
    currentToken[currentAttribute.name] = currentAttribute.value;
    return selfClosingStartTag
  }else if(c == ">") {
    // 属性值处理完成 标签处理结束，重新进入入口状态机
    currentToken[currentAttribute.name] = currentAttribute.value;
    emit(currentToken)
    return dataState
  }else if(c == "\u0000") {

  }else if(c == "\'" || c == "\"" || c == "<" || c == "=" || c == "`") {
    // 这是一个意外的非引号字符属性值分析错误。请将其视为下面的“ anything else”条目。
  }else if(c == EOF) {
    emit({
      type: 'EOF'
    })
  }else {
    currentAttribute.value += c;
    return UnquotedAttibutedValue
  }
}
// 处理双引号属性值
function doubleQuotedAttributeValue(c) {
  if(c == "\"") {
    currentToken[currentAttribute.name] = currentAttribute.value
    return afterQuotedAttributeValue
  }else if(c == "&") {
    // TODO
  }else if(c == "\u0000") {
    // TODO
  }else if(c == EOF) {
    emit({
      type: EOF
    })
  }else {
    currentAttribute.value += c;
    return doubleQuotedAttributeValue
  }
}
// 处理单引号属性值
function singleQuotedAttributeValue(c) {
  if(c == "\'") {
    currentToken[currentAttribute.name] = currentAttribute.value
    return afterQuotedAttributeValue
  }else if(c == "&") {
    // TODO
  }else if(c == "\u0000") {
    // TODO
  }else if(c == EOF) {
    emit({
      type: EOF
    })
  }else {
    currentAttribute.value += c;
    return doubleQuotedAttributeValue
  }
}
```

这一步就相当于将`<html maaa=a>`处理完成了，_currentAttribute_的值也补充完成了。最后遇到`>`字符，表明该**html**标签已经处理完成了，后续操作就是将当前的属性挂载到当前的**currentToken**中，然后通过_emit_方式将此**token**添加到**DOM**树中。

接下来看**HTML-parsing**是如何将`token`添加到DOM树中的

#### emitToken以及构造DOM树

每解析完一个标签，就会将改token添加到DOM树中，下面_emit_主要负责的就这部分内容。

```javascript
// emit 添加token到DOM树中
function emit(token) {
  // 获取栈顶的元素，给其添加子元素
  let top = stack[stack.length - 1];

  if(token.type == 'startTag') {
    // 创建一个元素节点
    let el = {
      type: 'element',
      children: [],
      attributes: []
    }
    // 给元素添加 tagName
    el.tagName = token.tagName;

    // 给元素添加属性
    for(let p in token) {
      if(p != 'type' && p != 'tagName') {
        el.attributes.push({
          name: p,
          value: token[p]
        })
      }
    }
    // 父元素与子元素之间的绑定
    top.children.push(el)
    el.parent = top

    // 自闭合标签不需要入栈
    if(!token.isSelfClosing) {
      stack.push(el)
    }

    currentTextNode = null

  }else if(token.type == 'endTag') {
    // 处理结束标签 对应的标签从栈中弹出

    // 异常处理 
    if(top.tagName !== token.tagName) {
      throw new Error('tagType does not match')
    }else {
      // 弹出栈中对应的开始标签
      stack.pop()
    }

    currentTextNode = null

  }else if(token.type == 'text') {
    // 处理文本节点
    if(currentTextNode == null) {
      currentTextNode = {
        type: 'text',
        content: ''
      }
      top.children.push(currentTextNode)
    }
    currentTextNode.content += token.content
  }
}
```

