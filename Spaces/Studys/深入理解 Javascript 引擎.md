##  Javascript 引擎
`Javascript` 是一门解释行语言，浏览器并不能理解 javascript 语言，其只关心执行的机器码，而从 `Javascript` 到机器码，这期间需要的工具就是 `Javascript` 引擎。

**有哪些编译器用来解释 `Javascript` 呢**
1. `v8`
2. `spiderMonkey`
3. `JavascriptCore`

## 引擎的共同之处

### 引擎都遵守 `ECMAScript` 标准
通过同样的标准，保证了在不同浏览器使用不同的编译器产生的产物都能表现出相同的现象。

### 引擎的工作流程
![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305091547035.png)
