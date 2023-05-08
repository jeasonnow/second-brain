---
aliases: WeakMap妙用
date created: 2023-05-08
date modified: 2023-05-08 17:46:35
tags: 
---
up:: [[javascript]]  

## 正文
使用 WeakMap 的特性：weakMap 的 key 值是弱引用的，当其作为 key 的元素被销毁，则其 value 也会被销毁，可以考虑将 dom 节点作为 key，而 dom 所相关的事情作为 value，这能更好的做到内存清理。


---

## 参考文献
[Why I Like Using Maps (and WeakMaps) for Handling DOM Nodes | Alex MacArthur](https://www.macarthur.me/posts/maps-for-dom-nodes)