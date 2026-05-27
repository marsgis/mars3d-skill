# API Errors

帮助解决mars3d使用api发生的错误问题

## 确定用户是哪个api发生错误
假如是下面代码
```js
  const graphicLayer = new mars3d.layer.GraphicLayer()
  map.addLayer(graphicLayer)
  const graphic = new mars3d.graphic.PointEntity({
    position: [116.244399, 30.920459, 573.6],
    style: {
      color: "#ff0000"
    },
    attr: { remark: "示例1" }
  })
  graphicLayer.addGraphic(graphic)
```
如果用户说的图层的显示隐藏失效
```js
graphicLayer.show = false
```
那么，就是`GraphicLayer`这个api错误
如果用户说的点的颜色不对，那么就是`PointEntity`这个api错误

## 用户版本问题造成api使用错误

通过查阅更新文档 `http://mars3d.cn/docs/guide/change/` ，搜索发生错误的api，可以知道版本相关问题

1. 如果没有加载 `references/integration.md`，加载 `references/integration.md`，判断用户的版本，因为有的api，在旧版本可以使用，但是新版版本已经移除了,比如
  案例：用户使用的 map.keyboardRoam = true ，根据更新文档，在3.9.0版本已经移除map.keyboardRoam，解耦改为按 需构造mars3d.thing.KeyboardRoam对象

2. 用户使用的是老版本，但是使用的api是新版本才加上的
   案例：用户使用new mars3d.graphic.HollowCylinder，但是根据更新文档，这个api是3.11.2才新增加的

3. 用户版本有问题，但是新版本已经解决了这个问题，建议用户更新版本即可
   案例： 用户在3.11.1中使用DivPlane无法聚合，但是根据更新文档，3.11.2中已经修复了这个问题

## 版本没有问题，但仍然报错

1. 询问用户是否已经下载官方示例代码 `mars3d-vue-example`。

2. 如果用户没有下载，建议先下载官方示例仓库：

```bash
git clone git@gitee.com:marsgis/mars3d-vue-example.git
```

3. 如果用户已经下载，让用户提供示例仓库的本地地址，例如 `E:\temp\mars3d-vue-example`。

4. 得到本地地址后，调用工具在该仓库中搜索目标 api。优先搜索完整构造语句，例如：

```powershell
rg -n "new mars3d\.graphic\.PolylineEntity" "E:\temp\mars3d-vue-example"
```

5. 搜索到示例后，优先查看命中的 `map.js` 或 `map.ts`。整理内容时需要说明上下文，例如：
   - 示例文件路径。
   - 示例所在功能或目录名称。
   - 相关的图层创建、`map.addLayer`、`graphicLayer.addGraphic`、生命周期函数等必要代码。
   - 目标 api 的关键配置项，比如 `positions`、`style`、`materialType`、`materialOptions`。

6. 归纳示例代码时删除无效或干扰判断的代码：
   - 删除大量坐标点，只保留 `positions: [...]` 或少量代表性点位。
   - 删除与目标 api 无关的按钮、表格、弹窗、随机数据、演示说明、批量生成代码。
   - 删除重复示例中的重复结构。
   - 保留能解释 api 用法的图层创建、对象构造、样式配置、添加到图层等关键代码。

7. 如果同一个 `map.js` 或多个示例中出现很多相似写法，只保留有代表性的一份。比如多个 `new mars3d.graphic.PolylineEntity` 只是坐标、颜色、宽度不同，代码结构和 `materialType` 一样，则只保留一份，并在总结中说明“其他示例仅坐标、颜色或宽度不同，已合并”。

8. 如果示例差异会影响用户问题，需要分别保留。比如 `materialType` 不同、图层类型不同、是否启用编辑/聚合/事件不同、生命周期写法不同，不能合并成一份。

9. 整理完成后，不要直接把全部示例强行加载进上下文。先把整理好的内容、来源路径和精简后的上下文告诉用户大小，让用户判断是否需要加载进入上下文。

