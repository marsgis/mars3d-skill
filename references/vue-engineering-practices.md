# Vue 工程实践

这份文档记录 Mars3D 在 Vue 工程项目中的实战经验。核心原则是：Vue 响应式数据只保存可序列化的业务状态和 Mars3D 对象 id，不保存 Mars3D 或 Cesium 对象实例。

## Vue 响应式边界

不要把 `mars3d.Map`、`mars3d.layer.GraphicLayer`、`mars3d.graphic.*`、Cesium 对象等实例放进 Vue 的 `ref`、`reactive`、`data`、`computed`、Pinia/Vuex store 或表格数据中。

错误模式：

```ts
const g = ref()
const graphic = new mars3d.graphic.PointEntity({})

g.value = graphic
```

这样会让 Vue 尝试递归代理大型对象图，项目运行越久越容易卡顿。地图对象、图层对象、矢量对象应保存在非响应式变量中，或由 Mars3D 自己的容器管理。

## 只存 id 的关联模式

响应式状态中只保存 `graphic.id`。后续需要操作地图对象时，再通过对应图层查询。

推荐模式：

```ts
const g = ref<string>()

const graphic = new mars3d.graphic.PointEntity({
  position: [116.391, 39.904, 120],
  style: { color: "#ff4d4f", pixelSize: 10 }
})

graphicLayer.addGraphic(graphic)
g.value = graphic.id
```

需要操作时：

```ts
const graphic = graphicLayer.getGraphicById(g.value)

if (graphic) {
  graphic.openPopup?.()
}
```

`graphicLayer` 应是普通变量或由地图模块管理的实例，不要放入 Vue 深度响应式状态。

## 表格与地图标绘联动

在实际项目中，页面表格常常需要和地图标绘对象同步。例如用户在地图上标绘了一个点 `p1`，表格应该新增一条记录，但 `tableData` 里不要保存 `p1` 这个 Graphic 对象。

表格行只保存业务字段和 `id`：

```ts
type PointRow = {
  id: string
  name: string
  lng?: number | string
  lat?: number | string
  alt?: number | string
}

const tableData = ref<PointRow[]>([])

function addPointRow(graphic: mars3d.graphic.PointEntity) {
  tableData.value.push({
    id: graphic.id,
    name: graphic.attr?.name ?? "未命名点"
  })
}
```

删除表格记录时，用 `id` 找回地图中的点，再从图层删除。

```ts
function removePointRow(row: PointRow) {
  const graphic = graphicLayer.getGraphicById(row.id)

  if (graphic) {
    graphicLayer.removeGraphic(graphic)
  }

  tableData.value = tableData.value.filter((item) => item.id !== row.id)
}
```

这种模式可以让 Vue 只管理轻量业务状态，让 Mars3D 管理地图对象生命周期。

## 按具体 Graphic 类型查 API

通过 `graphicLayer.getGraphicById(id)` 找回的对象，要按它真实的 Graphic 类型查 API。

例如地图上标绘的是 `mars3d.graphic.PointEntity`，后续修改样式、弹窗、显隐、编辑状态等操作，应优先参考 `PointEntity` 的 API；通用行为再参考 `BaseGraphic`、`BaseEntity` 或对应父类。

常见判断方式：

```ts
const graphic = graphicLayer.getGraphicById(row.id)

if (graphic instanceof mars3d.graphic.PointEntity) {
  graphic.setStyle({
    color: "#1677ff",
    pixelSize: 12
  })
}
```

如果项目中存在多种矢量对象，表格行可以额外保存业务类型或 Graphic 类型，避免对找回的对象调用不存在的 API。

## 推荐代码片段

地图标绘完成后新增表格记录：

```ts
async function drawPoint() {
  const graphic = await graphicLayer.startDraw({
    type: "point",
    style: {
      color: "#ff4d4f",
      pixelSize: 10
    }
  })
  const point = graphic.centerPoint
  point.format()

  tableData.value.push({
    id: graphic.id,
    name: "标绘点",
    lng: mars3d.Util.formatNum(point.lng, 6),
    lat: mars3d.Util.formatNum(point.lat, 6)
  })
}
```

点绘制完成后不要从 `graphic.position.lng` 或 `graphic.position.lat` 取值；点对象的经纬度优先从 `graphic.centerPoint` 读取。如果只有 Cartesian 坐标，先用 `mars3d.LngLatPoint.fromCartesian(cartesian)` 转换。

根据表格行定位并打开弹窗：

```ts
function locatePoint(row: PointRow) {
  const graphic = graphicLayer.getGraphicById(row.id)

  if (!graphic) {
    return
  }

  graphic.flyTo?.()
  graphic.openPopup?.()
}
```

根据 id 更新地图对象样式：

```ts
function highlightPoint(graphicId: string) {
  const graphic = graphicLayer.getGraphicById(graphicId)

  if (graphic instanceof mars3d.graphic.PointEntity) {
    graphic.setStyle({
      color: "#faad14",
      pixelSize: 14
    })
  }
}
```

记录经验时优先描述“Vue 保存 id，Mars3D 保存对象”的边界；实现代码时优先查具体类 API，例如 `PointEntity`、`PolygonEntity`、`PolylineEntity`。
