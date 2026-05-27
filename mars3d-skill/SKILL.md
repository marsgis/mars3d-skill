---
name: mars3d-skill
description: "构建、修改、调试、解释和集成 Mars3D Web GIS 应用，使用 Mars3D 开发者中心 和 官方API。适用于任务提到 Mars3D、mars3d、mars3d.Map、Cesium 或 mars3d-cesium 集成、安装或集成报错、Vue 或 ES5 Mars3D 项目、Mars3D Vue widget architecture、widget-store.ts、useLifecycle(mapWork)、map.ts widgets、config.json、map scene 、basemaps、terrain、tile layers、graphics、3D Tiles、glTF或 坐标转化、events、camera、controls、effects、量算、ArcGIS services、WebGL，或 Mars3D 常见问题。"
---

# Mars3D API

使用这个 skill 时，优先参考 Mars3D 开发者中心 (`https://mars3d.cn/docs/`) 和 API 文档 (`https://mars3d.cn/api/`)。

## 工作流程

1. 先判断任务类型。
2. 如果任务是普通功能开发、代码修改、API 解释、类用法、图层/矢量/控件/效果实现时，直接基于现有项目代码、本地声明文件和对应 reference 工作，如果是vue技术栈，加载`references/vue-engineering-practices.md`；如果使用了widget机制，`index.vue` 和 `map.ts` widget patterns，加载`references/widget-patterns.md`。

   **注意：** 如果用户的问题属于"参数没起作用"或"调用报错"（即 Step 5 的覆盖范围），则 Step 5 **优先级高于 Step 2**，直接进入 Step 5 加载 `api-errors.md`，不要用 Step 2 的处理方式。
3. 只有用户询问安装、集成、升级、打包失败、地图空白/不显示、Cesium 资源报错、`mars3d-cesium` 冲突、webpack/vue-cli/vite 配置异常、静态文件或 CDN 引入失败时，才进入集成排查流程。
4. 进入集成排查流程后，先加载 `references/integration.md`，检查本地版本和集成方式：`package.json`、lockfiles、`node_modules/mars3d`、`node_modules/mars3d-cesium`、`mars3d.d.ts`、入口 HTML 和构建配置。识别为 npm、静态文件或 CDN 集成后，再按需加载 `integration-npm.md`、`integration-static.md` 或 `integration-cdn.md`。
5. 当用户使用mars3d当中的某一个api，但是其中的参数没有起作用，或者设置了某个参数但是报错，比如：
   ```js
      map.keyboardRoam = true
   ```
   设置了map.keyboardRoam = true，但是没有效果，或者报错，此时需要加载`references/api-errors.md`


6. 按任务选择对应 reference：
   - 安装集成入口、集成方式判断、本地版本确认：`references/integration.md`
   - npm安装方式集成、webpack/vue-cli/vite、Cesium资源复制：`references/integration-npm.md`
   - 静态文件集成、入口HTML通过script引入mars3d.js：`references/integration-static.md`
   - CDN临时演示集成、远程script资源引入：`references/integration-cdn.md`
   - Vue practical engineering patterns、reactive id storage、table-map synchronization：`references/vue-engineering-practices.md`
   - Mars3D Vue widget detection、page-map separation、`index.vue` plus `map.ts` widget patterns：`references/widget-patterns.md`
   - 当代码或任务出现具体 Mars3D class/enum，并需要查官方 API 页面时，加载 `references/api-navigation.md`。例如用户使用 `new mars3d.graphic.PointEntity` 且需要参考 `PointEntity` API，该 reference 会说明取 `PointEntity` 作为 `{ClassName}`，访问 `https://mars3d.cn/api/PointEntity.html`。
   - code templates：`references/coding-patterns.md`
7. 优先使用 Mars3D abstractions，而不是直接使用 raw Cesium：`mars3d.Map`、`mars3d.layer.*`、`mars3d.graphic.*`、`mars3d.thing.*`、`mars3d.EventType`、`mars3d.LayerUtil`、`mars3d.GraphicUtil` 和 `mars3d.MaterialUtil`。

## 来源优先级

按以下顺序使用信息来源：

- 本地已安装 SDK files 和 type declarations。
- 现有项目代码、`config.json`、examples 和 templates。
- `https://mars3d.cn/docs/` 下的 Mars3D Developer Center docs。
- `https://mars3d.cn/api/` 下的 Official API pages。
- docs 链接的 official examples，例如 editor pages，仅用于参考 usage patterns。

当用户询问 current 或 latest behavior 时，重新检查 online docs 或 installed package。Mars3D options、plugins 和 Cesium integration details 会随版本变化。

## 实现规则

- 不要把项目变量或 class 命名为 `mars3d`；SDK 保留 `mars3d.*` namespace。
- 在 npm projects 中，除非项目使用 static Cesium setup，否则同时引入 Cesium widget CSS 和 Mars3D CSS：

```ts
import "mars3d-cesium/Build/Cesium/Widgets/widgets.css"
import "mars3d/mars3d.css"
import * as mars3d from "mars3d"
```

- 确保 map DOM container 有真实 width 和 height；blank maps 经常来自 CSS sizing 问题。
- 把 `mars3d.Map` 当作 app entry point。Initial scene、terrain、basemaps、layers、controls、effects 和 things 可以通过 constructor options 或 `config.json` 参数化。
- layers 使用 `map.addLayer(layer)`；graphics 使用 `graphicLayer.addGraphic(graphic)`。避免混用 layer 和 graphic APIs。
- 坐标数组默认保持常见 `[lng, lat, alt]` 形式，除非 class docs 明确要求 `Cesium.Cartesian3`、`LngLatPoint` 或其他类型。
- 从绘制完成的 point graphic 读取坐标时，优先使用 `graphic.centerPoint`，或用 `mars3d.LngLatPoint.fromCartesian` 转换 `graphic.center`/Cartesian。不要假设 `graphic.position` 暴露 `lng` 和 `lat`。
- Mars3D objects 的事件绑定和解绑使用 `on` 和 `off`。事件类型优先使用 `mars3d.EventType` 和 class-specific event docs。
- 在 Vue/ES5 中，等 DOM container 存在后再创建 map，并在 unmount 时 destroy。不要把 Mars3D object instances 放进 Vue reactive `data`、`computed` 或 central stores。
- 在 Mars3D Vue projects 中，添加 map-linked UI 前先判断项目是否已经使用 widget mechanism。如果已经使用，page UI 放在 views，map operations 放进现有 `widget-store.ts` -> `index.vue` -> `map.ts` -> `useLifecycle(mapWork)` pattern。
- 避免 hard-coded real third-party tokens。使用 environment/config values，并在创建依赖 token 的 layers 前设置 SDK tokens。
- 除非项目明确 externalizes Cesium，否则 Mars3D 项目中保持使用 `mars3d-cesium`，不要意外打包两份 Cesium packages。

## 验证清单

- Install/build：dependencies 能正确 resolve，Cesium runtime assets 被正确复制或 externalized，并且不存在 duplicate Cesium package。
- Runtime：console 打印预期 Mars3D/Cesium versions，map container 尺寸正常，WebGL 可用，并且没有 CORS/token errors。
- Data：service URLs 能在 browser/network tab 加载，CRS/chinaCRS 正确，并用 `flyTo` 或 camera view 确认可见。
- Lifecycle：map、layers、graphics、events、timers 和 plugin objects 在 teardown 时被 remove 或 destroy。