## 排查"参数问题"还是"API本身问题"：将含有逻辑的代码转换为不含逻辑的代码

当 API 报错或表现异常时，需要先判断是 API 本身的 bug，还是用户传入的参数、变量、业务流程导致的问题。方法是将用户含有逻辑的代码转换为不含逻辑的硬编码代码，只验证目标 API 和目标属性本身。

### 判断标准

- 硬编码代码能正常运行：是参数、变量或业务流程问题，检查用户代码中的动态值。
- 硬编码代码同样报错：是 API 本身或当前版本的问题，需要进一步查阅 API、版本更新文档或官方示例。

### 转换规则

先定位用户描述中真正受影响的 API 和属性路径，再做转换。

- 例如“删除 billboard 对象时会影响 `PointEntity` 对象的 text 显示”，目标 API 是 `mars3d.graphic.PointEntity`，目标属性路径是 `style.label.text`。
- 如果用户说的是弹窗不显示，目标可能是 `GraphicLayer.bindPopup`、`graphic.bindPopup`、`tooltip` 或 `popup`，不要误判成点对象本身。

将含有变量、循环、条件判断、外部数据的代码，转换为纯字面量、无逻辑的简单代码。

需要移除的内容：

- 循环语句：`for`、`while`、`forEach`、`map`。
- 变量引用：`wp.lng`、`wp.lat`、`wp.height`、`UI_CONFIG.POINT` 等。
- 动态表达式：`wp.index + 1`、`String(n)`、模板字符串、计算属性等。
- 条件判断：`if/else`、三元表达式、短路表达式。
- 外部数据源：`waypoints` 数组、接口返回值、配置常量、store、组件 props。
- 展开运算符：`...UI_CONFIG.LINE`、`...pointStyle` 等。
- 和目标 API 无关的业务逻辑：Vue 组件、弹窗、事件监听、右键菜单、接口请求、异步高度查询、鼠标绘制交互、批量生成、批量删除。

需要保留的内容：

- API 构造语句本身，例如 `new mars3d.graphic.PointEntity({...})`。
- 和问题直接相关的参数，例如 `position` / `positions`、`style`、`style.label.text`、`attr`。
- 图层创建和添加逻辑，例如 `new mars3d.layer.GraphicLayer()`、`map.addLayer(graphicLayer)`、`graphicLayer.addGraphic(graphic)`。
- 生命周期函数框架，例如 `onMounted`、`onUnmounted`，方便用户直接放回官方示例中测试。

### 转换示例

#### 场景：`PointEntity` 的 `text` 属性显示异常

转换前，代码含有循环、外部数组、配置常量、变量引用和动态表达式：

```js
const waypoints = [
  { lat: 28.2287741, lng: 112.9329878, height: 160, speed: 15, index: 0 },
  { lat: 28.2310714, lng: 112.9329878, height: 160, speed: 15, index: 1 }
]

const UI_CONFIG = {
  POINT: { color: "#05DF72", pixelSize: 22, outline: false, visibleDepth: false }
}

for (let i = 0; i < waypoints.length; i++) {
  const wp = waypoints[i]
  const point = new mars3d.graphic.PointEntity({
    position: [wp.lng, wp.lat, wp.height],
    hasEdit: false,
    style: {
      ...UI_CONFIG.POINT,
      disableDepthTestDistance: Number.POSITIVE_INFINITY,
      label: {
        text: String(wp.index + 1),
        color: "#ffffff",
        font_size: 14,
        horizontalOrigin: mars3d.Cesium.HorizontalOrigin.CENTER,
        verticalOrigin: mars3d.Cesium.VerticalOrigin.CENTER,
        disableDepthTestDistance: Number.POSITIVE_INFINITY
      }
    }
  })
  graphicLayer.addGraphic(point)
}
```

转换后，只保留一个 `PointEntity`，所有参数都改为硬编码字面量：

