## 前言：GraphQL 的作用
![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111530111.png)

在传统的 `React` 开发中，我们通常使用组件互相组合的形式完成页面元素的描述，而页面组件中所需的数据我们都需要通过接口在服务端去获取，但是这么做却可能导致页面上的问题：

![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111524014.png)

![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111525343.png)
1. **UnderFetching**: 前后端修改不同步导致的依赖服务数据缺失，从而导致页面上的数据展示问题。
2. **OverFetching**: 前后端修改不同步导致某些数据不再依赖，但是服务端数据依然返回导致的数据冗余问题。

`GraphQL` 通过提供灵活的语义化查询语言，作为前端后后端数据库间的中间层，能够做到前端需要多少数据就仅仅从数据库中拉取多少数据，避免了前端在修改页面组件树时依赖变动而服务端也需要同步更改的问题。且能尽量少的减少传输给前端的数据，一切仅根据需要来决定。

## Relay 心智模型
 GraphQL 在请求的层次将大部分的冗余数据、请求从前、后端移除，从而减少了同步需求，避免了许多的问题的同时提高了性能，但是 Relay 在其基础上更进一步，简单点理解，Relay 的心智模型如下：
 
1. 作为 `GraphQL` 和前端  `Client` 之间的中间缓存层，将部分可复用的请求内容进行缓存，在 `Client` 端发送请求后检查缓存，没有缓存则发送请求获取 `state` 进行缓存，如果有则使用缓存。
   ![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111609599.png)

2. 提供优先级策略，让部分请求可以做到优先获取优先渲染，其他优先级低的可以等主要内容完成渲染后进行更新渲染。
   ![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305111609476.png)
## Relay 的处理细节
当完成初次渲染之后，当用户在 Client 端产生交互，Relay 会完成一套复杂的流程，用来保证请求的范围尽量小：
- 发送请求时创建 `Mutation` 交予 `Mutation System` 处理，获得最小、完全的 State 变化，交予 `State` 管理里进行 `State` 初始化。
- `State` 完成初始化之后，通知变动处的 `UI` 先进行初始化， `Mutation System` 会将变动范围涉及的请求进行合并交予 `GraphQL` 进行请求。
- `GraphQL` 请求完成之后将 `Payload` 返回给 `Relay State` 进行更新，并广播给 UI 进行更新。