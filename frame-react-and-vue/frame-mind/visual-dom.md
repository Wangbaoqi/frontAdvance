# 虚拟DOM

> visual-dom 产生是为了解决DOM操作的性能损耗大的手段。将真实的DOM解析成JS对象，然后通过diff算法计算新老虚拟dom对象的差异，最后将这些差异更新到真实DOM上

### 真实DOM和其解析流程

```text
![real-dom](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/browser/domLayout.png)

浏览器的渲染过程大致分为:
创建```DOM```树 -> 创建```style rules``` -> 构建```render``` 树 -> 布局```Layout``` -> 绘制```painting
```

* 构架DOM树，使用HTML分析器，分析HTML元素，构建一个DOM树
* 生成样式表，使用CSS分析器，分析CSS文件和元素的样式，生成样式表
* 构架render树，将DOM树和CSS规则关联起来，构建一颗render树
* 确定节点坐标，根据render树结构，每个元素在屏幕上显示的精确坐标
* 绘制页面，根据节点坐标，调用每个节点的paint方法，绘制出来

而基于浏览器这样的渲染流程，在操作DOM的过程中，存在的问题:

* 操作DOM 会导致页面进行重绘或者回流
* 频繁操作DOM可能会导致页面的卡顿，影响用户体验
* 影响性能 

### Visual-DOM转化真实DOM

虚拟`DOM`之所以会存在，就是解决了频繁操作DOM带来的性能损耗，为解决浏览器渲染性能而提出来的

它其实就是无论之前操作过多少次DOM，到最后某个时刻会一次性渲染，避免了频繁操作DOM带来损耗

**visual-dom 转换成真实的DOM**

如何生成visual-dom, 首先有一个真实的DOM结构，visual-dom生成之后进行对比

```markup
<div id='visual-dom'>
  <ul id='list'>
    <li class="list-item">item01</li>
    <li class="list-item">item03</li>
    <li class="list-item">item03</li>
  </ul>
  <p class='content'>visual dom</p>
</div>
```

1. 使用对象来表示`dom`节点

```javascript
/**
 * 每个节点表示的对象
 *  @param {String} tagName 标签名称
 *  @param {String} props 标签属性
 *  @param {String} children 子元素
 **/
function Element(tagName, props, children) {
  this.tagName = tagName
  this.props = props
  this.children = children

  if(props.key) {
    this.key = props.key
  }
  // 子元素个数
  var count = 0 
  children.forEach(function(child, index) {
    if(child instanceof Element) {
      count += child.count
    }else {
      children[index] = '' + child
    }
    count++
  })
  this.count = count
}

function createElement(tagName, props, children) {
  return new Element(tagName, props, children)
}

// 调用createElement 生成虚拟dom对象
var visualObj = createElement(
  'div', {id: 'visual-dom'},
  [
    createElement('ul', {id: 'list'}, [
      createElement('li', {class: 'list-item'}, ['item01']),
      createElement('li', {class: 'list-item'}, ['item02']),
      createElement('li', {class: 'list-item'}, ['item03'])
    ]),
    createElement('p', {class: 'content'}, ['visual dom'])
  ]
)
```

调用之后，查看`visual-dom`的数据结构

```javascript
{
  "tagName": "div",
  "props": {
    "id": "visual-dom"
  },
  "children": [{
    "tagName": "ul",
    "props": {
        "id": "list"
    },
    "children": [
      {
        "tagName": "li",
        "props": {
          "class": "list-item"
        },
        "children": [
          "item01"
        ],
        "count": 1
      },
      {
        "tagName": "li",
        "props": {
          "class": "list-item"
        },
        "children": [
          "item02"
        ],
        "count": 1
      },
      {
        "tagName": "li",
        "props": {
          "class": "list-item"
        },
        "children": [
          "item03"
        ],
        "count": 1
      }
    ],
      "count": 6
    },
    {
      "tagName": "p",
      "props": {
        "class": "content"
      },
      "children": [
        "visual dom"
      ],
      "count": 1
    }
  ],
  "count": 9
}
```

1. 使用`visual-dom`对象渲染真实DOM结构

现在虚拟DOM对象已经有了，接下来就是将虚拟DOM对象渲染到页面上

```javascript
// 虚拟DOM对象生成真实DOM
Element.prototype.render = function() {
  // 创建对应标签
  var el = document.createElement(this.tagName)
  // 获取对应标签节点属性
  var props = this.props
  // 设置节点的属性
  for(var item in props) {
    el.setAttribute(item, props[item])
  }
  // 获取子节点集合
  var children = this.children
  // 遍历子节点
  children.forEach(function(child) {
    // 子元素是非文本节点 继续递归render 是文本节点 创建文本dom节点
    var childEl = (child instanceof Element) ?
      child.render() : document.createTextNode(child)

    el.appendChild(childEl)
  })
  return el
}
```

最后将构建好的真实DOM渲染到页面中

```javascript
var realDom = visualObj.render()
document.body.appendChild(realDom)
```

页面展示跟上述真实节点是一致的，完整的[visual-dom转换真实dom](https://wangbaoqi.github.io/nateCase/visualDom/index.html)

### diff算法-比较虚拟Dom对象之间差异

在diff算法中，对比新老虚拟dom对象的差异，而这种对比的方式并不是遍历整个DOM树，造成大量的不必要的操作和性能损耗，DOM树的层级越多，性能的损耗会更严重，从时间复杂度来讲，一个层级为3层的DOM树，时间复杂度会达到`O(n^3)`。因此，鉴于对DOM的操作很少有跨层级的，对比的方式也就变成了在`同级`之间对比，时间复杂度也会降低`O(n)`

**1. 深度优先遍历，记录新老虚拟DOM的差异**

在这个过程中，代码的设计使用了`循环-递归`的方式,`循环`是每个节点都会有子节点（子元素），`递归`是在循环的过程中，对新老虚拟DOM的对应的子节点进行差异对比，在每次递归结束之后，将差异记录下来。

首先是对比的入口代码:

```javascript
/**
 * 虚拟dom树差异对比
 *  @param {Object} oldTree 现有的虚拟DOM树
 *  @param {Object} newTree 修改之后的虚拟DOM树
 *  @return {Object} patchs 返回两者间的差异
 **/