```js
const graphicLayer = new mars3d.layer.GraphicLayer()
map.addLayer(graphicLayer)

const graphic = new mars3d.graphic.PointEntity({
  position: [117.194842, 31.831489, 41.21521],
  style: {
    color: "#05DF72",
    pixelSize: 22,
    visibleDepth: false,
    disableDepthTestDistance: Number.POSITIVE_INFINITY,
    label: {
      text: "1",
      color: "#ffffff",
      font_size: 14,
      horizontalOrigin: mars3d.Cesium.HorizontalOrigin.CENTER,
      verticalOrigin: mars3d.Cesium.VerticalOrigin.CENTER,
      disableDepthTestDistance: Number.POSITIVE_INFINITY
    }
  }
})
graphicLayer.addGraphic(graphic)
```

### 排查步骤

1. 从用户描述中确定目标 API 和目标属性路径，例如 `PointEntity.style.label.text`。
2. 从用户代码中找到目标 API 的构造语句，例如 `new mars3d.graphic.PointEntity({...})`。
3. 提取构造语句中涉及问题的参数，例如 `position`、`style`、`style.label`、`attr`。
4. 将参数中的变量替换为硬编码字面量：
   - `position: [wp.lng, wp.lat, wp.height]` -> `position: [112.9329878, 28.2287741, 160]`
   - `text: String(wp.index + 1)` -> `text: "1"`
   - `...UI_CONFIG.POINT` -> 逐个写出 `color: "#05DF72"`、`pixelSize: 22` 等。
5. 删除循环，只保留一条构造语句。
6. 删除外部数据源和配置常量。
7. 保留图层创建、`map.addLayer` 和 `graphicLayer.addGraphic` 调用。
8. 让用户在官方示例仓库中替换对应 `map.js` 或 `map.ts` 后测试。
9. 根据测试结果判断问题来源：
   - 硬编码正常：检查用户变量的值是否正确，例如 `wp.index`、`wp.lng`、`wp.lat`、`wp.height` 是否为 `undefined`，展开后的配置是否覆盖了关键参数。
   - 硬编码仍异常：按 API 本身问题继续排查，查阅更新文档、API 文档、官方示例，必要时建议提 issue。

### 对象交互场景：删除 A 对象影响 B 对象

如果用户问题明确是“删除 A 对象影响 B 对象”“添加 A 后 B 异常”这类对象交互，再生成第二份“目标对象 + 干扰对象”的最小对照代码。

- B 是目标对象，必须先保证 B 单独加载正常。
- A 是干扰对象，只保留最小构造和最小删除逻辑。
- 删除逻辑只保留 `graphicLayer.removeGraphic(xxx)` 或 `xxx.remove()`，不要混入业务定时器之外的其他流程。

```js
const point = new mars3d.graphic.PointEntity({
  position: [117.194842, 31.831489, 41.21521],
  style: {
    color: "#05DF72",
    pixelSize: 22,
    label: {
      text: "1",
      color: "#ffffff",
      font_size: 14,
      horizontalOrigin: mars3d.Cesium.HorizontalOrigin.CENTER,
      verticalOrigin: mars3d.Cesium.VerticalOrigin.CENTER
    }
  }
})
graphicLayer.addGraphic(point)

const billboard = new mars3d.graphic.BillboardEntity({
  position: [117.194842, 31.831489, 80],
  style: {
    image: "https://data.mars3d.cn/img/marker/mark-blue.png",
    width: 30,
    height: 30
  }
})
graphicLayer.addGraphic(billboard)

setTimeout(() => {
  graphicLayer.removeGraphic(billboard)
}, 5000)
```

如果只加载 B 正常、加入 A 的增删后异常，重点排查同图层对象交互、删除方式、对象销毁、渲染顺序、label 附带对象、深度检测、`combine` 等参数。

### 回复用户时需要说明

- 目标 API 是什么，目标属性路径是什么。
- 删除了哪些业务逻辑。
- 哪些参数被硬编码保留下来。
- 让用户先测试哪份最小代码。
- 不同测试结果分别代表“参数问题”还是“API 本身问题”。
