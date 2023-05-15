---
title: Think Qwik
date created: 2023-05-10
date modified: 2023-05-10
---

## 什么是 Qwik

现在的前端应用大多的性能瓶颈仍然在于大 `bundle` 的下载和执行（也就是所谓的 `hydration`）流程， `Qwik` 是一个挑战解决这种方式的框架，其目标是减少、延迟初次加载应用时下载、执行的 `js` 量，从而获得极快的启动速度。

它的提升性能的简单心智模型如下：

1. 尽量原子化组件、事件为更小的 `chunk`，加载时尽量少、尽量延迟的去下载执行这些 `chunk`，让首屏性能获得极大的提升。
2. 在服务端生成响应 `html` 时，尽量将已执行的 `client` 端 `js` 的状态序列化到 `html` 中一并返回，通过框架层处理让他恢复执行状态，而不是重新执行 js，即用 `Resumable` 代替 `hydration`

## Qwik 如何做到延迟执行

我们可以从一个 `Qwik` 编译的示例来看这个过程

```javascript
// app.js
import { component$ } from '@builder.io/qwik';

export const App = component$(() => {
  console.log('render');
  return <p onClick$={() => console.log('hello')}>Hello Qwik</p>;
});
```

这里的 `$` 就是一个特殊标识，它意味着这就是一个拆离 `chunk` 的关键点，而这个例子中能拆出去的就是 `App` 组件和其中的点击事件。

```javascript
// app.js
import { componentQrl, qrl } from "@builder.io/qwik";

const App = /*#__PURE__*/ componentQrl(qrl(()=>import('./app_component_akbu84a8zes.js'), "App_component_AkbU84a8zes"));

export { App };
```

这里的 `App` 引用组件的部分被分离出了一个单独的 `js Chunk`，且调用这个组件的时候才会动态加载，完全实现了组件级别的按需加载。

```javascript
// app_component_akbu84a8zes.js
import { jsx as _jsx } from "@builder.io/qwik/jsx-runtime";
import { qrl } from "@builder.io/qwik";
export const App_component_AkbU84a8zes = ()=>{
    console.log('render');
    return /*#__PURE__*/ _jsx("p", {
        onClick$: qrl(()=>import("./app_component_p_onclick_01pegc10cpw"), "App_component_p_onClick_01pEgC10cpw"),
        children: "Hello Qwik"
    });
};
```

而在被拆出去的动态组件中，其保持了原有的 `dom` 渲染，而 `js` 事件逻辑因为 `$` 标志的原因也被拆离为了单独的 `chunk`，唯有点击时才会动态加载执行。

```javascript
// app_component_p_onclick_01pegc10cpw
export const App_component_p_onClick_01pEgC10cpw = ()=>console.log('hello');
```

通过颗粒度细致到极点的拆分，让绝大多数无需在首屏加载的 `js` 、组件被分离为了异步加载的 `chunk`，在需要他们时才进行加载，以此完全实现了极致的延迟加载、执行。

## Resumable Vs. Hydration
![image.png](https://raw.githubusercontent.com/jeasonnow/pics/main/202305101652691.png)
### 昂贵的 Hydration
1. 客户端必须要等待加载所有的组件和业务逻辑。
2. 客户端必须执行所有的初始化 `js` 执行完成，方便绑定监听和初始化 `state`，而这部分内容很多都可以被服务端执行。

### Qwik 如何避免步 Hydration 后尘
#### 绑定监听
为了让页面可交互，在过去的时代里都需要 `js` 来进行绑定的操作，而 `Qwik` 却有一个很明显的优势，其在初次加载时为了保证页面加载的资源尽可能的少，会把所有的 `event` 都抽离成单独的 `chunk` 等待使用的时候再加载，且其会在页面中提供对应的加载器去绑定和执行，所以其绑定操作实际上完全可以由加载器接管。

```javascript
// dom
<button class="button-dark button-small" on:click="/src/counter_component_div_button_onclick_1_lkcvrojx09y.js#counter_component_div_button_onClick_1_LkCVrojX09Y[0 1]" q:id="g" data-qwik-inspector="components/starter/counter/counter.tsx:24:7">+</button>

// qwikLoader
const dispatch = async (element, onPrefix, ev, eventName = ev.type) => {
            const attrName = "on" + onPrefix + ":" + eventName;
            element.hasAttribute("preventdefault:" + eventName) && ev.preventDefault();
            const ctx = element._qc_;
            // check 是否加载过，加载过走缓存
            const qrls = null == ctx ? void 0 : ctx.li.filter((li => li[0] === attrName));
            if (qrls && qrls.length > 0) {
                for (const q of qrls) {
                    await q[1].getFn([ element, ev ], (() => element.isConnected))(ev, element);
                }
                return;
            }
            // 未加载过则加载资源
            const attrValue = getAttribute(element, attrName);
            if (attrValue) {
                const container = element.closest("[q\\:container]");
                const base = new URL(getAttribute(container, "q:base"), doc.baseURI);
                for (const qrl of attrValue.split("\n")) {
                    const url = new URL(qrl, base);
                    const symbolName = url.hash.replace(/^#?([^?[|]*).*$/, "$1") || "default";
                    const reqTime = performance.now();
                    const module = import(url.href.split("#")[0]);
                    resolveContainer(container);
                    const handler = (await module)[symbolName];
                    const previousCtx = doc.__q_context__;
                    if (element.isConnected) {
                        try {
                            doc.__q_context__ = [ element, ev, url ];
                            emitEvent("qsymbol", {
                                symbol: symbolName,
                                element: element,
                                reqTime: reqTime
                            });
                            await handler(ev, element);
                        } finally {
                            doc.__q_context__ = previousCtx;
                        }
                    }
                }
            }
        };
```

基于这种分离策略，`Qwik` 可以只在返回的页面中注入触发的事件资源路径、事件 `id`，和加载器源代码即可实现绑定监听事件和原始 `js` 文件的完全分离。

### 恢复页面渲染
**这个部分还没有完全理清，但是可以说一下思路**
通过在 `Server` 端直接将所有组件的节点，边界都使用标记在 `html` 返回之中标注，并将初始渲染的组件和依赖样式都放置到对应的 `id` 边界位置，从而实现在初始加载时仅加载首屏组件样式和返回的 `html` 即可完成首屏渲染的能力，且可以把页面的 `state` 直接通过一定的规则序列化为 `JSON String` 然后注入到 `html` 文件，达到后续可以基于该 state 进行变化。


## 思考
没有银弹，`Qwik` 很美好，思路非常优秀，但是其基于服务的特性会让前端开发脱离大众熟知的领域，一个刚诞生没有多久的框架必然伴随社区不成熟问题，其实也有很多细节，比如 `treeShaking` 在 `Qwik` 中的实现，点击事件加载的事件会不会因为依赖变得极其庞大而导致延迟问题。这些都是我们需要在使用时考虑的。

框架的切换是昂贵的，特别是在大型和老项目中，我们虽然不能直接使用最新的技术但是却可以将其中优秀的想法去粗取精，尽量的应用到我们的应用中，`Qwik` 的想法就是一个好例子。