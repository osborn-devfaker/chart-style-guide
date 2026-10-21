# 設計說明：DIMP 組合圖（amCharts 5）

> 用途：提供給 AI 產生或檢查 DIMP「組合圖」（折線＋長條＋區域混搭）的 amCharts 5 設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**；長條部分的數值軸與標籤規則另見 `bar-chart-design-spec.md`。
> 實測樣本（2026-10-15）：2257（折線掛右軸＋長條）、2250（折線＋區域掛右軸＋折線）、2316（堆疊：折線＋長條）、2319。

---

## 1. 適用範圍

| DIMP 圖表 | 序列組成 | `stackType` |
| --- | --- | --- |
| 折線＋多軸長條圖 | `line`、`column` | `multiple` |
| 折線＋堆疊長條 | `line`、`column` | `stacked` |
| 折線＋區域圖 | `line`、`area` | `multiple` |
| 折線＋長條＋區域 | `line`、`column`、`area` | `multiple` |

一律為**直式**（分類軸在 X 軸），`amChartConfig.type = "combo"`。

## 2. 輸入參數

```json
{
  "stackType": "multiple",
  "categoryField": "<分類欄位>",
  "categoryType": "date",
  "series": [
    { "field": "<欄位 1>", "seriesType": "line", "opposite": true },
    { "field": "<欄位 2>", "seriesType": "column", "opposite": false }
  ],
  "axisTitles": { "x": "<X 軸標題>", "y": "<Y 軸標題>", "yRight": "<右軸標題>" }
}
```

`series` 對應存檔的 `ySeries[i]`（`chartType` → `seriesType`、`opposite`），**順序即建立順序與配色順序**。

## 3. 建構

### 3.1 軸

分類軸、數值軸、右側數值軸同共用說明第 9 節。任一序列 `opposite = true` 即建立右側軸（`syncWithAxis` = 主軸）。

### 3.2 序列（依 `seriesType`）

| 設定 | `column` | `line` | `area` |
| --- | --- | --- | --- |
| 類別 | `ColumnSeries` | `LineSeries` | `SmoothedXYLineSeries`（`tension: 0.5`） |
| `valueYField`／`categoryXField` | 欄位／`categoryField` | 同左 | 同左 |
| `connect` | — | `false` | `false` |
| 堆疊（`stackType = stacked`） | `stacked: true`、`valueYShow: "valueY"` | **不堆疊** | `stacked: true`、`valueYShow: "valueY"` |
| 線寬 | — | `3` | `3` |
| 填色 | 序列色 | 不顯示 | `visible: true`、`fillOpacity: 0.4` |
| `tooltip.labelText` | `"{name}: {valueY}"` | `"{name}: {valueY}"` | `"{name}: {valueY}"` |
| 數值標籤 | 共用說明第 6.1 節 | 共用說明第 6.2 節 | 共用說明第 6.2 節 |

> 與單獨的折線圖、區域圖相比：組合圖的折線線寬 `3`（折線圖為 `2`）、`connect: false`（折線圖為 `true`）；區域用 `SmoothedXYLineSeries`、透明度 `0.4`（區域圖為 `SmoothedXLineSeries`、`0.2`）。

## 4. 注意事項

- 存檔的 `ySeries` 有幾個欄位，不保證執行期就有幾個序列：2319 存了 `line,column,area` 三個，實測只畫出 1 條折線。產生設計時以 `ySeries` 為準，驗收時要實際渲染核對序列數。
- 長條與折線共用同一個主數值軸時，堆疊只影響長條與區域，折線的值不會被加總。
