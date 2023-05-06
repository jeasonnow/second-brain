---
aliases: 
date created: 2023-05-06
date modified: 2023-05-06 09:26:42
tags: 
---
up:: [[React]]  

> [!INFO] 提示
>  提示内容

## Fiber 的意义
在 React 15 前，使用递归树的方式进行同步的更新整颗 Vitural Dom，这在树结构变得复杂时将指数级提升等待渲染时间，所以在 React 16+ 中引入了 Fiber 的概念，通过 Fiber 类似链表的结构可以随时中断、开始渲染进程。

## React 16 的 Render 流程概览
在 16 中，存在两种 render 模式，一种同步（sync），一种可中断，当使用可中断模式时会有一个判断条件判断是否不能继续进行 render，而同步则无这个限制。
```javascript
// performSyncWorkOnRoot会调用该方法
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

// performConcurrentWorkOnRoot会调用该方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```
而 performUnitOfWork 这个函数正是把原来的递归树转换为可中断的

---

## 参考文献
