---
title: react tutorial-tic tac toe
author: Yu Mengchi
categories:
  - JS
tags:
  - React
---
  
App.js中的代码如下
```java
export default function Square() {
  return <button className = "square">X</button>;
}
```
上面的代码定义了一个componnet，component的名字叫Square，component是用来render、manage和更新UI元素的。
第一行定义了一个function叫Square。export是一个JavaScript的关键字，表示这个function可以被其他文件访问，相当于java中的public关键字。
default关键字表示这是一个main function。
第二行返回了一个button，return关键字表示这个function的返回值。<button>是一个JSX元素。一个JSX元素包含JavaScript代码和HTML tag。
className="square"表示一个button的属性，告诉CSS如何展示这个button。X表示展示在button里面的文本。
index.js文件里的内容相当于给App.js文件中的内容和浏览器之间做桥梁的工作。比如index.js中的内容可能是下面的样式：
```java
import {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';

import App from './App';
```


1.board和square之间怎么传递属性，
2.square中发生点击事件后怎么让board中的属性发生改变
3.怎么在square中调用board中的方法
4.怎么把board中的方法作为参数传给square

board中有9个square，square中发生点击事件后，会调用一个方法，这个方法是从board传给square的，这个方法会改变数组的值，把第i个square的值设置为"X"，
然后board中会时刻检测数组，检查9个square的值，判断9个值是否构成成功条件





















