---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
---

# Vite 实现原理

## create by shengliang

<style>
  h1{
    color: red;
    background: linear-gradient(to right, #70ADFF, #BD34FE);
    background-clip: text;
    -webkit-background-clip: text;
  }
</style>
---

# What is Vite ?

<div v-click>
下一代的前端工具链，为开发提供极速响应
</div>

<br>

<div v-click class="mb-3">
  Vite 是由 Vue 作者尤雨溪开发的 Web 开发工具，尤雨溪在微博上推广时对 Vite 做了简短介绍：
</div>

<div v-click class="text-4 indent">
  Vite，一个基于浏览器原生 ES imports 的开发服务器。利用浏览器去解析 imports，在服务器端按需编译返回，完全跳过了打包这个概念，服务器随起随用。同时不仅有 Vue 文件支持，还搞定了热更新，而且热更新的速度不会随着模块增多而变慢。针对生产环境则可以把同一份代码用 Rollup 打包。虽然现在还比较粗糙，但这个方向我觉得是有潜力的，做得好可以彻底解决改一行代码等半天热更新的问题。
</div>

<br>

<div v-click>
 从这段话中我们能够提炼一些关键点：
</div>

<ul v-click class="mt-1 pl-5">
  <li class="text-4">Vite 基于 ESM，因此实现了快速启动和即时模块热更新能力；</li>
  <li class="text-4">Vite 在服务端实现了按需编译。</li>
</ul>

<br>

<div v-click>
  甚至可以说得更直白一些：<strong>Vite 在开发环境下并没有打包和构建过程。</strong>
</div>

<style>
strong{
  color: #42b883;
}

h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---

# Project

首先，我们先创建一个基于 Vite 的应用，并启动：
<br>
```bash{all|1|2|3|all}
pnpm create vite
pnpm i
pnpm dev
```

<br>

然后浏览器请求 https://localhost:3000/ 得到的内容即是我们应用项目中的 index.html 内容。
<br>
```html{all|1,4|2|3|all}
<body> 
  <div id="app"></div> 
  <script type="module" src="/src/main.js"></script>
</body> 
```

<br>

依据 ESM 规范在浏览器 script 标签中的实现，对于`<script type="module" src="/src/main.js"></script>`内容：<strong>当出现 script 标签 type 属性为 module 时，浏览器将会请求模块相应内容。</strong>


<style>
  h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
strong{
  color: #42b883
}
</style>

---
layout: image-right
image: https://source.unsplash.com/collection/94734566/1920x1080
---

# Code

经过 Vite Server 处理 http://localhost:3000/src/main.js 请求后

返回内容和项目中的 ./src/main.js 略有差别：

```ts {all|1|2|3|all}
import { createApp } from 'vue' 
import App from './App.vue' 
import './index.css' 
```

<br>

```ts {all|1|2|3|all}
import { createApp } from '/@modules/vue.js' 
import App from '/src/App.vue' 
import '/src/index.css?import' 
```

![resolvePath](path.jpg)

<style>
h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# Principle

这里我们拆成两部分来看。

其中import { createApp } from 'vue'改为import { createApp } from '/@modules/vue.js'，原因很明显：<strong>import 对应的路径只支持 "/""./"或者 "../" 开头的内容，直接使用模块名 import，会立即报错。</strong>

所以在 Vite Server 处理请求时，通过 serverPluginModuleRewrite 这个中间件来给 import from 'A' 的 A 添加 /@module/ 前缀为 from '/@modules/A'

整个过程和调用链路较长，我对 Vite 处理 import 方法做一个简单总结：
<ul class="pl-5">
  <li>在 koa 中间件里获取请求 path 对应的 body 内容；</li>
  <li>通过 es-module-lexer 解析资源 AST，并拿到 import 的内容；</li>
  <li>如果判断 import 的资源是绝对路径，即可认为该资源为 npm 模块，并返回处理后的资源路径。比如上述代码中，vue → /@modules/vue。</li>
</ul>

<style>
strong{
  color: #42b883;
}
h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---
class: px-20
---

# Principle

<div class="white">对于形如：import App from './App.vue'和import './index.css'的处理，与第一种情况类似：</div>

<ul class="pl-5">
  <li>在 koa 中间件里获取请求 path 对应的 body 内容；</li>
  <li>通过 es-module-lexer 解析资源 AST，并拿到 import 的内容；</li>
  <li>如果判断 import 的资源是相对路径，即可认为该资源为项目应用中资源，并返回处理后的资源路径。比如上述代码中，./App.vue → /src/App.vue。</li>
</ul>

接下来浏览器根据 main.js 的内容，分别请求：

```ts {1|2|3}
/@modules/vue.js 
/src/App.vue 
/src/index.css?import 
```

对于 /@module/ 类请求较为容易，我们只需要完成下面三步：

<ul class="pl-5">
  <li>在 koa 中间件里获取请求 path 对应的 body 内容；</li>
  <li>判断路径是否以 /@module/ 开头，如果是，取出包名（这里为 vue.js）；</li>
  <li>去 node_modules 文件中找到对应的 npm 库，并返回内容。</li>
</ul>

<style>
h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>




---
preload: false
---

# Principle

<div class="white">接着，就是对 /src/App.vue 类请求进行处理，这就涉及 Vite 服务器的编译能力了。
我们先看结果，对比项目中的 App.vue，浏览器请求得到的结果显然出现了大变样：</div>

![parse](parse.jpg)

<style>
h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---

# Principle

<div class="white">实际上，App.vue 这样的单文件组件对应 script、style 和 template，在经过 Vite Server 处理时，服务端对 script、style 和 template 三部分分别处理，对应中间件为 serverPluginVue。即对 .vue 文件请求进行处理，通过 parseSFC 方法解析单文件组件，而 parseSFC 具体所做的事情，是调用 @vue/compiler-sfc 进行单文件组件解析。精简一下逻辑就是：</div>


<br>

```ts {1,12|2,11|3-10|all}
if (!query.type) { 
  ctx.body = ` 
    const __script = ${descriptor.script.content.replace('export default ', '')} 
    // 单文件组件中，对于 style 部分的编译，编译为对应 style 样式的 import 请求 
    ${descriptor.styles.length ? `import "${url}?type=style"` : ''} 
    // 单文件组件中，对于 template 部分的编译，编译为对应 template 样式的 import 请求 
    import { render as __render } from "${url}?type=template" 
    // 渲染 template 的内容 
    __script.render = __render; 
    export default __script; 
  `; 
} 
```

<style>
h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---

# Principle

<div class="white">总而言之，每一个 .vue 单文件组件都被拆分成多个请求。比如对应上面场景，浏览器接收到 App.vue 对应的实际内容后，发出 HelloWorld.vue 以及 App.vue?type=template 的请求（通过 type 这个 query 来表示是 template 还是 style）。koa server 进行分别处理并返回，对于 template 的请求，将使用 @vue/compiler-dom 进行编译 template 并返回内容。</div>

精简一下逻辑就是：

```ts {1,8|2|3-5|6|7|all}
if (query.type === 'template') { 
	const template = descriptor.template; 
	const render = require('@vue/compiler-dom').compile(template.content, { 
	  mode: 'module', 
	}).code; 
	ctx.type = 'application/javascript'; 
	ctx.body = render; 
} 
```

<style>
h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
preload: false
---

# Principle

<div class="white">对于 http://localhost:3000/src/index.css?import 请求稍微特殊，在浏览器中执行 updateStyle 方法：</div>

<div class="mt-2 overflow-auto w-770px h-410px">

```ts 
const supportsConstructedSheet = (() => { 
  try { 
    // 生成 CSSStyleSheet 实例，试探是否支持 ConstructedSheet 
    new CSSStyleSheet() 
    return true 
  } catch (e) {} 
  return false 
})() 
export function updateStyle(id: string, content: string) { 
  let style = sheetsMap.get(id) 
  if (supportsConstructedSheet && !content.includes('@import')) { 
    if (style && !(style instanceof CSSStyleSheet)) { 
      removeStyle(id) 
      style = undefined 
    } 
    if (!style) { 
      // 生成 CSSStyleSheet 实例 
      style = new CSSStyleSheet() 
      style.replaceSync(content) 
      document.adoptedStyleSheets = [...document.adoptedStyleSheets, style] 
    } else { 
      style.replaceSync(content) 
    } 
  } else { 
    if (style && !(style instanceof HTMLStyleElement)) { 
      removeStyle(id) 
      style = undefined 
    } 
    if (!style) { 
      // 生成新的 style 标签并插入到 document 挡住 
      style = document.createElement('style') 
      style.setAttribute('type', 'text/css') 
      style.innerHTML = content 
      document.head.appendChild(style) 
    } else { 
      style.innerHTML = content 
    } 
  } 
  sheetsMap.set(id, style) 
} 
```

</div>

<style>
h1 {
  background-color: #70ADFF;
  background-image: linear-gradient(45deg, #70ADFF 10%, #BD34FE 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>