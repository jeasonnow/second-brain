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
两轮遍历，第一轮找到更新节点，第二轮找到非更新操作的节点。
#### 第一轮
1.  `let i = 0`，遍历 `newChildren`，将 `newChildren[i]` 与 `oldFiber` 比较，判断 `DOM节点` 是否可复用。
    
2.  如果可复用，`i++`，继续比较`newChildren[i]`与`oldFiber.sibling`，可以复用则继续遍历。
    
3.  如果不可复用，分两种情况：
    

-   `key`不同导致不可复用，立即跳出整个遍历，**第一轮遍历结束。**
    
-   `key`相同`type`不同导致不可复用，会将`oldFiber`标记为`DELETION`，并继续遍历
    

4.  如果 `newChildren` 遍历完（即 `i === newChildren.length - 1`）或者 `oldFiber` 遍历完（即 `oldFiber.sibling === null`），跳出遍历

#### 第二轮
##### NewChilder 遍历完但是 OldFiber 未遍历完
说明删除了节点，将 Oldfiber 剩下的节点都标记为删除

##### NewChildren 未遍历完但是 OldFiber 遍历完
说明新增了节点，将 NewChildren 的接下来的节点标记为插入

##### NewChildren 和 OldChildren 都遍历完
说明一遍遍历完后知道了所有的更新节点。

##### NewChildren 和 OldChildren 都没有遍历完
通过 Demo 查看
```
// 之前
abcd

// 之后
acdb

===第一轮遍历开始===
a（之后）vs a（之前）  
key不变，可复用
此时 a 对应的oldFiber（之前的a）在之前的数组（abcd）中索引为0
所以 lastPlacedIndex = 0;

继续第一轮遍历…

c（之后）vs b（之前）  
key改变，不能复用，跳出第一轮遍历
此时 lastPlacedIndex === 0;
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === cdb，没用完，不需要执行删除旧节点
oldFiber === bcd，没用完，不需要执行插入新节点

将剩余oldFiber（bcd）保存为map

// 当前oldFiber：bcd
// 当前newChildren：cdb

继续遍历剩余newChildren

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index;
此时 oldIndex === 2;  // 之前节点为 abcd，所以c.index === 2
比较 oldIndex 与 lastPlacedIndex;

如果 oldIndex >= lastPlacedIndex 代表该可复用节点不需要移动
并将 lastPlacedIndex = oldIndex;
如果 oldIndex < lastplacedIndex 该可复用节点之前插入的位置索引小于这次更新需要插入的位置索引，代表该节点需要向右移动

在例子中，oldIndex 2 > lastPlacedIndex 0，
则 lastPlacedIndex = 2;
c节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：bd
// 当前newChildren：db

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
oldIndex 3 > lastPlacedIndex 2 // 之前节点为 abcd，所以d.index === 3
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：b
// 当前newChildren：b

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index;
oldIndex 1 < lastPlacedIndex 3 // 之前节点为 abcd，所以b.index === 1
则 b节点需要向右移动
===第二轮遍历结束===

最终acd 3个节点都没有移动，b节点被标记为移动
```

---

## 参考文献
