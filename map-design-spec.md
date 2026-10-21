# 設計說明：DIMP 地圖（amCharts 5）

> 用途：提供給 AI 產生或檢查 DIMP 地圖 4 種子類型的 amCharts 5 設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**（樣式層、JSON 規則、禁止事項）。
> 實測樣本（2026-10-15）：2534（行政區色塊）、2531（＋氣泡）、2525（＋環形）、2529（經緯度點位）。
> 地圖是所有家族中最複雜的一種（序列管理器約 2,000 行），本文件只描述**建出哪些物件與預設樣式**；視角校正、聚焦、圖例合併等互動邏輯不在範圍內。

---

## 1. 適用範圍

`amChartConfig.type = "map"`，子類型存在 `ChartConfig` 的 `mapSubType`：

| 子類型 | DIMP 名稱 | 疊加的序列 |
| --- | --- | --- |
| `colorRegion` | 地圖（行政區色塊） | 行政區多邊形（色塊）＋選用的數值標籤 |
| `colorRegionWithBubble` | 行政區色塊＋氣泡數值 | 上列＋每區中心一顆氣泡 |
| `colorRegionWithDonut` | 行政區色塊＋環形數值 | 上列＋每區中心一個小環形 |
| `zoomRegionWithPoint` | 經緯度資料 | 鄉鎮多邊形＋縣市框線＋可聚合的座標點 |

## 2. 輸入參數（`DataLogicConfig.amChartConfig.mapConfig`）

```json
{
  "mapSubType": "colorRegion",
  "geodataKey": "<GeoJSON 代號>",
  "scope": "county",
  "regionField": "<行政區欄位>",
  "colorField": "<色塊數值欄位>",
  "labelField": "<標籤欄位，可省略>",
  "bubbleField": "<氣泡數值欄位，限 colorRegionWithBubble>",
  "donutFields": ["<環形欄位 1>", "<環形欄位 2>"],
  "points": [{ "lngColumn": "<經度欄位>", "latColumn": "<緯度欄位>" }],
  "relatedFields": ["<點位提示框附加欄位>"],
  "heatEnabled": true,
  "polygonFill": { "color": "#dadada" },
  "polygonStroke": { "color": "#ffffff", "width": 1.5 },
  "bubbleConfig": { "mode": "singleset", "color": { "color": "#3498db" }, "ranges": [] },
  "donutColors": [],
  "pointColor": { "color": "#ff0000" }
}
```

| 參數 | 說明 |
| --- | --- |
| `geodataKey` | 對應 `window["am5geodata_" + geodataKey]` 的 GeoJSON |
| `scope` | `county`（縣市）或 `town`（鄉鎮）；`zoomRegionWithPoint` 一律用鄉鎮底圖 |
| `regionField` | 對應 GeoJSON 行政區的欄位；`town` 範圍下未選時**只畫底圖、不疊任何資料** |

## 3. 資料整形

產生兩組資料：

**行政區列（`polygonRows`）**：每筆原始資料一列，新增：

| 欄位 | 來源 |
| --- | --- |
| `id` | `regionField` 的值依序比對「縣市名＋鄉鎮名」→「名稱」→「直接當代碼」，換成 GeoJSON 的 feature id |
| `name` | `regionField` 原值；為空時用 GeoJSON 內建名稱 |
| `value` | `colorField` 轉數字 |
| `labelValue` | `labelField` 原值 |
| `bubbleValue` | `bubbleField` 轉數字 |
| `donutSlices` | 每個 `donutFields` 一片 `{ field, value, percent }`，`percent` 為小數 1 位 |

`regionField` 有值但對不到任何 GeoJSON 區域的列會被**靜默過濾**。

**點位列（`pointRows`）**：每組 `points` 一個陣列；經緯度都可解析且落在 `[-180, 180]`／`[-90, 90]` 內才產出 `{ geometry: { type: "Point", coordinates: [lng, lat] }, relatedValues }`。

## 4. MapChart

| 設定 | 值 |
| --- | --- |
| `projection` | `am5map.geoMercator()` |
| `panX`／`panY` | `"translateX"`／`"translateY"`；`zoomRegionWithPoint` 的 `panX` 為 `"rotateX"` |
| `wheelX`／`wheelY` | `"none"`／`"zoom"` |
| `maxZoomLevel` | `5000` |
| `zoomControl` | `am5map.ZoomControl`，顯示 Home 按鈕 |
| 容器背景 | `#FFFFFF` |
| 初始視角 | 預設依 GeoJSON 範圍以 `zoomToGeoBounds` 自動對焦；`mapConfig.viewportEnabled` 開啟時改用 `viewCenter`／`viewZoomLevel` 指定（本文件未展開這段邏輯） |

## 5. 序列

### 5.1 行政區多邊形（四種子類型都有）

