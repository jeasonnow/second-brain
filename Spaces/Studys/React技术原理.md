---
aliases: 
date created: 2023-05-06
date modified: 2023-05-06
tags:
  - 
title: React技术原理
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

而 performUnitOfWork 这个函数正是把原来的递归树转换为可中断的循环遍历，向上回归的两个过程，且其分为多步来完成，可在任意环节中断：

1. 首先从 `rootFiber` 开始向下深度优先遍历。为遍历到的每个 `Fiber节点` 调用 [beginWork方法 (opens new window)](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L3058)。
2. 当达到叶子节点时，归阶段开始，先调用当前节点的 completeWork，如果没有 siblings ，则其父节点也开始 completeWork，如果有则执行兄弟节点的 beginWork 并进行递操作，当最后所有的节点都完成遍历的递阶段，则会往上层完成归阶段直至 rootFiber 也完成归操作。

## BeginWork 工作细节

![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305061112608.png)

### Mount

当不存在可复用节点时，则直接根据类型创建新的 FiberNode

### Update

根据 type 和 Props 等判断条件判断是否可以直接复用子节点。

### Reconciler

使用协调器对更新做处理，通过 diff 算法和是否 mount，来生成真正 Fiber 节点。

### EffectTag

判断每个 Fiber Node 需要做的真实 Dom 操作，附带到每个 Fiber 节点上。

## CompleteWork

![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305061458035.png)

## Diff 算法
为了减少遍历树的昂贵消费（O(n^3)），如果是跨层级的更新，React 默认不复用 Fiber，而只关注同级的更新。

### 单节点 Diff
通常只需要对比 dom 类型和 key ，不一致时则不能复用，一致时则复用。

### 多节点 Diff



---

## 参考文献
