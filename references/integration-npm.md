# NPM Integration / NPM 安装方式集成

当项目使用 Node 前端技术栈，并通过 `package.json` 安装 `mars3d` 时，使用这份文档。先确认构建工具是 webpack、vue-cli 还是 vite，再只使用对应配置段。

## 1. 从 npm 安装获取 SDK

安装 Mars3D 主库，其中 `mars3d-cesium`、`@turf/turf` 为依赖库：

```bash
npm install mars3d mars3d-cesium @turf/turf --save
pnpm add mars3d mars3d-cesium @turf/turf
yarn add mars3d mars3d-cesium @turf/turf
```

Mars3D 插件按需安装，例如：

```bash
npm install mars3d-space --save
pnpm add mars3d-space --save
yarn add mars3d-space --save
```

## 2. 增加 DIV 展示容器

在需要呈现 Mars3D 地图的 HTML 页面或 Vue/ES5 等组件中，加上 `div` 容器，并注意设置容器 CSS 高宽样式。可以直接使用内置样式 `mars3d-container`：

```html
<div id="mars3dContainer" class="mars3d-container"></div>
```

## 3. 在相关组件中 import 导入 SDK

安装后在相关组件文件中 import 导入 SDK，即可创建地图对象并显示到页面中：

```ts
// 引入 css
import "mars3d-cesium/Build/Cesium/Widgets/widgets.css"
import "mars3d/mars3d.css" // v3.8.6 及之前版本使用 import "mars3d/dist/mars3d.css";

import * as mars3d from "mars3d"
import "mars3d-space" // 导入 mars3d 插件，导入即可自动注册；按需使用，需要先 npm install mars3d-space
// import "mars3d-heatmap" // 其他插件类型，自行修改名称

const Cesium = mars3d.Cesium

// Map 的参数请看 API 文档：http://mars3d.cn/api/Map.html
const map = new mars3d.Map("mars3dContainer", {
  basemaps: [{ name: "天地图", type: "tdt", layer: "img_d", show: true }]
})
```

## 4. 修改相关配置

因为 Mars3D 所依赖的 Cesium 是一个多文件资源的 JS 库，需要对其依赖的一些资源文件进行拷贝处理。多数集成问题都出现在 Cesium 库资源拷贝处理上，项目运行时没有正确识别 Cesium 主目录会造成各种问题。

根据项目技术栈选择对应的配置方式，主要包括 webpack、vue-cli、vite。

### 4.1 webpack 技术栈时的项目配置修改

- 参考项目：`https://gitee.com/marsgis/mars3d-vue-template/tree/master/mars3d-webpack/`
- 修改文件：`webpack.config.js`

安装依赖库，注意 `copy-webpack-plugin` 与 webpack 版本的依赖关系，可能需要按 webpack 版本安装对应插件版本：

```bash
npm install copy-webpack-plugin --save-dev
```

修改 `webpack.config.js`：

```js
const path = require("path")
const CopyWebpackPlugin = require("copy-webpack-plugin")
const webpack = require("webpack")

const cesiumSourcePath = "node_modules/mars3d-cesium/Build/Cesium/" // cesium 库安装目录
const cesiumRunPath = "./mars3d-cesium/" // cesium 运行时路径

module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: ["style-loader", "css-loader"] },
      { test: /\.js$/, use: { loader: "babel-loader", options: { plugins: ["@babel/plugin-proposal-class-properties"] } } }
    ]
  },
  plugins: [
    // CESIUM_BASE_URL 是标识 cesium 资源所在的主目录，其内部资源加载、多线程等处理时需要用到
    new webpack.DefinePlugin({
      CESIUM_BASE_URL: JSON.stringify(cesiumRunPath)
    }),
    // Cesium 相关资源目录需要拷贝到系统目录下面；部分 CopyWebpackPlugin 版本的语法可能没有 patterns
    new CopyWebpackPlugin({
      patterns: [
        { from: path.join(cesiumSourcePath, "Workers"), to: path.join(cesiumRunPath, "Workers") },
        { from: path.join(cesiumSourcePath, "Assets"), to: path.join(cesiumRunPath, "Assets") },
        { from: path.join(cesiumSourcePath, "ThirdParty"), to: path.join(cesiumRunPath, "ThirdParty") },
        { from: path.join(cesiumSourcePath, "Widgets"), to: path.join(cesiumRunPath, "Widgets") }
      ]
    })
  ]
}
```

### 4.2 vue-cli 技术栈时的项目配置修改

- 参考项目：`https://gitee.com/marsgis/mars3d-vue-template/tree/master/mars3d-vue2`
- 修改文件：`vue.config.js`
- `@vue/cli-service` 建议使用 v5 及以上版本；否则建议使用静态方式集成 Mars3D。

安装依赖库，注意 `copy-webpack-plugin` 与 webpack 版本的依赖关系，可能需要按 webpack 版本安装对应插件版本：

```bash
npm install copy-webpack-plugin --save-dev
```

修改 `vue.config.js`：

