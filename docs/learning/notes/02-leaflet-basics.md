---
name: leaflet-basics
description: Leaflet 示例页面解析及基础概念
type: reference
---

# Leaflet 示例页面解析

本页面位于 `practice/01-基础地图/index.html`，演示了如何在浏览器中快速加载天地图底图并渲染一个 GeoJSON 点。下面按代码块逐段说明其作用。

## 1️⃣ HTML 结构

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <title>基础地图示例</title>
    <link
      rel="stylesheet"
      href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
    />
    <style>
      #map {
        height: 100vh;
        margin: 0;
      }
    </style>
  </head>
  <body>
    <div id="map"></div>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
      …
    </script>
  </body>
</html>
```

- `#map` 是容器 div，Leaflet 会把地图渲染进这个元素。
- 引入 Leaflet CSS/JS（通过 CDN），不需要本地安装。

## 2️⃣ 初始化地图对象

```js
const map = L.map('map').setView([30, 120], 5);
```

- `L.map('map')` 创建地图实例并绑定到 `id="map"` 的 DOM 元素。
- `setView([lat, lng], zoom)` 设置初始中心为北京附近（纬度 30°, 经度 120°）和缩放级别 5。

## 3️⃣ 添加底图图层（TileLayer）

```js
L.tileLayer(
  'http://t{s}.tianditu.gov.cn/vec_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=vec&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=',
  {
    attribution: '&copy; 天地图',
    subdomains: '01234567',
  },
).addTo(map);
```

- `L.tileLayer(urlTemplate, options)` 用来加载瓦片切片。
- `urlTemplate` 中的 `{s}`、`{z}`、`{x}`、`{y}` 会在运行时被 Leaflet 替换为对应的子域、缩放级别、列、行。
- `subdomains: '01234567'` 告诉 Leaflet 可以使用 `t0`‑`t7` 八个子域，以分摊请求负载。
- `attribution` 用于合法显示数据来源。

## 4️⃣ 加载并显示 GeoJSON 数据

```js
const geojson = {
  type: 'FeatureCollection',
  features: [
    {
      type: 'Feature',
      geometry: { type: 'Point', coordinates: [117.12, 36.65] },
      properties: { name: '示例点' },
    },
  ],
};

L.geoJSON(geojson, {
  pointToLayer: function (feature, latlng) {
    return L.marker(latlng);
  },
  onEachFeature: function (feature, layer) {
    if (feature.properties && feature.properties.name) {
      layer.bindPopup(feature.properties.name);
    }
  },
}).addTo(map);
```

- `L.geoJSON` 将任意符合 GeoJSON 规范的对象转为 Leaflet 图层。
- `pointToLayer` 决定点要素的表现形式，这里返回 `L.marker`（默认蓝色图钉）。
- `onEachFeature` 为每个要素绑定弹窗，点击图钉会弹出 `properties.name`。

## 5️⃣ 基础 Leaflet 概念速记

| 概念                          | 说明                                                                                                            |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Map**                       | 地图实例，负责管理视图、交互、坐标系统。创建方式 `L.map(containerId)`。                                         |
| **TileLayer**                 | 瓦片图层，通常从公开的 WMTS/XYZ 服务获取瓦片。通过 `L.tileLayer(url, options)` 添加。                           |
| **Marker / Circle / Polygon** | 基本矢量要素，用来标记点、画圆或多边形。                                                                        |
| **LayerControl**              | `L.control.layers` 提供底图与覆盖层切换 UI。                                                                    |
| **CRS**                       | 坐标参考系统，默认 `L.CRS.EPSG3857`（Web Mercator）。可自行设定其他投影。                                       |
| **Event**                     | Leaflet 为地图、图层提供丰富的交互事件（`click`, `zoomend`, `moveend` 等），可在 `map.on('click', fn)` 中监听。 |
| **GeoJSON**                   | GeoJSON 是一种 JSON 格式的空间要素描述，Leaflet 可直接用 `L.geoJSON` 读取并渲染。                               |

## 6️⃣ 扩展思路（可在后续练习中尝试）

1. **添加图层切换**：使用 `L.control.layers` 把天地图矢量层、影像层切换为不同 `L.tileLayer` 实例。
2. **自定义 Marker 图标**：通过 `L.icon` 定义自己图片的图标，替换默认蓝色图钉。
3. **交互控件**：添加比例尺 `L.control.scale()`、全屏 `leaflet.fullscreen` 插件等。
4. **更多 GeoJSON**：加载外部 GeoJSON 文件（`fetch(...).then(r=>r.json())`）或使用 `L.geoJSON(url)` 动态加载。
5. **投影转换**：如果需要经纬度转为天地图的投影坐标，可配合 `proj4js` 实现（天地图本身已是 Web Mercator）。

---

> 本笔记旨在帮助您快速理清示例代码结构与 Leaflet 基础概念，后续可在 `practice/02-Leaflet-API` 继续探索更丰富的 API。
