# API Navigation / API 导航

把 Mars3D symbol 或功能需求映射到官方 API URL。涉及 exact signatures、option names、method names 或版本兼容时，先查 installed type declarations，例如 `node_modules/mars3d`、`mars3d.d.ts`；官方页面用于补充确认。

## Lookup Flow / 查找流程

1. 代码或问题中有明确 class/enum 时，按 URL Patterns 生成完整官方 API URL。
2. 只有功能描述时，先用下方分类索引选择最可能的 class，再生成 URL。
3. 如果页面不存在、页面版本和 installed package 不一致，回到 `https://mars3d.cn/api/` 搜索，并以 installed declarations 为准。

## URL Patterns / URL 规则

Class page:

`https://mars3d.cn/api/{ClassName}.html`

取完整命名空间最后一段作为 `{ClassName}`：

- `mars3d.Map` -> `Map` -> `https://mars3d.cn/api/Map.html`
- `new mars3d.graphic.PointEntity(...)` -> `PointEntity` -> `https://mars3d.cn/api/PointEntity.html`
- `mars3d.layer.GeoJsonLayer` -> `GeoJsonLayer` -> `https://mars3d.cn/api/GeoJsonLayer.html`

Enum/global page:

`https://mars3d.cn/api/global.html#{Name}`

取最后一段作为 `{Name}`：

- `mars3d.EventType` -> `EventType` -> `https://mars3d.cn/api/global.html#EventType`
- `LayerType` -> `LayerType` -> `https://mars3d.cn/api/global.html#LayerType`
- `CRS` -> `CRS` -> `https://mars3d.cn/api/global.html#CRS`

下方分类索引中的 PascalCase 名称默认都是 `{ClassName}`。`EventType`、`LayerType`、`GraphicType`、`MaterialType`、`CRS`、`ChinaCRS`、`TerrainType`、`ThingType`、`EffectType`、`ControlType` 等属于 enum/global。小写字段、事件名和报错词只是搜索关键词，不要直接拼 class page URL。

## Core Pages / 核心页面

- `Map`: 地图初始化、scene、terrain、basemaps、layers、controls、effects、things、mouse settings、camera。
- `BaseClass`: 通用 event API，例如 `on`、`off`、`fire`。
- `BaseLayer`、`BaseTileLayer`、`BaseGraphicLayer`: 通用 layer、visibility、add/remove 行为。
- `BaseGraphic`、`BaseEntity`、`BasePrimitive`: 通用 graphic lifecycle、style、attr、popup、tooltip、context menu、highlight、editing 行为。
- `LngLatPoint`、`LngLatArray`、`PointTrans`、`PointUtil`、`PolyUtil`: 坐标、投影转换和 geometry helpers。
- `LayerUtil`、`GraphicUtil`、`MaterialUtil`、`ThingUtil`、`ControlUtil`、`EffectUtil`: factory、registry、根据配置创建对象。
- `Token`: 第三方服务 token 配置。

## Common Layers / 常见图层

用户问“加载某类数据/服务/图层”时，从这里选择 class page。每个名称都按 `https://mars3d.cn/api/{ClassName}.html` 查找。

- Terrain and tiles: `TerrainLayer`、`XyzLayer`、`WmsLayer`、`WmtsLayer`、`ArcGisLayer`、`ArcGisTileLayer`、`TdtDmLayer`。
- Vector data: `GraphicLayer`、`GeoJsonLayer`、`CzmGeoJsonLayer`、`KmlLayer`、`CzmlLayer`、`WfsLayer`、`Wfs2Layer`、`ArcGisWfsLayer`。
- 3D data: `TilesetLayer`、`ModelLayer`、`I3SLayer`、`S3MLayer`、`OsmBuildingsLayer`。
- Visualization: `HeatLayer`、`EchartsLayer`、`MapVLayer`、`WindLayer`、`CanvasWindLayer`。
- Management: `GroupLayer`、`LodGraphicLayer`、`Lod2GraphicLayer`。

## Common Graphics / 常见矢量对象

- Point-like: `PointEntity`、`PointPrimitive`、`BillboardEntity`、`LabelEntity`、`DivGraphic`、`DivBillboardEntity`、`CanvasLabelEntity`。
- Lines and areas: `PolylineEntity`、`PolylinePrimitive`、`PolygonEntity`、`RectangleEntity`、`CircleEntity`、`EllipseEntity`、`WallEntity`、`CorridorEntity`。
- 3D objects: `BoxEntity`、`CylinderEntity`、`EllipsoidEntity`、`PlaneEntity`、`ModelEntity`、`ModelPrimitive`。
- Effects/special: `Water`、`DiffuseWall`、`ScrollWall`、`ParticleSystem`、`Video2D`、`Video3D`。
- Measurement graphics: `DistanceMeasure`、`AreaMeasure`、`HeightMeasure`、`VolumeMeasure`、`SectionMeasure`。

普通业务矢量优先选择 `Entity` classes。大数据量或渲染压力大的场景，在确认 class docs 后再选择 `Primitive` 或 combine classes。

## Analysis, Effects, And Controls / 分析、特效与控件

- Analysis/Thing: `Measure`、`Sightline`、`ViewShed`、`ViewDome`、`TerrainClip`、`TerrainFlat`、`TilesetClip`、`TilesetFlat`、`Shadows`、`Slope`、`FloodByGraphic`。
- Camera/roam: `KeyboardRoam`、`FirstPersonRoam`、`RotatePoint`、`RotateOut`、`CameraHistory`。
- Effects: `Rain`、`Snow`、`Fog`、`Bloom`、`Outline`、`Brightness`、`ColorCorrection`、`DepthOfField`。
- Controls: `Compass`、`LocationBar`、`DistanceLegend`、`Zoom`、`BaseLayerPicker`、`OverviewMap`、`MapSplit`、`MapCompare`、`Toolbar`。
