---
title: 可控与不可控组件
date: 2018-05-22 09:53:11
categories:
- JavaScript
tags:
- React
---

#### 可控组件

```html
<input
  type="text"
  defaultValue={this.state.inputVal}
/>
```

- 上段代码中的defaultValue是通过state获取到的，如果state发生了改变，input的值也会随之进行改变。

---

#### 不可控组件

```html
<input
  type="text"
  defaultValue="Hello world!"
/>
```

- 上段代码中defaultValue的值是固定的，如果要获取input的value值，只有使用ref获取节点来获取值。

---

#### 可控与不可控的组件区别是什么？可控组件有什么优点？

- **核心区别：** 值与state是否是对应。
- **优点：**
    1. 符合React数据流。
    2. 数据在state中，修改跟使用更加方便，只需要对state进行修改，而不用对DOM操作。

---

#### 示例

{% jsfiddle devin_yuan/6quee3x8 js,result %}
