![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202304271612522.png)
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
2. 如何获取更新