| 設定 | 值 |
| --- | --- |
| 類別 | `am5map.MapPolygonSeries` |
| `geoJSON` | 依 `geodataKey` |
| `valueField` | `"value"` |
| `calculateAggregates` | `true` |
| `reverseGeodata` | 鄉鎮 GeoJSON 為 `true`（頂點順序相反） |
| `mapPolygons.template` | `fill` = `polygonFill.color`（預設 `#dadada`）、`stroke` = `polygonStroke.color`（預設 `#ffffff`）、`strokeWidth` = `polygonStroke.width`（預設 `1.5`） |
| 色塊（`heatEnabled ≠ false`，連續色階） | `heatRules`：`target` = `mapPolygons.template`、`dataField: "value"`、`key: "fill"`、`min`／`max` = 起訖色 |
| 色塊（分段色階 `rangeset`） | 改用 `fill` adapter 依區間由上到下比對，都沒命中時用底色 |
| 提示框 | `tooltipHTML` adapter：標題為行政區名，下方表格列出各欄位值（缺值的列不顯示） |

起訖色優先序：地圖樣式的 `heatConfig.start`／`end` → `ChartConfig.colors` 第一色／最後一色 → `#d3a29f`／`#6f0600`。實測樣本為 `#c8a67f`／`#5c3a21`。

### 5.2 數值標籤（有 `labelField` 時）

`MapPointSeries`（`polygonIdField: "id"`），每區中心一個 `Container`：背景 `RoundedRectangle`（白底、`fillOpacity: 0.9`、黑框 `1`、`cornerRadius: 4`）、padding 上下 `4`／左右 `6`；內含 `Label`：`text: "{labelValue}"`、`fontSize: 12`、`fontWeight: "bold"`、黑字。`labelValue` 為空時不建立。

### 5.3 氣泡（`colorRegionWithBubble`）

`MapPointSeries`（`polygonIdField: "id"`），每區中心一個 `Circle`：

| 項目 | 值 |
| --- | --- |
| 半徑 | 依 `bubbleValue` 在所有區的最小～最大值線性對應到 `5`～`25`；全部相同時為 `15` |
| 顏色 | `bubbleConfig.mode = "singleset"` → `bubbleConfig.color`；`"rangeset"` → 依區間比對，沒命中用 `bubbleConfig.color`；預設 `#3498db` |
| 其他 | `fillOpacity: 0.6`、白框 `1`、`tooltipHTML` 同行政區 |

### 5.4 環形（`colorRegionWithDonut`）

`MapPointSeries`（`polygonIdField: "id"`），每區中心一個 `Container`，內含多個 `am5.Slice`（**不能**用 `PieChart`，放進地圖 bullet 不會渲染）：

| 項目 | 值 |
| --- | --- |
| 每片 | `radius: 20`、`innerRadius: 10`、從 `-90` 度開始，角度 = 該片值 ÷ 總和 × 360 |
| 略過 | 值無效、≤ 0，或 `donutFieldsEnabled[i] = false` 的片 |
| 顏色 | `donutColors[i]`；未設定時依序 `#5b9bd5`、`#70ad47`、`#ffc000`、`#ff6b6b`、`#4ecdc4`、`#45b7d1`、`#a29bfe`、`#fd79a8` |
| 其他 | 白框 `1`、`tooltipHTML` 同行政區 |

### 5.5 經緯度點位（`zoomRegionWithPoint`）

依序建立三個序列：

| 序列 | 設定 |
| --- | --- |
| 鄉鎮多邊形 | 同 5.1 |
| 縣市框線 | `MapPolygonSeries`：`fill: #000000`、`fillOpacity: 0`、`stroke: #333333`、`strokeWidth: 1.5`、`strokeOpacity: 0.85` |
| 座標點 | `am5map.ClusteredPointSeries`：`minDistance: 20`、`stopClusterZoom: 4500` |

| 點位 bullet | 設定 |
| --- | --- |
| 單點 | `Circle`：`radius: 6`、`fill` = `pointColor.color`（預設 `#ff0000`）、白框 `2`；`tooltipHTML` 列出座標與 `relatedFields` 的值 |
| 聚合圈 | `Circle`：`radius: 14`、`fillOpacity: 0.85`、白框 `2`；中央 `Label`：`text: "{value}"`（點數）、白字、`fontSize: 11`、粗體；點擊時 `zoomToCluster()` |

## 6. 圖例

| 圖例 | 條件與設定 |
| --- | --- |
| 熱力圖例 | 色塊啟用時：`am5.HeatLegend`，`stepCount` 預設 `8`；`position` 為 `top`／`bottom` 時水平（長 300px），否則垂直（長 200px）；兩端文字為實際最低／最高值（小數 2 位），無資料時顯示 `Lowest`／`Highest` |
| 序列圖例 | 氣泡：1 項（`bubbleField`）；環形：每個 `donutFields` 1 項；點位：每組 `points` 1 項；純色塊：無 |

## 7. JSON 化限制

GeoJSON、`tooltipHTML` adapter、分段色階 adapter、聚合圈點擊、視角自動對焦都**無法寫進 JSON**。地圖的 JSON 設計檔只適合保存 `mapConfig` 參數本身，實際物件仍應由程式依本文件建立。
