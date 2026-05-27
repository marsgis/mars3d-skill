# Coding Patterns / 代码模板

把这些 patterns 作为起点使用，然后通过本地 declarations 或 official API pages 确认精确 option names。

## NPM Map Setup / NPM 地图初始化

```ts
import "mars3d-cesium/Build/Cesium/Widgets/widgets.css"
import "mars3d/mars3d.css"
import * as mars3d from "mars3d"
```

```html
<div id="mars3dContainer" class="mars3d-container"></div>
```

```ts
const map = new mars3d.Map("mars3dContainer", {
  scene: {
    center: { lng: 116.391, lat: 39.904, alt: 90000, heading: 0, pitch: -60 }
  },
  basemaps: [{ name: "Tianditu imagery", type: "tdt", layer: "img_d", show: true }],
  terrain: { show: false },
  control: { compass: true, locationBar: true, zoom: true }
})
```

## Framework Lifecycle / 框架生命周期

```ts
let map: mars3d.Map | undefined

export function initMap(containerId: string) {
  map = new mars3d.Map(containerId, { basemaps: [] })
}

export function destroyMap() {
  map?.destroy()
  map = undefined
}
```

在 Vue 中，`map`、layers、graphics 不要放进 reactive `data`、`computed` 或 stores。使用普通变量；只有在现有项目已经采用时，才考虑 `shallowRef` 或 `markRaw`。

## Config JSON Pattern / config.json 模式

Mars3D examples 和 project templates 经常从 `config.json` 构造 map。

```ts
const options = await fetch("/config/config.json").then((res) => res.json())
const map = new mars3d.Map("mars3dContainer", options.map3d || options)
```

常见 `Map` option sections 包括 `scene`、`terrain`、`basemaps`、`layers`、`control`、`effect`、`thing`、`mouse` 和 `method`。

## Layer Patterns / 图层模式

```ts
const layer = mars3d.LayerUtil.create({
  type: "xyz",
  name: "Business tiles",
  url: "https://example.com/tiles/{z}/{x}/{y}.png",
  show: true
})

map.addLayer(layer)
```

```ts
const graphicLayer = new mars3d.layer.GraphicLayer({ name: "Business graphics" })
map.addLayer(graphicLayer)
```

```ts
const geoJsonLayer = new mars3d.layer.GeoJsonLayer({
  name: "Regions",
  url: "/data/region.geojson",
  symbol: {
    type: "polygon",
    styleOptions: {
      color: "#2f83ff",
      opacity: 0.25,
      outline: true,
      outlineColor: "#ffffff"
    }
  },
  popup: "all",
  flyTo: true
})

map.addLayer(geoJsonLayer)
```

```ts
const tilesetLayer = new mars3d.layer.TilesetLayer({
  name: "3D Tiles",
  url: "/tileset/tileset.json",
  maximumScreenSpaceError: 16,
  flyTo: true
})

map.addLayer(tilesetLayer)
```

## Graphic Patterns / 矢量对象模式

```ts
const point = new mars3d.graphic.PointEntity({
  position: [116.391, 39.904, 120],
  style: {
    color: "#ff4d4f",
    pixelSize: 10,
    outline: true,
    outlineColor: "#ffffff"
  },
  attr: { name: "Example point" }
})

graphicLayer.addGraphic(point)
point.bindPopup("Example point")
```

```ts
const polygon = new mars3d.graphic.PolygonEntity({
  positions: [
    [116.38, 39.9, 0],
    [116.4, 39.9, 0],
    [116.4, 39.92, 0],
    [116.38, 39.92, 0]
  ],
  style: {
    color: "#00b96b",
    opacity: 0.35,
    outline: true,
    outlineColor: "#ffffff",
    highlight: { type: mars3d.EventType.click, opacity: 0.8 }
  }
})

graphicLayer.addGraphic(polygon)
```

## Draw And Edit / 绘制与编辑

```ts
const graphic = await graphicLayer.startDraw({
  type: "polygon",
  style: {
    color: "#3388ff",
    opacity: 0.35,
    outline: true,
    outlineColor: "#ffffff"
  }
})

graphic.startEditing?.()
```

绘制 point 并生成 table data 时，不要读取 `graphic.position.lng` 或 `graphic.position.lat`。Mars3D point graphics 的 `position` 可能是 Cesium position/property 或其他内部结构。经纬高优先使用 `centerPoint`。

```ts
const graphic = await graphicLayer.startDraw({
  type: "point",
  style: {
    color: "#ff4d4f",
    pixelSize: 10,
    outlineColor: "#ffffff",
    outlineWidth: 2
  }
})

const point = graphic.centerPoint
point.format()

const row = {
  id: String(graphic.id),
  lng: mars3d.Util.formatNum(point.lng, 6),
  lat: mars3d.Util.formatNum(point.lat, 6),
  alt: mars3d.Util.formatNum(point.alt, 2)
}
```

如果拿到的是 Cartesian coordinate 而不是 graphic，先转换：

```ts
const point = mars3d.LngLatPoint.fromCartesian(cartesian)
point.format()
```

实现 draw/edit 前，依据 installed version 检查 `GraphicLayer`、`DrawUtil` 和 edit class docs 中的方法名。

## Events / 事件

```ts
function onCameraChanged(event: unknown) {
  console.log("camera changed", event)
}

map.on(mars3d.EventType.cameraChanged, onCameraChanged)
map.off(mars3d.EventType.cameraChanged, onCameraChanged)
```

```ts
graphicLayer.on(mars3d.EventType.click, (event: any) => {
  console.log("graphic click", event.graphic?.attr)
})
```

Mars3D event handlers 支持 `event.stopPropagation()`；当 event 不应继续向上冒泡时使用。

## Camera And Async Data / 相机与异步数据

```ts
map.setCameraView({
  lng: 116.391,
  lat: 39.904,
  alt: 5000,
  heading: 0,
  pitch: -45
})
```

```ts
await layer.readyPromise
await layer.flyTo({ scale: 1.5, duration: 1.2 })
```

调试 invisible data 时，使用 `flyTo: true` 或显式 `flyTo`。

## Coordinates And Tokens / 坐标与 Token

```ts
const point = mars3d.LngLatPoint.fromCartesian(cartesian)
const cartesian = mars3d.LngLatPoint.toCartesian([116.391, 39.904, 120])
```

```ts
mars3d.Token.tianditu = import.meta.env.VITE_TIANDITU_TOKEN
```

在 WGS84、GCJ-02、BD-09、projected 或国内 provider coordinate systems 之间转换时，使用 `PointTrans`、`CRS` 和 `ChinaCRS`。

## Cleanup / 清理

```ts
map.off(mars3d.EventType.cameraChanged, onCameraChanged)
graphicLayer.clear()
map.removeLayer(graphicLayer, true)
map.destroy()
```

使用对象精确支持的 cleanup 方法。有些对象直接 destroy，有些需要由所属 map、layer 或 Thing manager remove。
