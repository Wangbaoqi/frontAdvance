# 前端进阶修炼

 ![github](https://img.shields.io/github/stars/Wangbaoqi/frontAdvance?affiliations=OWNER&style=social)![Live Demo](https://img.shields.io/badge/ecma262-green)

![](.gitbook/assets/logo.gif)

这里是Nate前端进阶修炼得主战场，对前端进行重新学习和整理，建立自己完整的知识体系，将杂乱的知识通过方法论（整理法和追溯法）规整到对应的体系中。

建立自己的一套前端技术知识脑图，对不了解、不懂的知识进行查漏补缺。

### HTML 知识脑图

建立HTML知识脑图，以追本溯源的方式，大家都知道`HTML`是**超文本标记语言**，但是有谁知道`markup language`-标记语言呢？

可以从下图看到，对`HTML`脑图的划分是以**维度**的方式来划分的，每一门计算机语言都会有`语法`和`词法`的。 之后又以`SGML`和`XML`的维度来进一步了解`HTML`, 那`SGML`、`XML`和`HTML`之间又是什么联系呢？

![html mind](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/index/htmlModel.png)

**标记语言**: 特指用约定好的一系列标记对电子文档进行标记，来实现电子文档的语义、结构和格式的定义，定义什么样的标记是允许的，什么样的标记是必须的

**SGML**: \(Standard Generilized MarkUp Language\), 标准通用标记语言，是描述电子文档标记的国际标准。SGML通过标记来描述文档结构，以便存储、提取、处理文档中的数据。 准确的来说，SGML是一种元语言-可以衍生出其他的语言（XML/HTML）

**元素**: \(Element\) 是SGML/XML中具有一定结构的文字片段，是组成文档的基本的单位，不同的元素类型具有不同的含义

**实体**: \(Entities\) 是一个命名了的标记数据块，可以是一个字符串，也可以是一个文件，其形式为" "，[更多Entities](https://www.w3.org/TR/WD-html40-970708/sgml/entities.html)

**DTD**: \(Document Type Definition\)，文档类型定义，SGML引入**DTD**，表示不同的文档可能有不同的结构类型，不同的文档类型，遵循对应DTD规范，而在HTML中，DTD的规范被作为标准固定了下来。HTML就是SGML衍生出来的一个**应用**，因此，不能作为元语言来定义其他的语言。**在HTML4**以及之前的版本，HTML文档中都要指定DTD,而在**HTML5**中，则不需要去指定DTD，HTML5有了自己的一套解析规则，不再以SGML为基础

**XML**: \(Extensible MarkUp Language\), 可扩展标记语言，XML作为SGML的一个子集，它也有自己的DTD标准。

### CSS 知识脑图

重新建立一份关于CSS的知识脑图，CSS在大部分开发者中是处于比较低的位置的，因此，为了能更好的掌握前端，CSS是必不可少的一环。

![css mind](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/index/css_model.png)

### JavaScript 知识脑图

JavaScript 在前端中处在不可缺少的地位，其重要性相当于一栋建筑的地基。 JavaScript 在维度方面可以分为`语法`, `语义`和`运行时`，任何计算机语言都是用规定的语法去表达对应的语义，最终操作运行时的过程。

![JS mind](https://raw.githubusercontent.com/Wangbaoqi/blogImgs/master/nateImgs/index/JavaScript.png)

