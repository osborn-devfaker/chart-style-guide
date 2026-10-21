# 設計說明：DIMP 圓餅／環形圖（amCharts 5）

> 用途：提供給 AI 產生或檢查 DIMP「圓餅圖」「環形圖」的 amCharts 5 設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**（樣式層、配色、圓餅標籤、圖例、JSON 規則）。
> 實測樣本（2026-10-15）：2320（圓餅）、2397（環形）。

---

## 1. 適用範圍

| DIMP 圖表 | `amChartConfig.type` | 圖表物件 | 差異 |
| --- | --- | --- | --- |
| 圓餅圖 | `pie` | `am5percent.PieChart` | 無內徑 |
| 環形圖 | `donut` | `am5percent.PieChart` | `innerRadius: 50%` |

## 2. 輸入參數

```json
{
  "chartKind": "pie",
  "categoryField": "<分類欄位>",
  "valueField": "<數值欄位>"
}
```

只用**一個**數值欄位（`valueFields[0]`）。

## 3. 資料整形

每列一個扇形：分類 = `categoryField`、值 = `valueFields[0]` 轉數字。

## 4. 建構

### 4.1 PieChart

| 設定 | 圓餅 | 環形 |
| --- | --- | --- |
| `radius` | `65%` | `65%` |
| `innerRadius` | 不設 | `50%` |
| `startAngle`／`endAngle` | `-90`／`270` | `-90`／`270` |

### 4.2 PieSeries

| 設定 | 值 |
| --- | --- |
| `valueField`／`categoryField` | 數值欄位／分類欄位 |
| `startAngle`／`endAngle` | `-90`／`270`（從 12 點鐘方向順時針） |
| `alignLabels` | 依 `plotOptions.pie.dataLabels.offset`：`0` → `false`，其他 → `true` |
| `legendLabelText` | 圓餅 `"{category}"`；環形不設 |
| `colors` | 由 `ChartConfig.colors` 建立的 `ColorSet`，第 i 個扇形取 `colors[i]` |
| 引線 `ticks.template.stroke` | 與標籤文字同色（`dataLabels.style.colors[0]`，預設 `#333333`） |

扇形標籤（文字、位置、換行、< 3% 隱藏）見共用說明第 6.3 節。

### 4.3 圖例

扇形逐一列入圖例，`useDefaultMarker: true`，其餘同共用說明第 7 節。

## 5. 標籤文字變數

圓餅標籤只能使用 `{category}`、`{value}`、`{valuePercentTotal}` 三個變數。若設定的文字不含任何一個，DIMP 會改用預設 `{valuePercentTotal.formatNumber('#.0')}%`；單獨寫 `{valuePercentTotal}` 時會自動補成 `{valuePercentTotal.formatNumber('#.0')}%`。
