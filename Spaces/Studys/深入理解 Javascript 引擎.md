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
源码 - parser -> 语法数 - interpreter -> ByteCode 且生成优化机器码。

所有引擎都遵循这个大体结构，通过解释器解释 js 的抽象语法树，如果有可以进行优化的部分则继续使用优化编译器针对性的进行优化，最后生成字节码和优化过的机器码。共同产物提供给机器进行执行。

### 大同小异的编译器
#### V 8 (Chorme/NodeJs)
- 解释器：`Ignition`
- 优化编译器：`TurboFan`

#### JSC (Safari/Ios)
不同的解释器、优化器在针对不同复杂度的代码时生效。
- 解释器（`LLInt` / `Baseline JIT`）
- 优化编译器（`DFG` / `FTL`）

#### SpiderMonkey
- 解释器：`BaseLink Interpreter`
- 优化编译器：`Baseline Compiler` 、 `WarpMonkey`


### 优化编译器做了什么
#### 内联
相比函数调用，直接使用函数的内容替代函数调用会减少昂贵的引用消耗，所以优化编译器会将部分循环或者函数主体内的函数调用替换成被调用函数主体本身。
```javascript

```