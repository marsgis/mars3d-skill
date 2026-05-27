# Mars3D Vue Widget Patterns / Mars3D Vue widget 模式

当 Mars3D Vue 项目通过 widgets 分离 page UI 和 map operations 时，使用这份 reference。不要假设每个 Mars3D Vue 项目都使用这套机制；必须先检测。

## Detect The Widget Mechanism / 检测 widget 机制

当项目中存在多个以下信号时，视为使用 Mars3D widget mechanism：

- 存在 `src/common/store/widget.ts`，并导出 `injectState`、`useWidget` 或 `useWidgetStore`。
- 存在 `src/common/uses/use-lifecycle.ts`，并通过 inject `getMapInstance` 后调用 `mapWork.onMounted(map, params)`。
- `src/components/mars-work/main-view.vue` 同时渲染 `<mars-map>` 和 `<mars-widget>`，并 provide `getMapInstance`。
- 页面入口如 `src/pages/*/main.ts` import `./widget-store`，并调用 `app.use(injectState(store), key)`。
- 存在 `src/pages/*/widget-store.ts`，里面定义 `widgets`、`openAtStart`，并用 `defineAsyncComponent` 注册 async widget components。
- widget 文件夹中同时存在 `index.vue` 和 `map.ts`，并且 `index.vue` 调用 `useLifecycle(mapWork)`。

可用搜索：

```bash
rg -n "useLifecycle\(|useWidget\(|useWidgetStore\(|injectState\(|openAtStart|mars-widget|main-view|widget-store|getMapInstance|getCurrentWidget" src
rg --files src | rg "(widget-store\.ts|common/.*/widget|use-lifecycle|components/.*/main-view\.vue|components/.*/widget\.vue|widgets/)"
```

如果项目只有普通 `<mars-map>` 创建，没有 widget store、`main-view`、`mars-widget` 或 `useLifecycle`，就不要按这套 widget mechanism 实现。此时沿用项目已有结构，除非用户明确要求引入 Mars3D widget architecture。

如果只存在部分信号，说明项目可能是 Mars3D widget-template 的裁剪版或自定义变体。优先扩展最接近的已有模式，不要另起一套平行框架。

## Separation Rule / 分离规则

普通 page operations 留在 page views：

- routes、headers、titles、static layout、pure page buttons、page-only dialogs；
- 不需要 `mars3d.Map`、layer、graphic、camera 或 map event 的 UI。

map-linked operations 放进 widgets：

- 创建或移除 layers 和 graphics；
- 绑定 popups、tooltips、context menus 和 map events；
- 执行 draw、edit、query、measure、analysis、camera、layer-management workflows；
- 展示控制或反映 map state 的 floating panels。

这个模式的目的，是避免普通 Vue pages 和 Mars3D object lifecycles 直接耦合。

## Standard Widget Shape / 标准 widget 结构

项目已有这条链路时，沿用它：

```text
widget-store.ts -> main-view.vue -> mars-widget -> widget/index.vue -> useLifecycle(mapWork) -> widget/map.ts
```

- `widget-store.ts`: 注册 widget metadata，通常使用 `markRaw(defineAsyncComponent(...))`、唯一 `name`，以及 `autoDisable`、`disableOther`、`group`、`meta` 或 `openAtStart` 等 options。
- `main-view.vue`: 用 `<mars-map>` 创建 map，等待 map load，provide `getMapInstance`，再渲染 visible widgets。
- `mars-widget`: 渲染 async widget component，并 provide `getCurrentWidget` 给 `useWidget()` 使用。
- `index.vue`: 实现 floating UI 和用户交互。只处理 panel state 和 button events。
- `map.ts`: 承载 Mars3D business logic。`map`、layers、graphics、events、timers 和 cleanup 都作为普通 module variables 保存在这里。
- `useLifecycle(mapWork)`: widget mount 时调用 `mapWork.onMounted(map, params)`，unmount 时调用 `mapWork.onUnmounted()`。

不要把 `mars3d.Map`、`GraphicLayer`、graphics 或 Cesium objects 放进 Vue deep reactive state。Vue 中只保存 ids 或 serializable business state，Mars3D objects 留在 `map.ts` 或 Mars3D containers。

## Implementation Pattern / 实现模式

注册 widget：

```ts
{
  component: markRaw(defineAsyncComponent(() => import("@mars/widgets/demo/menu/index.vue"))),
  name: "menu",
  autoDisable: false,
  disableOther: false
}
```

需要默认打开时配置：

```ts
openAtStart: ["menu"]
```

在 `index.vue` 中，把 UI 事件接到 map module：

```ts
import useLifecycle from "@mars/common/uses/use-lifecycle"
import * as mapWork from "./map"

useLifecycle(mapWork)

function addDivGraphic() {
  mapWork.addDivGraphic()
}
```

在 `map.ts` 中接收 map，并管理 map work：

```ts
import * as mars3d from "mars3d"

export let map: mars3d.Map | undefined
export let graphicLayer: mars3d.layer.GraphicLayer | undefined

export function onMounted(mapInstance: mars3d.Map): void {
  map = mapInstance
  graphicLayer = new mars3d.layer.GraphicLayer()
  map.addLayer(graphicLayer)
}

export function addDivGraphic() {
  const graphic = new mars3d.graphic.DivGraphic({
    position: [117.229619, 31.8, 1521],
    pointerEvents: true,
    style: {
      html: `<div class="marsGreenGradientPnl">安徽欢迎您</div>`
    }
  })

  graphicLayer?.addGraphic(graphic)
}

export function onUnmounted(): void {
  graphicLayer?.clear()
  if (graphicLayer && map) {
    map.removeLayer(graphicLayer, true)
  }
  graphicLayer = undefined
  map = undefined
}
```

cleanup 要贴合项目现有风格。如果 owning framework 已经在其他位置移除 widget layers，就遵循该约定；否则在 `onUnmounted` 中 remove layers、events、timers 和 temporary graphics。

## Working Guidelines / 工作准则

- 添加新的 map-linked UI 前，先检测项目是否使用 widget mechanism。
- widget projects 中，优先新增 widget 或扩展已有 widget，不要把 map operations 放进 route views。
- 保持 `index.vue` 薄：收集 UI input，然后调用 `map.ts` 导出的函数。
- 保持 `map.ts` 可读可测：暴露 `addDivGraphic`、`removeGraphic`、`startDraw`、`locateGraphic` 等小函数。
- `map.ts` 绘制 point 并把数据回传给 `index.vue` 时，从 `graphic.centerPoint` 取坐标，只 fire `id`、`lng`、`lat`、`alt` 等 serializable row data。不要回传 graphic instance，也不要从 `graphic.position` 读取 lng/lat。
- 现有项目使用 `useWidget()` 时，用它处理 widget activation、deactivation、update events、current widget data 或 cross-widget communication。
- 保留已有 widget options，例如 `autoDisable`、`disableOther`、`group`；它们决定 panel coexistence behavior。