function diffTree(oldTree, newTree) {
  // 记录当前节点
  var index = 0;
  // 记录两者的差异对象
  var patchs = {};
  // 深度优先遍历 同层级节点进行对比
  diffWalk(oldNode, newNode, index, patchs);
  return patchs;
}
```

**2. 差异类型**

接下来对两颗虚拟DOM树进行深度优先遍历，对比同层级节点差异。 节点的修改以及变动基本可以归纳为几种情况，这里采用`枚举`来表示节点变动类型

```javascript
var patchEnum = {};
// 替换原来的节点 
patchEnum.REPLACE = 0
// 重新排序
patchEnum.REORDER = 1
// 修改子节点的属性
patchEnum.PROPS = 2
// 修改文本节点内容
patchEnum.TEXT = 3
```

**3. 循环递归记录每个节点的差异**

有了这些差异类型，在之后将差异更新到真实DOM的时候，就有了一个规则。接下来是具体实现深度优先遍历的操作

```javascript
function diffWalk(oldNode, newNode, index, patchs) {
  // 记录当前层级节点的差异
  var currentPatchs = [];
  // 对比文本节点
  if(typeof oldNode === 'string' && typeof newNode === 'string') {
    // 文本内容不一致，记录新节点的值
    if(oldNode !== newNode) {
      currentPatchs.push({
        type: patchEnum.TEXT,
        content: newNode
      })
    }
  }else if(newNode !== null && newNode[tagName] === oldNode[tageName] && oldNode[key] === newNode[key]) {
    // 同层级的节点类型一致 对比两者节点属性
    var patchProps = diffProps(oldNode, newNode);
    // 节点属性有差异，记录
    if(patchProps) {
      currentPatchs.push({
        type: patchEnum.PROPS,
        props: patchProps
      })
    }
    if(!isIgnoreChildren(newNode)) {
      diffChildren(oldNode, newNode, index, patchs, currentPatchs)
    }
  }else if(newNode !== null) {
    // 同层级节点类型不一致 代替老节点
    currentPatchs.push({
      type: patchEnum.REPLACE,
      node: newNode
    })
  }
  // 记录新老节点同一层级的差异
  if(currentPatch.length) {
    patchs[index] = currentPatch
  }
}
```

上述就是同层级节点进行差异对比，差异进行记录。接下来着重分析同层级节点相同的差异对比，这里除了节点属性对比之外，还采用递归 对子元素也进行同样的差异对比，下面看子元素差异对比的分析, 首先看节点属性之间差异对比

```javascript
// 对比节点的属性
// 主要有两点 1. 新节点属性有没有改变 2. 新节点有没有新增属性
function diffProps(oldNode, newNode) {
  // 记录属性有没有改变
  var count = 0;
  // 记录差异节点对象
  var patchProps = {};
  var oldProps = oldNode.props;
  var newProps = newNode.props;

  // 遍历老节点属性跟新节点属性，不一致的属性进行记录
  for(var props in oldProps) {
    if(oldProps[props] !== newProps[props]) {
      count++;
      patchProps[props] = newProps[props];
    }
  }
  // 遍历新节点属性跟老节点属性，有没有新增的属性
  for(var props in newProps) {
    if(!oldProps.hasOwnProperty(props)) {
      count++;
      patchProps[props] = newProps[props]
    }
  }

  if(!count) return null
  return patchProps
}
```

然后再来看下子元素的差异对比, 列表对比算法 假如旧的节点顺序： a, b, c, d 新的节点顺序: b, a, d, e, f

可以看到新的节点中，有新增的元素，删除旧的元素，旧的元素顺序改变了

求最小的插入、删除操作。这个问题抽象成字符串的最小编辑距离Edition Distance，通过动态规划求解，时间复杂度O\(n\)=O\(m\*n\), 但不是最小的操作，可以优化一些简单的移动情况，牺牲DOM操作，让时间复杂度达到线性的O\(Max\(m,n\)\), 具体的算法实现

```javascript
function diffChildren(oldNode, newNode, index, patchs, currentPatchs) {
  var oldChildren = oldNode.children;
  var newChildren = newNode.children;
  // 获取子节点最小更改量 
  var listDiff = listDiff(oldNode, newNode, 'key')
  // 子节点中顺序发生变化 记录差异
  if(listDiff.moves.length) {
    var recoderPatch = { type: patchEnum.REORDER, moves: diffs.moves }
    currentPatch.push(recoderPatch)
  }

  var leftNode = null
  var currentNodeIndex = index

  // 子节点递归寻找差异
  oldChildren.forEach((child, index) => {
    var newChild = newChildren[index]
    currentNodeIndex = (leftNode && leftNode.count) 
      ? currentNodeIndex + leftNode.count + 1
      : currentNodeIndex + 1
    diffWalk(child, newChild, currentNodeIndex, patchs)
    leftNode = child
  })
}
```

至此，新老节点所有的差异都已经记录完成了, 接下来就是将这些差异应用到真实DOM上 

### diff算法-把patchs应用到真实DOM上

将patchs应用到真实的DOM上，也要对旧的DOM进行深度优先遍历，遍历中将patchs作用到DOM上

patchs入口脚本

```javascript
function patch(node, patches) {
  var walker = {index: 0}
  dsfWalk(node, walker, patches)
}

