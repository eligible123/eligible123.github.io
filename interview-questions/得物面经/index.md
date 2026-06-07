---
title: 得物面经
date: 2026-06-07
type: page
---

# 得物面经

> 共 1 道面试题

## 1. 前端得物一面

🟢 **简单** | 标签：JavaScript, 前端, 得物, 面经, 校招

# 得物前端一面高频考点总结

## 一、JavaScript 核心

**1. 闭包**
函数能够访问其词法作用域中变量的机制。常见应用：数据私有化、函数柯里化。注意内存泄漏风险，需及时释放引用。

**2. 原型链**
每个对象都有 `__proto__` 指向构造函数的 `prototype`，形成链式结构，实现属性和方法的继承。`instanceof` 就是沿原型链查找。

**3. 事件循环（Event Loop）**
同步代码进入调用栈，异步任务分为宏任务（setTimeout、I/O）和微任务（Promise.then、MutationObserver）。每轮宏任务执行完后清空微任务队列。

**4. this 指向**
- 普通函数：调用时决定（谁调用指向谁）
- 箭头函数：定义时继承外层 this
- call/apply/bind 可显式绑定

**5. Promise**
解决回调地狱，三种状态：pending/fulfilled/rejected。`Promise.all` 并发执行，`Promise.race` 取最先完成的结果。async/await 是其语法糖。

## 二、CSS 基础

- **BFC**：块级格式化上下文，解决外边距重叠、清除浮动
- **Flex 布局**：`justify-content`（主轴）、`align-items`（交叉轴）
- **盒模型**：`box-sizing: border-box` 让宽高包含 padding 和 border

## 三、网络与浏览器

- **HTTP 缓存**：强缓存（Cache-Control）、协商缓存（ETag/Last-Modified）
- **跨域**：CORS、JSONP、代理服务器
- **TCP 三次握手**：确保双方收发能力正常

## 四、框架基础

**Vue 响应式原理**：Vue2 用 `Object.defineProperty`，Vue3 用 `Proxy`。Vue3 还引入了 Composition API，逻辑复用更灵活。

**Vue 和 React 区别**：Vue 模板驱动、React JSX 书写；Vue 双向绑定、React 单向数据流；Vue 组合式 API 对标 React Hooks。

## 五、手写题高频

- 防抖/节流
- 深拷贝
- 数组扁平化
- 继承实现（寄生组合式继承）
- Promise 基础实现

> **面试技巧**：回答时先说结论，再展开解释，适当举例。遇到不确定的题目，先说思路，展示解决问题的能力。

---

