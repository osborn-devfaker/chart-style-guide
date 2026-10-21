# 設計說明：DIMP KPI 指標圖（迷你趨勢圖）（amCharts 5 ＋ HTML）

> 用途：提供給 AI 產生或檢查 DIMP「KPI」圖表的設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**（樣式層、配色、JSON 規則）。
> 實測樣本（2026-10-15）：2522。

---

## 1. 組成

KPI 由兩個**獨立**的部分組成，產生設計時要分開處理：

| 部分 | 技術 | 內容 |
| --- | --- | --- |
| 指標文字 | **HTML**（不是 amCharts） | 本期值、上期值、增減值、期別，插在圖表容器**之前**的 `<div class="sparkline-label">` |
| 迷你趨勢圖 | amCharts `XYChart` | 一條平滑填色曲線，無軸線、無格線、無互動 |

判定方式：`DataLogicConfig.amChartConfig.type = "sparklines"`；指標文字只在 `ChartConfig.chart.type = "area"` 且 `ChartConfig.chart.sparkline.enabled = true` 時產生。

## 2. 輸入參數

```json
{
  "valueField": "<數值欄位>",
  "trendField": "<期別欄位，可省略>",
  "sortColumn": "<排序欄位，可省略>",
  "sortDirection": "asc",
  "template": "<含預留字的 HTML 範本>",
  "valueStyle": { "scales": 0, "thousands": true },
  "preValueStyle": { "scales": 0, "thousands": true },
  "trendValueStyle": { "scales": 1, "thousands": false, "percentage": true }
}
```

| 參數 | 存檔來源（`DataLogicConfig`） |
| --- | --- |
| `valueField`／`trendField` | `ySeries[0].column`／`ySeries[1].column` |
| `sortColumn`／`sortDirection` | `sortConfig.colName`／`sortConfig.direction` |
| `template` | `textboxContent`（Base64 編碼的 HTML） |
| `*Style` | `trending.value.style`／`trending.preValue.style`／`trending.trendValue.style` |

## 3. 指標文字

### 3.1 計算

1. 有 `trendField` 且有 `sortColumn` 時，先依排序欄位排序。
2. 取 `valueField` 整欄轉數字：本期 = 最後一筆、上期 = 倒數第二筆。
3. 增減值：`percentage = true` → `(本期 − 上期) / 上期 × 100`（上期為 0 或空值時視為無值）；否則 `本期 − 上期`。
4. 期別：有 `trendField` 時取對應列的值，空值顯示 `[--]`；沒有 `trendField` 時為空字串。

### 3.2 格式

`scales` 為 0～9 → 固定小數位；其他 → 原值。`thousands = true` → 整數部分加千分位。**任何無法計算的值一律顯示 `[--]`**。增減值為百分比且可解析為數字時，結尾加 `%`。

### 3.3 範本預留字

`{value}`、`{previousValue}`、`{trendValue}`、`{currentPeriod}`、`{previousPeriod}`、`{colNameFirst}`、`{colNameSecond}`，各取代**第一次**出現的位置。

⚠️ 取代後的字串以 `innerHTML` 寫入。AI 產生範本時只放靜態 HTML 與預留字，**不要**把資料值直接拼進範本。

## 4. 迷你趨勢圖

### 4.1 資料整形

每列轉成 `{ category: <列序號 0, 1, 2…>, value: <valueField 的值> }`。X 軸是**列序號**，不是日期或分類。

### 4.2 XYChart

| 設定 | 值 |
| --- | --- |
| `panX`／`panY` | `false`／`false` |
| `wheelX`／`wheelY` | `"none"`／`"none"` |
| padding | 四邊皆 `0` |
| 游標 | 無 |

### 4.3 軸

| 軸 | 設定 |
| --- | --- |
| X | `ValueAxis`：`min: 0`、`strictMinMax: true`、`extraMin: 0`、`extraMax: 0`；renderer `strokeOpacity: 0` |
| Y | `ValueAxis`：`strictMinMax: false`；renderer `strokeOpacity: 0` |

兩軸皆設 `visible: false`、格線 `visible: false`（實測），畫面上只有曲線；軸上雖仍掛有標題 Label，但隨軸一起隱藏。圖表高度實測 150px。

### 4.4 序列

| 設定 | 值 |
| --- | --- |
| 類別 | `SmoothedXLineSeries`（`tension: 0.5`） |
| `valueYField`／`valueXField` | `"value"`／`"category"` |
| `maskBullets` | `false` |
| `connect` | `true` |
| 線寬 | `2` |
| 填色 | `visible: true`、`fillOpacity: 0.5` |
| 顏色 | `ChartConfig.colors[0]` |
| 數值標籤 | 無 |