function dsfWalk(node, walker, patches) {
  // 获取当前节点的差异
  var currentPatches = patches[walker.index]
  var len = node.childNodes ? node.childNodes.length : 0

  for(var i = 0; i < len; i++) {
    var child = node.childNodes[i]
    walker.index ++
    dsfWalk(child, walker, patches)
  }

  if(currentPatches) {
    // 对当前节点进行DOM操作
    applyPatches(node, currentPatches)
  }
}
```

对每一个节点进行递归深度遍历，如果patches中有差异，则对当前节点进行DOM操作

```javascript
function applyPatches(node, currentPatches) {
  currentPatches.forEach(currentPatche => {
    switch(currentPatche.type) {
      case patchEnum.REPLACE:
        var newNode = (typeof currentPatch.node === 'string')
          ? document.createTextNode(currentPatch.node)
          : currentPatch.node.render()
        node.parentNode.replaceChild(newNode, node)
        break
      case patchEnum.REORDER:
        reorderChildren(node, currentPatch.moves)
        break
      case patchEnum.PROPS:
        setProps(node, currentPatch.props)
        break
      case patchEnum.TEXT:
        node.textContent = currentPatch.content
        break
      default:
        throw new Error('Unknown patch type ' + currentPatch.type)
    }
  })
}
```

对当前节点中的差异进行DOMrender，接下来看属性的操作

```javascript
function setProps(node, props) {
  for (var key in props) {
    if (!props[key]) {
      node.removeAttribute(key)
    } else {
      var value = props[key]
      setAttr(node, key, value)
    }
  }
}

function setAttr(node, key, value) {
  switch (key) {
    case 'style':
      node.style.cssText = value
      break
    case 'value':
      var tagName = node.tagName || ''
      tagName = tagName.toLowerCase()
      if (
        tagName === 'input' || tagName === 'textarea'
      ) {
        node.value = value
      } else {
        // if it is not a input or textarea, use `setAttribute` to set
        node.setAttribute(key, value)
      }
      break
    default:
      node.setAttribute(key, value)
      break
  }
}
```

最后就是对节点的重排操作了

```javascript
function reorderChildren(node, moves) {
  var staticNodeList = Array.from(node.childNodes);
  var maps = {}

  staticNodeList.forEach(node => {
    // 如果是一个元素节点
    if (node.nodeType === 1) {
      var key = node.getAttribute('key')
      if (key) {
        maps[key] = node
      }
    }
  })

  moves.forEach(move =>{
    var index = move.index
    if (move.type === 0) {
      // type为 0，表示新的dom对象已经删除该节点
      if (staticNodeList[index] === node.childNodes[index]) { // maybe have been removed for inserting
        node.removeChild(node.childNodes[index])
      }
      staticNodeList.splice(index, 1)
    } else if (move.type === 1) {
      // type为 1，表示新的dom对象插入该节点 
      var insertNode = maps[move.item.key]
        ? maps[move.item.key].cloneNode(true) // reuse old item
        : (typeof move.item === 'object')
            ? move.item.render()
            : document.createTextNode(move.item)
      staticNodeList.splice(index, 0, insertNode)
      node.insertBefore(insertNode, node.childNodes[index] || null)
    }
  })
}
```

完整的实现查看[这里](https://wangbaoqi.github.io/nateCase/visualDom/index.html)

