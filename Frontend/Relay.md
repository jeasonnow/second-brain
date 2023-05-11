## 前言：GraphQL 的作用
![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111530111.png)

在传统的 `React` 开发中，我们通常使用组件互相组合的形式完成页面元素的描述，而页面组件中所需的数据我们都需要通过接口在服务端去获取，但是这么做却可能导致页面上的问题：

![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111524014.png)

![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111525343.png)
1. **UnderFetching**: 前后端修改不同步导致的依赖服务数据缺失，从而导致页面上的数据展示问题。
2. **OverFetching**: 前后端修改不同步导致某些数据不再依赖，但是服务端数据依然返回导致的数据冗余问题。

`GraphQL` 通过提供灵活的语义化查询语言，作为前端后后端数据库间的中间层，能够做到前端需要多少数据就仅仅从数据库中拉取多少数据，避免了前端在修改页面组件树时依赖变动而服务端也需要同步更改的问题。且能尽量少的减少传输给前端的数据，一切仅根据需要来决定。