1. 每个模块都会进行的处理。
   - 检测模块的父子依赖，并记录。
   - 提供各种 hot API 用来管理更新关系 
     ```javascript
// 接收自己更新，更新后重复执行自己，不往上冒泡

module.hot.accept();

  

// 接收依赖更新，更新后执行回调函数，不往上冒泡

module.hot.accept(['dep1'], () => {

console.log('dep1 changed');

});

  

// 让自己失效，往上冒泡

// 通常在 accept 之后，遇到一些场景又希望自己失效时调用

module.hot.invalidate();

  

// 标记一些依赖为不可更新，这些依赖的更新会触发页面 reload

module.hot.decline(['dep1']);

// 同上，标记自己为不可更新

module.hot.decline();

  

// 设置或移除当前模块被自动替换时执行的回调函数

module.hot.dispose(fn);

module.hot.removeDisposeHandler(fn);
```
 - 提供 check 和 apply 方法
2. 怎么获取更新？先 check 再 apply 。check 就是检查更新并下载更新的 module 和 chunk。而 apply 则会走如下流程：
   - 将所有更新的模块状态更新为不可用
   - 每个模块自己检测自己和父模块是否有 accept handler，没有的话会刷新，有的话会一直向上冒泡到最初的 accept handler 模块为止。
   - dispose 并 upload 所有不可用的模块。
   - 执行所有的 Accept Handler
3. 上图是 webpack 配合 webpack-dev-server 进行应用开发的模块热更新流程图。
   ![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202304271753341.png)


-   上图底部红色框内是服务端，而上面的橙色框是浏览器端。
-   绿色的方框是 webpack 代码控制的区域。蓝色方框是 webpack-dev-server 代码控制的区域，洋红色的方框是文件系统，文件修改后的变化就发生在这，而青色的方框是应用本身。

上图显示了我们修改代码到模块热更新完成的一个周期，通过深绿色的阿拉伯数字符号已经将 HMR 的整个过程标识了出来。

1.  第一步，在 webpack 的 watch 模式下，文件系统中某一个文件发生修改，webpack 监听到文件变化，根据配置文件对模块重新编译打包，并将打包后的代码通过简单的 JavaScript 对象保存在内存中。
2.  第二步是 webpack-dev-server 和 webpack 之间的接口交互，而在这一步，主要是 dev-server 的中间件 webpack-dev-middleware 和 webpack 之间的交互，webpack-dev-middleware 调用 webpack 暴露的 API对代码变化进行监控，并且告诉 webpack，将代码打包到内存中。
3.  第三步是 webpack-dev-server 对文件变化的一个监控，这一步不同于第一步，并不是监控代码变化重新打包。当我们在配置文件中配置了[devServer.watchContentBase](https://link.zhihu.com/?target=https%3A//webpack.js.org/configuration/dev-server/%23devserver-watchcontentbase) 为 true 的时候，Server 会监听这些配置文件夹中静态文件的变化，变化后会通知浏览器端对应用进行 live reload。注意，这儿是浏览器刷新，和 HMR 是两个概念。
4.  第四步也是 webpack-dev-server 代码的工作，该步骤主要是通过 [sockjs](https://link.zhihu.com/?target=https%3A//github.com/sockjs/sockjs-client)（webpack-dev-server 的依赖）在浏览器端和服务端之间建立一个 websocket 长连接，将 webpack 编译打包的各个阶段的状态信息告知浏览器端，同时也包括第三步中 Server 监听静态文件变化的信息。浏览器端根据这些 socket 消息进行不同的操作。当然服务端传递的最主要信息还是新模块的 hash 值，后面的步骤根据这一 hash 值来进行模块热替换。
5.  webpack-dev-server/client 端并不能够请求更新的代码，也不会执行热更模块操作，而把这些工作又交回给了 webpack，webpack/hot/dev-server 的工作就是根据 webpack-dev-server/client 传给它的信息以及 dev-server 的配置决定是刷新浏览器呢还是进行模块热更新。当然如果仅仅是刷新浏览器，也就没有后面那些步骤了。
6.  HotModuleReplacement.runtime 是客户端 HMR 的中枢，它接收到上一步传递给他的新模块的 hash 值，它通过 JsonpMainTemplate.runtime 向 server 端发送 Ajax 请求，服务端返回一个 json，该 json 包含了所有要更新的模块的 hash 值，获取到更新列表后，该模块再次通过 jsonp 请求，获取到最新的模块代码。这就是上图中 7、8、9 步骤。
7.  而第 10 步是决定 HMR 成功与否的关键步骤，在该步骤中，HotModulePlugin 将会对新旧模块进行对比，决定是否更新模块，在决定更新模块后，检查模块之间的依赖关系，更新模块的同时更新模块间的依赖引用。
8.  最后一步，当 HMR 失败后，回退到 live reload 操作，也就是进行浏览器刷新来获取最新打包代码。