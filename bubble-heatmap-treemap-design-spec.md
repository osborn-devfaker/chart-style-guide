# 設計說明：DIMP 氣泡圖、熱力圖、樹狀圖（amCharts 5）

> 用途：提供給 AI 產生或檢查 DIMP「其他圖表」群組中三種圖表的 amCharts 5 設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**（樣式層、數字格式、配色、線型數值標籤、圖例、XY 共用預設、JSON 規則）。
> 實測樣本（2026-10-15）：2307（氣泡）、2322（熱力）、2398（樹狀）。

---

## A. 氣泡圖（`amChartConfig.type = "bubble"`）

### A.1 輸入參數

```json
{
  "categoryField": "<分類欄位（X）>",
  "valueFields": ["<數值欄位（Y）>"],
  "sizeField": "<決定氣泡大小的欄位>",
  "oppositeFields": []
}
```

### A.2 資料整形

每列保留 `categoryField` 與數值欄位，並新增 `value` = `sizeField` 的值（供氣泡半徑使用）。

### A.3 建構

軸與圖表同共用說明第 9 節（直式）。每個數值欄位一個序列：

| 設定 | 值 |
| --- | --- |
| 類別 | `LineSeries`（以 bullet 畫氣泡，線條不顯示） |
| `valueYField`／`categoryXField` | 數值欄位／`categoryField` |
| `valueField` | `"value"`（即 `sizeField`） |
| `calculateAggregates` | `true` |
| 線條 | `strokes.template.strokeOpacity: 0` |
| `tooltip.labelText` | `"<數值欄位>:{valueY}[/]\n<sizeField>:{value}"` |
| 游標 | `XYCursor.snapToSeries` = 全部氣泡序列（只顯示離滑鼠最近的一個提示框） |

### A.4 氣泡

樣式層會把氣泡重建為：

| 項目 | 值 |
| --- | --- |
| 圓點 `Circle` | `fill` = 序列色、`stroke` = 序列色、`strokeWidth: 5`；`locationY: 1` |
| 半徑 | `heatRules`：`target` = 圓點 template、`dataField: "value"`、`key: "radius"`、`min: 3`、`max: 20` |
| 數值標籤 | 共用說明第 6.2 節的 Label（在氣泡上方） |

---

## B. 熱力圖（`amChartConfig.type = "heatmap"`）

### B.1 輸入參數

```json
{
  "categoryField": "<列分類欄位（X）>",
  "valueFields": ["<欄位 1>", "<欄位 2>"]
}
```

### B.2 資料整形

把寬表轉成長表：每一列 × 每個數值欄位產生一筆 `{ xField: <categoryField 的值>, yField: <數值欄位名稱>, value: <該欄位的值> }`。也就是 **X 軸是分類值、Y 軸是欄位名稱**。

### B.3 XYChart

| 設定 | 值 |
| --- | --- |
| `panX`／`panY` | `false`／`false` |
| `wheelX`／`wheelY` | `"none"`／`"none"` |
| padding | 左 `0`、右 `20`、上下 `16` |
| 游標 | 無 |

### B.4 軸

| 軸 | 設定 |
| --- | --- |
| X | `CategoryAxis`（`categoryField: "xField"`）；renderer `opposite: true`（**畫在上方**）、`minGridDistance: 60`；標籤 `oversizedBehavior: "wrap"`、`textAlign: "center"` |
| Y | `CategoryAxis`（`categoryField: "yField"`）；renderer `inversed: true`（第一個欄位在最上方） |

### B.5 序列

| 設定 | 值 |
| --- | --- |
| 類別 | `ColumnSeries` |
| `categoryXField`／`categoryYField`／`valueField` | `"xField"`／`"yField"`／`"value"` |
| `calculateAggregates` | `true` |
| `columns.template` | `width: 100%`、`height: 100%`、`strokeOpacity: 1`、`strokeWidth: 1`、`fontSize: 14`、`tooltipText: "{categoryX}:{value}"` |
| 色階 `heatRules` | `target` = `columns.template`、`dataField: "value"`、`key: "fill"`、`min: #FFFB77`（最低值，淺黃）、`max: #FE131A`（最高值，紅） |

⚠️ 格子**填色**固定用上表兩色的色階，不受 `ChartConfig.colors` 影響；但配色策略仍會把 `colors[0]` 套到序列與 `columns.template` 的 `stroke`，所以**格子框線是 `colors[0]`**（實測 2322 為綠色 `#6de082`）。

### B.6 熱力圖例

`am5.HeatLegend` 放在 `chart.bottomAxesContainer`：`orientation: "horizontal"`、`startColor: #FE131A`、`endColor: #FFFB77`；資料驗證後 `startValue` = 最高值、`endValue` = 最低值（圖例由高到低）；滑鼠移到格子上時以 `showValue()` 標出該格的值。一般圖例不建立。

---

## C. 樹狀圖（`amChartConfig.type = "treemap"`）

### C.1 輸入參數

```json
{
  "categoryField": "<第一層分類欄位>",
  "valueFields": ["<欄位 1>", "<欄位 2>"]
}
```

### C.2 資料整形

建成兩層樹：

```
{ name: "Root", children: [
    { name: <分類值>, children: [
        { name: <數值欄位名稱>, value: <數字>, parentName: <分類值> }, ...
    ] }, ...
] }
```

第一層 = 分類值，第二層 = 各數值欄位。

### C.3 Treemap

`am5hierarchy.Treemap` 直接放在圖表容器的 `children`（**不是**放在 XYChart 裡）：

| 設定 | 值 |
| --- | --- |
| `valueField`／`categoryField`／`childDataField` | `"value"`／`"name"`／`"children"` |
| `downDepth`／`topDepth`／`initialDepth`／`upDepth` | `1`／`1`／`1`／`0` |
| `singleBranchOnly` | `true` |
| `layoutAlgorithm` | `"squarify"` |
| `nodePaddingOuter`／`nodePaddingInner` | `0`／`0` |
| `nodes.template` | `tooltipText: "{parentName}:{value}"`、`interactive: true` |
| 進場 | `appear(1000, 100)` |

### C.4 配色

第 i 個第一層節點底下的所有子節點填 `ChartConfig.colors[i % colors.length]`（同一分類同色）。
