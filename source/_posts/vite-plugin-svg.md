---
title: 给 Vite 写个插件 —— SVG 优化
date: 2022-9-01 20:20:32
tags:
top:
---

我最近在开发一个 `vue3` + `tsx` + `vite` 项目的时候，发现有个用户体验上的问题：

我在切换页面的时候，如果下一个页面有静态资源，比如 `SVG`，那么当网速慢的情况下，切换完页面后会有短暂的空白，如果有动画，就会让人感觉 `动画有点卡顿`，即使我提前固定了高度

效果如下：

![2722161a844ce1e57dea376c0c3db9f1_1 (1).gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2a6dca37a734c478753b90a0b772364~tplv-k3u1fbpfcp-watermark.image?)

那么该怎么解决这个问题？

## 一点想法

我一开始想到的方法是：当用户在第一页的时候，就提前加载第二页的 `SVG`\
不过这有个问题：那我是不是在第一页的时候要请求第二页的 `SVG`，在第二页的时候也要请求第三页的 `SVG`，以此类推？

我想一劳永逸地的解决这个问题：我把整个应用的所有 `SVG` 一次性加载，行不行？

这个时候我想到一个技术，叫做 —— `CSS 雪碧图`

> 让我想起之前在猪场游戏开发时经常做的事 —— 「打图集」

但是！`CSS 雪碧图` 这个东西，已经「过时了」—— 并且这个方式不太适合 `矢量图`，它适合 `.png`，而我用的是 `.svg`

那能不能我将所有 `SVG` 都打包成一个大的 `SVG`，然后用到的时候就显示，不用到的时候就把宽高设置为 0，并且隐藏？

那就得使用 `SVG Sprites`！

不过，`webpack` 有很多插件在 `Vite` 上没有，比如我目前想要的这个插件，所以就写一个 `SVG Sprites` 吧！

## 实现步骤

### 一、思路

1. 如果加载的是 `svg`，就遍历某个文件目录
2. 将遍历的这些内容放进一个大东西里
3. 这个大东西插到 `div` 里
4. 再把这个 `div` 插到 `body` 里

### 二、安装 `svgo` 与 `svgstore`

```zsh
pnpm i -D svgo svgstore
```

### 三、开始写插件

#### 1. 设计

新建这么个文件，这个文件就是我给 `Vite` 写的一个插件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25e2e3119d744e1ca898d8df943ff6e1~tplv-k3u1fbpfcp-watermark.image?)

插件里有两个重要的函数：`resolveId` 和 `load`

- `resolveId` 用于 `解析/兼容`
- `load` 就相当于 `webpack` 的 `loader`，下面会有实现思路

#### 3. 为什么要写 `resolveId`？

> 这个很简单：只是为了兼容编辑器，免得在其他文件引用的时候 `@svgstore` 的时候会报错

我们包含 `所有 SVG` 的 `大SVG` 其实是不存在的，那么就没办法 import，所以我们要在 `main.ts` 里引入 `@svgstore`，但是有一些编辑器不支持这样引入，会报错 `xxx不存在`，所以我们要做这么一个 `没必要的判断`

```js
resolveId(id) {
  if (id === '@svgstore') {
    return 'svg_bundle.js'
  }
}
```

#### 4. 写 `load` 的思路

> 这其实就是 `webpack` 的 `loader`

1. 如果加载的 `文件id` 是 `@svgstore`，我就创建一个 `sprites`，`sprites` 是 `svgstore` 提供的一个函数
2. 这个 `sprites` 是一个空的 `大SVG`，最终想要它 `包含所有 svg`
3. 我们遍历了 `'src/assets/icons'` 目录下的 `所有文件`，也就是某个目录下的 `所有SVG`，`文件名` 对应 `svg 的 id`，`文件内容` 对应 `svg 的 内容`，最后把这些 `svgid` 和 `code` 加到 `sprites` 里
4. 到这里其实就已经完成了，不过还需要使用 `svgo` 来做优化 svg 文件：删除一些没有用的属性、空格...，这样可以让文件变得更小
5. 最后我们把 `code` 变成一个 `js` 文件 `return` 出去 —— 因为我们最终是要生成 `svg_bundle.js` 的 `js文件`，所以这个内容必须是一个 `合法的 JavaScript`
6. 我们创建一个 `div`，`div` 的内容就是 `大SVG` 的内容
7. `没用到的 svg` 我们就把它的宽高设置为 0，并且隐藏起来，如果用到了再从后面插入

#### 5. 最终代码

```js
import path from 'path'
import fs from 'fs'
import store from 'svgstore'
import { optimize } from 'svgo'

export const svgstore = (options = {}) => {
  const inputFolder = options.inputFolder || 'src/assets/icons'
  return {
    name: 'svgstore',
    resolveId(id) {
      if (id === '@svgstore') {
        return 'svg_bundle.js'
      }
    },
    load(id) {
      if (id === 'svg_bundle.js') {
        const sprites = store(options)
        const iconsDir = path.resolve(inputFolder)
        for (const file of fs.readdirSync(iconsDir)) {
          const filepath = path.join(iconsDir, file)
          const svgid = path.parse(file).name
          let code = fs.readFileSync(filepath, { encoding: 'utf-8' })
          sprites.add(svgid, code)
        }
        const { data: code } = optimize(sprites.toString({ inline: options.inline }), {
          plugins: ['cleanupAttrs', 'removeDoctype', 'removeComments', 'removeTitle', 'removeDesc', 'removeEmptyAttrs', { name: 'removeAttrs', params: { attrs: '(data-name|data-xxx)' } }]
        })
        return `const div = document.createElement('div')
	  div.innerHTML = \`${code}\`
	  const svg = div.getElementsByTagName('svg')[0]
	  if (svg) {
	      svg.style.position = 'absolute'
	      svg.style.width = 0
	      svg.style.height = 0
	      svg.style.overflow = 'hidden'
	      svg.setAttribute("aria-hidden", "true")
	    }
	    // listen dom ready event
	    document.addEventListener('DOMContentLoaded', () => {
	      if (document.body.firstChild) {
	        document.body.insertBefore(div, document.body.firstChild)
	      } else {
	      document.body.appendChild(div)
	    }
	  })
	`
      }
    }
  }
}
```

### 四、配置/引用插件

#### 1. tsconfig.node.json

> 在 `"include"` 里把 `"src/vite_plugins/**/*"` 加进去

```json
{
  "compilerOptions": {
    "composite": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts", "src/vite_plugins/**/*"]
}
```

#### 2. vite.config.ts

> 引入 `svgstore`，并且使用 `svgstore()`

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
// @ts-nocheck
import { svgstore } from './src/vite_plugins/svgstore'

// https://vitejs.dev/config/
export default defineConfig({
  base: './',
  plugins: [
    vue(),
    vueJsx({
      transformOn: true,
      mergeProps: true
    }),
    svgstore()
  ]
})
```

### 五、使用插件

#### 1. 在 `main.ts` 引入

```ts
// ... some codes
// import '...'
// import '...'
import '@svgstore'

// ... some codes
// xxx()
// xxx()
```

#### 2. 使用 `svg`

> 语法是：
>
> ```html
> <svg>
>   <use xlinkHref='#<文件名>' >
> </svg>
> ```

```tsx
export const Demo = () => (
  <svg>
    <use xlinkHref='#welcome_1' />
  </svg>
)

First.displayName = 'Demo'
```

## 最终效果

> 是不是很爽滑？

![540d1432c407cda85d883c34b731af6d.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e19a9042033c4eda8d40dd8c0d8de2c3~tplv-k3u1fbpfcp-watermark.image?)
