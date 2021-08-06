# 性能相关篇

## 事件委托

事件委托常使用的场景是列表点击，将每一个元素的点击的事件委托到其父元素，其实现的原理是基于事件冒泡完成的

```javascript
/**
 * el 委托的元素
 * type 事件类型
 * selector 选择的元素
 * fn 回调函数
 **/

const bindEvent = (el, type, selector, fn) => { 
  if(fn == void 0) {
    fn = selector;
    selector = null;
  }
  
  el.addEventListener(type, (evt) => {
    const target = evt.target;
    // 事件委托
    if(selector && el.matches(selector)) {
      fn.call(target, evt)
    }
    // 普通监听事件
    if(!selector) {
     fn.call(target, evt)
    }
  })
}

```

## 节流防抖

## 图片懒加载

## Sleep/Delay 实现

## setTimeout实现setInterval



