# 設計說明：DIMP 折線／區域圖（amCharts 5）

> 用途：提供給 AI 產生或檢查 DIMP「折線圖」「區域圖」「堆疊區域圖」的 amCharts 5 設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**（樣式層、數字格式、配色、線型數值標籤、圖例、XY 共用預設、JSON 規則）。
> 實測樣本（2026-10-15）：2258（折線＋右側軸、日期型分類）、2314（區域）、2315（堆疊區域）。

---

## 1. 適用範圍

| DIMP 圖表 | `amChartConfig.type` | `stackType` | amCharts 序列 | 右側數值軸 |
| --- | --- | --- | --- | --- |
| 折線圖 | `line` | `multiple` | `LineSeries` | 可用 |
| 區域圖 | `area` | `multiple` | `SmoothedXLineSeries` | 可用 |
| 堆疊區域圖 | `area` | `stacked` | `SmoothedXLineSeries` | 不可用 |

## 2. 輸入參數

```json
{
  "chartKind": "line",
  "stackType": "multiple",
  "categoryField": "<分類欄位>",
  "categoryType": "date",
  "valueFields": ["<數值欄位 1>", "<數值欄位 2>"],
  "oppositeFields": ["<數值欄位 2>"],
  "axisTitles": { "x": "<X 軸標題>", "y": "<Y 軸標題>", "yRight": "<右軸標題>" }
}
```

| 參數 | 說明 |
| --- | --- |
| `valueFields` | 每個欄位一條線，順序即配色順序 |
| `oppositeFields` | 掛右側數值軸的欄位（`ySeries[i].opposite = true`）；堆疊區域圖一律為空 |

## 3. 資料整形

每列的數值欄位轉數字。堆疊區域圖與長條相同：依分類加總，每個分類一列。

## 4. 建構

### 4.1 軸

| 項目 | 值 |
| --- | --- |
| 分類軸 | 共用說明第 9 節；折線圖在**非日期型**分類時格線 `location: 1`（畫在分類格右緣） |
| 數值軸 | `min`：有負值時不設，否則 `0`；`extraMax: 0.2`；堆疊區域 `calculateTotals: true`，renderer 用 amCharts 預設 |
| 右側數值軸 | `opposite: true`、`syncWithAxis` = 主數值軸；主軸刻度只依其餘欄位計算 |
| 游標 | `behavior: "none"`、綁分類軸；只隱藏 `lineY`（保留垂直參考線） |

### 4.2 序列

| 設定 | 折線圖 | 區域圖／堆疊區域圖 |
| --- | --- | --- |
| 類別 | `LineSeries` | `SmoothedXLineSeries`（`tension: 0.5`） |
| `valueYField`／`categoryXField` | 欄位名稱／`categoryField` | 同左 |
| `connect` | `true`（跨過缺值） | `false`（缺值斷開） |
| `maskBullets` | `true` | `true` |
| `stacked`／`valueYShow` | — | 堆疊區域：`true`／`"valueY"` |
| 線寬 `strokes.template.strokeWidth` | `2` | `2` |
| 填色 `fills.template` | 不顯示 | `visible: true`、`fillOpacity: 0.2` |
| `tooltip.labelText` | `"{valueY}"` | `"{valueY}"` |

資料點圓點與數值標籤見共用說明第 6.2 節。

## 5. 與組合圖的差異

組合圖裡的折線與區域序列設定不同（線寬 `3`、區域透明度 `0.4`、類別 `SmoothedXYLineSeries`），見 `combo-design-spec.md`，不要混用。