```js
const path = require("path")
const CopyWebpackPlugin = require("copy-webpack-plugin")
const webpack = require("webpack")

module.exports = {
  configureWebpack: (config) => {
    const cesiumSourcePath = "node_modules/mars3d-cesium/Build/Cesium/" // cesium 库安装目录
    const cesiumRunPath = "./mars3d-cesium/" // cesium 运行时路径

    const plugins = [
      // 标识 cesium 资源所在的主目录，cesium 内部资源加载、多线程等处理时需要用到
      new webpack.DefinePlugin({
        CESIUM_BASE_URL: JSON.stringify(path.join(config.output.publicPath, cesiumRunPath))
      }),
      // Cesium 相关资源目录需要拷贝到系统目录下面；部分 CopyWebpackPlugin 版本的语法可能没有 patterns
      new CopyWebpackPlugin({
        patterns: [
          { from: path.join(cesiumSourcePath, "Workers"), to: path.join(config.output.path, cesiumRunPath, "Workers") },
          { from: path.join(cesiumSourcePath, "Assets"), to: path.join(config.output.path, cesiumRunPath, "Assets") },
          { from: path.join(cesiumSourcePath, "ThirdParty"), to: path.join(config.output.path, cesiumRunPath, "ThirdParty") },
          { from: path.join(cesiumSourcePath, "Widgets"), to: path.join(config.output.path, cesiumRunPath, "Widgets") }
        ]
      })
    ]

    return {
      module: { unknownContextCritical: false }, // 配置加载的模块类型，cesium 时必须配置
      plugins
    }
  }
}
```

### 4.3 vite 技术栈时的项目配置修改

- 参考项目：`https://gitee.com/marsgis/mars3d-vue-template/tree/master/mars3d-vue3-vite/`
- 修改文件：`vite.config.ts`

安装依赖库：

```bash
npm install vite-plugin-mars3d --save-dev
```

修改 `vite.config.ts`：

```ts
import { defineConfig } from "vite"
import { mars3dPlugin } from "vite-plugin-mars3d"

export default defineConfig({
  plugins: [mars3dPlugin()]
})
```

Vite 常见问题：

1. 使用原生 Cesium 时，不要安装 `mars3d-cesium`，不能同时存在两个 Cesium 包；在 `vite.config.ts` 中配置为 `mars3dPlugin({ cesiumPackageName: "cesium" })`。
2. 如果项目的 `package.json` 没有 `type: "module"`，或是 Vite 4 及之前版本，请使用 `"vite-plugin-mars3d": "~3.1.3"`。
3. 如果项目的 `package.json` 有 `type: "module"`，直接使用最新版本即可。
4. 如果无法排除解决问题，也可以在 `vite.config.ts` 配置为静态引入：`mars3dPlugin({ useStatic: true })`。

## 5. 完成集成

这样 Mars3D 三维地球就集成完成了，可以预览相关页面查看地图效果。

## 6. 常见问题

### 6.1 集成失败报错无法解决时

如果尝试修改配置无法成功集成，也可以排除 Cesium 库，改为在 HTML 中直接引入 Cesium 资源。这样可以不用处理上一步的 Cesium 拷贝配置，配置方式更简单。

在 `index.html` 的 `head` 中引入 Cesium 资源：

```html
<!-- 引入 cesium 基础 lib -->
<link href="./lib/Cesium/Widgets/widgets.css" rel="stylesheet" type="text/css" />
<script src="./lib/Cesium/Cesium.js" type="text/javascript"></script>
```

然后在对应技术栈配置中排除 `mars3d-cesium` 的引用和打包。

`vue.config.js`：

```js
module.exports = {
  // 已忽略其他无关配置
  configureWebpack: {
    externals: { "mars3d-cesium": "Cesium" } // 排除使用 mars3d-cesium
  }
}
```

`vite.config.ts`：

```ts
import { defineConfig } from "vite"
import { viteExternalsPlugin } from "vite-plugin-externals"

export default defineConfig({
  // 已忽略其他无关配置
  plugins: [
    viteExternalsPlugin({
      externals: { "mars3d-cesium": "Cesium" } // 排除使用 mars3d-cesium
    })
  ]
})
```

### 6.2 临时更新 SDK 为最新版本

如果 Mars3D 官方临时修改 SDK，但未发布最新 npm 正式包，可以通过官网下载最新 SDK 后按以下方式更新。

方式 1：临时覆盖本地 `node_modules`。该方式可用于临时更新本地 `node_modules` 中的 `mars3d` 包，但每次 `npm install` 后都需要覆盖一次。

将离线包的 `mars3d.js`、`mars3d.css`、`mars3d.d.ts` 文件覆盖到 `node_modules/mars3d/` 目录下同名文件。下载地址：

- `http://mars3d.cn/lib/mars3d/mars3d.css`
- `http://mars3d.cn/lib/mars3d/mars3d.js`
- `http://mars3d.cn/lib/mars3d/mars3d.d.ts`

覆盖后执行 `npm run clean-cache` 清理缓存，确保浏览器 F12 控制台打印的编译日期是最新的。可在 `package.json` 的 `scripts` 节点下增加：

```json
"clean-cache": "rimraf node_modules/.cache/ && rimraf node_modules/.vite"
```

方式 2：修改 `package.json` 指向项目本地目录。该方式适合授权版本 SDK 或想永久固化版本时使用。

1. 在项目根目录创建 `packages/` 目录。
2. 将 `mars3d-sdk.zip` 离线包放到 `packages` 目录并解压到当前目录下。
3. 解压后目录结构为 `packages/mars3d/`，其中包含 `img/`、`mars3d.css`、`mars3d.d.ts`、`mars3d.js`、`package.json`。
4. 修改 `package.json` 中 `mars3d` 包配置为 `"mars3d": "file:packages/mars3d"`。
5. 删除 `node_modules`，重新执行 `npm install` 安装依赖。
