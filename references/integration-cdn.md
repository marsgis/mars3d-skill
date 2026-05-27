# CDN Integration / CDN 方式集成

CDN 方式集成与静态引入方式集成到项目类似，只是引入的资源是从 CDN 服务器获取的。这样可以避免下载安装包，但不适合生产环境，CDN 存在不稳定的情况。

## 1. 从 CDN 获取

在 HTML 的 `head` 标签中引入 Mars3D 包相关 CDN 资源，有以下不同的 CDN 服务可以选择，任意选择一种即可。

### 1.1 淘宝 npmjs.com 镜像服务站

- 提供商：国内淘宝
- 官网：`https://registry.npmmirror.com`
- 说明：国内访问速度快且稳定，推荐使用。

```html
<!-- 引入 cesium 基础 lib -->
<script>
  // window.CESIUM_BASE_URL = "https://registry.npmmirror.com/mars3d-cesium/latest/files/Build/Cesium/" // 非必须，如 jsp、asp.net 等非 html 框架报错时建议取消注释
</script>
<link href="https://registry.npmmirror.com/mars3d-cesium/latest/files/Build/Cesium/Widgets/widgets.css" rel="stylesheet" type="text/css" />
<script src="https://registry.npmmirror.com/mars3d-cesium/latest/files/Build/Cesium/Cesium.js" type="text/javascript"></script>
<script src="http://mars3d.cn/lib/turf/turf.min.js" type="text/javascript"></script>

<!-- 引入 mars3d 库 lib -->
<link href="https://registry.npmmirror.com/mars3d/latest/files/mars3d.css" rel="stylesheet" type="text/css" />
<script src="https://registry.npmmirror.com/mars3d/latest/files/mars3d.js" type="text/javascript"></script>

<!-- 引入 mars3d 库插件 lib（按需引入） -->
<script src="https://registry.npmmirror.com/mars3d-space/latest/files/mars3d-space.js" type="text/javascript"></script>
```

### 1.2 国外 unpkg.com 服务

- 提供商：国外 Michael Jackson 维护的开源项目，由 Cloudflare 提供服务器支持。
- 官网：`https://unpkg.com`
- 说明：国外稳定，国内偶尔被墙。

```html
<!-- 引入 cesium 基础 lib -->
<script>
  // window.CESIUM_BASE_URL = "https://unpkg.com/mars3d-cesium@latest/Build/Cesium/" // 非必须，如 jsp、asp.net 等非 html 框架报错时建议取消注释
</script>
<link href="https://unpkg.com/mars3d-cesium@latest/Build/Cesium/Widgets/widgets.css" rel="stylesheet" type="text/css" />
<script src="https://unpkg.com/mars3d-cesium@latest/Build/Cesium/Cesium.js" type="text/javascript"></script>
<script src="https://unpkg.com/@turf/turf/turf.min.js" type="text/javascript"></script>

<!-- 引入 mars3d 库 lib -->
<link href="https://unpkg.com/mars3d@latest/mars3d.css" rel="stylesheet" type="text/css" />
<script src="https://unpkg.com/mars3d@latest/mars3d.js" type="text/javascript"></script>

<!-- 引入 mars3d 库插件 lib（按需引入） -->
<script src="https://unpkg.com/mars3d-space@latest/mars3d-space.js" type="text/javascript"></script>
```

### 1.3 国外 jsDelivr 服务

- 提供商：国外一个开源的免费公共 CDN 平台
- 官网：`https://www.jsdelivr.com/package/npm/mars3d`
- 说明：国外稳定，国内偶尔被墙。

```html
<!-- 引入 cesium 基础 lib -->
<script>
  // window.CESIUM_BASE_URL = "https://cdn.jsdelivr.net/npm/mars3d-cesium@latest/Build/Cesium/" // 非必须，如 jsp、asp.net 等非 html 框架报错时建议取消注释
</script>
<link href="https://cdn.jsdelivr.net/npm/mars3d-cesium@latest/Build/Cesium/Widgets/widgets.css" rel="stylesheet" type="text/css" />
<script src="https://cdn.jsdelivr.net/npm/mars3d-cesium@latest/Build/Cesium/Cesium.js" type="text/javascript"></script>
<script src="https://unpkg.com/@turf/turf/turf.min.js" type="text/javascript"></script>

<!-- 引入 mars3d 库 lib -->
<link href="https://cdn.jsdelivr.net/npm/mars3d@latest/mars3d.css" rel="stylesheet" type="text/css" />
<script src="https://cdn.jsdelivr.net/npm/mars3d@latest/mars3d.js" type="text/javascript"></script>

<!-- 引入 mars3d 库插件 lib（按需引入） -->
<script src="https://cdn.jsdelivr.net/npm/mars3d-space@latest/mars3d-space.js" type="text/javascript"></script>
```

## 2. 增加 DIV 展示容器

在需要呈现 Mars3D 地图的 HTML 页面中，加上 `div` 容器，并注意设置 `div` 的 CSS 高宽样式。可以直接使用内置样式 `mars3d-container`：

```html
<div id="mars3dContainer" class="mars3d-container"></div>
```

## 3. 创建地图对象

在页面 JS 代码中，可通过 `window.mars3d.*` 使用 Mars3D 相关类及方法：

```ts
const mars3d = window.mars3d // 静态资源引入时，对象都是挂载在 window 中
const Cesium = window.mars3d.Cesium

// Map 的参数请看 API 文档：http://mars3d.cn/api/Map.html
const map = new mars3d.Map("mars3dContainer", {
  basemaps: [{ name: "天地图", type: "tdt", layer: "img_d", show: true }]
})
```

## 4. 完成集成

这样 Mars3D 三维地球就集成完成了，可以预览相关页面查看地图效果。

## 5. 常见问题

### 5.1 eslint 错误校验问题解决

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

### 5.2 如果想代码中 import 导入 mars3d，打包时实际使用 CDN

如果想在代码中通过 npm 安装 `mars3d` 包并使用 import 导入，但打包运行时实际使用 CDN，需要参考项目技术栈规则，排除 `mars3d`、`mars3d-cesium`、`@turf/turf` 的引用和打包。

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
