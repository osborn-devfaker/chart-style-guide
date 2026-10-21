# 設計說明：DIMP 量測圖（amCharts 5）

> 用途：提供給 AI 產生或檢查 DIMP「量測圖」（儀表指針）的 amCharts 5 設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**（樣式層、數字格式、JSON 規則）。
> 實測樣本（2026-10-15）：2323。

---

## 1. 適用範圍

`amChartConfig.type = "gauge"`，以 `am5radar.RadarChart` 繪製半圓儀表，一根指針指向單一彙總值。

## 2. 輸入參數

```json
{
  "valueField": "<數值欄位>",
  "calMethod": "first",
  "startAngle": 180,
  "endAngle": 360
}
```

| 參數 | 說明 |
| --- | --- |
| `valueField` | 只用一個數值欄位（`valueFields[0]`） |
| `calMethod` | 彙總方式：`first`／`last`／`sum`／`avg`／`min`／`max`／`count` |
| `startAngle`／`endAngle` | 來自 `ChartConfig.plotOptions.radialBar`，實測樣本為 `180`／`360`（上半圓） |

## 3. 資料整形

把整欄轉成數字（無法轉換者為 `0`）後一次算出 7 種彙總值，輸出**一列**：`{ first, last, sum, avg, min, max, count }`；`avg` 四捨五入到小數 2 位。指針值取 `calMethod` 指定的那一項。

## 4. 建構

### 4.1 RadarChart

| 設定 | 值 |
| --- | --- |
| `panX`／`panY` | `false`／`false` |
| `startAngle`／`endAngle` | 依輸入參數 |
| `radius`／`innerRadius` | `75%`／`0` |
| padding | 左右 `20`／上下 `16` |
| `bulletsContainer.mask` | 移除（讓指針可畫出圓外） |

### 4.2 數值軸（圓弧刻度）

| 設定 | 值 |
| --- | --- |
| 類別 | `ValueAxis` ＋ `am5radar.AxisRendererCircular`（`innerRadius: -40`） |
| `min`／`strictMinMax`／`maxDeviation` | `0`／`true`／`0` |
| `max` | 取「整數漂亮值」：令 v = max(指針值, 100)，m = 10^floor(log10(v))，n = v / m；n ≤ 1 → `m`、≤ 2 → `2m`、≤ 5 → `5m`，否則 `10m` |
| 格線 | `visible: true`、`strokeOpacity: 0.8`；`stroke` 程式先設為背景色，再被樣式層改成 `grid.borderColor`（實測 `#90a4ae`、`strokeWidth: 2`） |
| `numberFormat` | 依共用說明第 4 節 |

### 4.3 色帶

把 `0`～`max` 均分成 5 段，各建一個 axis range，`axisFill`：`visible: true`、`fillOpacity: 0.8`，顏色依序為：

`#B5CCFF`、`#80A8FF`、`#5C8FFF`、`#2167FF`、`#0051FF`

⚠️ 量測圖**不套用** `ChartConfig.colors`。

### 4.4 指針與中央數值

| 物件 | 設定 |
| --- | --- |
| 指針 | 以 axis data item 掛 `AxisBullet`，sprite 為 `am5radar.ClockHand`：`pinRadius: 15%`、`radius: 100%`、`bottomWidth: 25`；該 data item 的格線隱藏 |
| 指針動畫 | data item 的 `value` 從 0 動畫到指針值：`duration: 800`、`easing: out(cubic)` |
| 指針顏色 | 指針旋轉時，取指針值所落色帶的顏色，以 `500ms` 動畫套到 `hand` 與 `pin` |
| 中央數值 | `radarContainer` 內的 `Label`：`fill: #FFFFFF`、`centerX`／`centerY: 50%`、`textAlign: "center"`、`fontSize: "1.2em"`，文字隨指針值更新 |

## 5. JSON 化限制

指針動畫、依色帶變色、中央數值更新都靠事件，**無法寫進 JSON**（共用說明第 10 節）。JSON 只能保存圖表、軸、色帶的靜態設定，其餘須在 `parse()` 後以程式碼補上。
