# Static File Integration / 静态文件方式集成

静态文件方式适合原生项目、npm 安装存在问题时、离线内网环境，也适合 Vue、ES5 等前端框架项目在无法成功安装和集成、运行不正常时使用。

## 1. 下载 SDK 静态包

下载 3 个静态资源包，其中 `mars3d-cesium` 和 `turf` 为依赖库：

- `mars3d-sdk.zip`
- `mars3d-cesium.zip`
- `turf.min.js`

## 2. 拷贝 SDK 到项目中

### 2.1 原生 JS 项目中

当在传统技术栈时，例如 HTML、Java、PHP、.NET 等，可以在 HTML 的 `head` 标签中引入 Cesium 官方包和 Mars3D 包所有相关资源。此方法比较简单，`mars3d` 已经挂载在 `window` 对象下面，不容易出现各类集成问题。

第一步在项目根目录创建 `lib` 目录，将下载的 `mars3d-sdk.zip` 和 `mars3d-cesium.zip` 解压到 `lib` 目录下，解压整理后目录结构为：

```text
index.html
lib/
  mars3d/
    img/
    mars3d.css
    mars3d.js
  Cesium/
    Widgets/
      widgets.css
    Assets/
    ThirdParty/
    Workers/
    Cesium.js
  turf/
    turf.min.js
```

### 2.2 Vue 等前端框架项目中

如果是 Vue、ES5 等前端框架项目，也可以使用静态引入方式。第一步在对应框架的静态文件目录创建 `lib`，比如 Vue 3 项目的 `public` 目录下：

```text
public/
  lib/
    mars3d/
      img/
      mars3d.css
      mars3d.js
    Cesium/
      Widgets/
        widgets.css
      Assets/
      ThirdParty/
      Workers/
      Cesium.js
    turf/
      turf.min.js
src/
index.html
```

## 3. 修改 HTML 页面 head 增加静态资源引入

修改页面 `head` 部分，增加引入 Cesium、Mars3D、Turf 等静态资源：

```html
<!-- 引入 cesium 基础 lib -->
<script>
  // window.CESIUM_BASE_URL = "./lib/Cesium/" // 非必须，如 jsp、asp.net 等非 html 框架报错时建议取消注释
</script>
<link href="./lib/Cesium/Widgets/widgets.css" rel="stylesheet" type="text/css" />
<script src="./lib/Cesium/Cesium.js" type="text/javascript"></script>
<script src="./lib/turf/turf.min.js" type="text/javascript"></script>

<!-- 引入 mars3d 库 lib -->
<link href="./lib/mars3d/mars3d.css" rel="stylesheet" type="text/css" />
<script src="./lib/mars3d/mars3d.js" type="text/javascript"></script>

<!-- 引入 mars3d 库插件 lib（按需引入） -->
<script src="./lib/mars3d/plugins/space/mars3d-space.js" type="text/javascript"></script>
```

## 4. 增加 DIV 展示容器

在需要呈现 Mars3D 地图的 HTML 页面中，加上 `div` 容器，并注意设置 `div` 的 CSS 高宽样式。可以直接使用内置样式 `mars3d-container`：

```html
<div id="mars3dContainer" class="mars3d-container"></div>
```

## 5. 创建地图对象

在页面 JS 代码中，可通过 `window.mars3d.*` 使用 Mars3D 相关类及方法：

```ts
const mars3d = window.mars3d // 静态资源引入时，对象都是挂载在 window 中
const Cesium = window.mars3d.Cesium

// Map 的参数请看 API 文档：http://mars3d.cn/api/Map.html
const map = new mars3d.Map("mars3dContainer", {
  basemaps: [{ name: "天地图", type: "tdt", layer: "img_d", show: true }]
})
```

## 6. 完成集成

这样 Mars3D 三维地球就集成完成了，可以预览相关页面查看地图效果。

## 7. 常见问题

### 7.1 eslint 错误校验问题解决

如果是 TypeScript、ESLint 相关技术栈项目中，`window.*` 取值存在 ESLint 校验报错，可以在 `.eslintrc.json` 的 `globals` 中增加：

```json
{
  "root": true,
  "globals": {
    "Cesium": "readonly",
    "mars3d": "readonly"
  }
}
```

### 7.2 如果想代码中 import 导入 mars3d，打包时实际使用 CDN 或静态资源

如果想在代码中通过 npm 安装 `mars3d` 包并使用 import 导入，但打包运行时实际使用 CDN 或静态资源，需要参考项目技术栈规则，排除 `mars3d`、`mars3d-cesium`、`@turf/turf` 的引用和打包。

`vue.config.js`：

```js
module.exports = {
  // 已忽略其他无关配置
  configureWebpack: {
    externals: {
      "mars3d": "mars3d",
      "mars3d-cesium": "Cesium",
      "@turf/turf": "turf"
    }
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
      externals: {
        "mars3d": "mars3d",
        "mars3d-cesium": "Cesium",
        "@turf/turf": "turf"
      }
    })
  ]
})
```
