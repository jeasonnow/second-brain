---
title: 深入理解 Javascript 引擎
date created: 2023-05-09
date modified: 2023-05-11
---

## Javascript 引擎

`Javascript` 是一门解释行语言，浏览器并不能理解 javascript 语言，其只关心执行的机器码，而从 `Javascript` 到机器码，这期间需要的工具就是 `Javascript` 引擎。 ^c42e17

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

#### 内联 (避免重复的引用跳转)

相比函数调用，直接使用函数的内容替代函数调用会减少昂贵的引用消耗，所以优化编译器会将部分循环或者函数主体内的函数调用替换成被调用函数主体本身。

```javascript
// before
const add = (a, b) => {  
    return a + b;  
};

for (let i = 0; i < 100000; i++) {  
    add(i, i * 2);  
}

//optimized
for (let i = 0; i < 100000; i++) {  
    i + i * 2;
}
```

#### 循环内不可变元素提升（避免重复不变的计算）

针对函数内不会发生改变的计算量，将其从循环中提升到外部然后直接调用，避免在循环时重复调用，优化机器码运行速度。

```javascript
// before
let result = 0;
for (let i = 0; i < 1000; i++) {
	result += 24 * 60 * 60 * 1000; 
}

// optimized
let result = 0;
let count = 24 * 60 * 60 * 1000;
for (let i = 0; i < 1000; i++) {
	result += count; 
}

```

### 消除重复表达式定义（单一原则）

避免在源代码中出现的重复表达式，将其抽象成一个变量到处调用。

```javascript
// before
let foo = (a * b) + 1;
let bar = (a * b) * (a * b);

// optimized
let temp = a * b;
let foo = temp + 1;
let bar = temp * temp;
```

#### Tree Shaking (消除无用代码)

避免无用代码被编译，减少编译和优化量以及执行代码所需的内存。

#### 减少源代码中的昂贵操作

相比对每次都重新替换连续内存中的元素或引用，替换成对原有内存指针中的元素进行改变，会降低消耗。

```javascript
// before
let x = 7;  
let arr = [];for (let i = 0; i < 10; i++) {  
    arr[i] = x * i;  
}

// optimized
let x = 7;  
let s = 0;  
let arr = [];for (let i = 0; i < 10; i++) {  
    arr[i] = s;  
    s += x;  
}
```

#### 内置函数优化

针对一些在引擎内实现的 ECMA 基类，其实现本身其实已经可以确定，不需要重复执行一些标准函数，相应的只需要替换其内置的某些函数为被调用函数的函数体（类似 [[#内联 (避免重复的引用跳转)]]）

```javascript
// before
const someMethod = () => {  
    let callbackfn = x => x === true;  
    let len = this.length;  
    if (typeof callbackfn !== "function") {  
        throw new TypeError();  
    }  
    let k = 0;  
      
    while (k < len) {  
        if (k in this) {  
            if (callbackfn(this[k])) {  
                return true;  
            }  
        }  
        k = k + 1;  
    }    return false;  
}

//optimized
const someMethod = () => {  
    let len = this.length;  
    let k = 0;  
      
    while (k < len) {  
        if (this[k] === true) {  
            return true;  
        }  
        k = k + 1;  
    }    return false;  
}
```

## 如何在编码时减少引擎的优化工作

#### Hot Function

在引擎工作时，在调用参数全部趋同时，引擎会进行代码的优化，但是如果调用的参数在后续发生了变化，为了避免前面的优化影响到后续的函数调用，其会回退优化，所以尽量保证一个函数的调用使用的参数都是相同的，不同的参数使用不同的函数代替同名函数。

#### Shapes

对象的创建涉及许多的 property，在一个工程里，对象会有很多，如果来一个就重新创建且初始化会造成相当多的性能消耗，所以针对对象各个引擎都实现了公用结构体的实现，结构体相同的对象会公用同一个结构体，擅自去改变结构体的结构会导致引擎需要新建新的 Shape 来适应，所以尽量减少变动对象结构且尽量保证项目中对象的结构体一致，达到优化生成新对象的作用。  
![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305091731368.png)  
![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305091732698.png)

#### 文档优化建议速记

- 不要创建需要大量可选属性的函数。尽量使你的对象形状和参数类型保持一致。
- 尽量在对象初始化时设置你的属性。
- 如果你不能在对象的初始化中设置你的属性，那么就按同样的顺序添加你的属性。
- 尽量使你的函数具有单态性。尽量让你的思维像写静态类型语言一样（使用 Typescript 可以帮助你）。

## 文章来源

[Deep Dive Into Javascript Engines — Blazingly Fast ⚡️ | by Doğukan Akkaya | Medium](https://medium.com/@dogukanakkaya/deep-dive-into-javascript-engines-blazingly-fast-%EF%B8%8F-fc47069e97a4)
