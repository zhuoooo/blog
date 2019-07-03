# DOM事件类

**问题（从上之下，逐步深入）**：

- 基本概念：DOM事件的级别
- DOM事件模型
- DOM事件流
- 描述DOM事件捕获的具体流程
- Event对象的常见应用
- 自定义事件

------

### 基本概念--DOM事件的级别

| DOM事件类 | 事件级别                                                |
| --------- | ------------------------------------------------------- |
| DOM0      | element.onclick = function() {}                         |
| DOM2      | element.addEventListener('click', function() {}, false) |
| DOM3      | element.addEventListener('keyup', function() {}, false) |

> DOM3 中增加了许多事件，如`keyup` , `keydown`, `keypress`....



### DOM事件模型

捕获机制与冒泡机制



### DOM事件流

一个完整的事件流分三个阶段：

- 第一阶段：捕获
- 第二阶段：目标阶段
- 第三阶段：冒泡



### 描述DOM事件捕获的具体流程

- 捕获的具体流程：window -> document -> html -> body -> .... -> 目标元素
- 冒泡的具体流程：目标元素 -> ... -> body -> html -> document -> window



### Event对象的常见应用

```js
event.preventDefault();
event.stopPropagation();
event.stopImmediatePropagation(); // 当在一个DOM对象上绑定了A,B两个事件时，你希望点击它时，
							    // 执行A，不执行B；则在A中添加这个代码
event.currentTarget(); // 当前绑定事件的对象
event.target();
```



### 自定义事件

```js
var eve = new Event('hello');
dom.addEventListener('hello', function(){
    console.log('Hello, world');
})
dom.dispatchEvent(eve);
```

