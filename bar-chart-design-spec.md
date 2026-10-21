# 設計說明：DIMP bar 類圖表（amCharts 5）

> 用途：提供給 AI 產生或檢查 DIMP「長條圖」家族的 amCharts 5 圖表設定。本文件**不含任何資料列**。
> **先讀 `00-common-design-spec.md`**：樣式層對應、數字格式、配色、數值標籤、圖例、軸標題、XY 共用預設、JSON 規則與禁止事項都在該檔，本檔不重述。
> 實測樣本（2026-10-15，本機站台）：1534（直條＋多序列）、2311（直條＋多序列）、2312（直條＋堆疊）、2313（直條＋100% 堆疊）、2558（橫條＋多序列）、2309（橫條＋堆疊）、2310（橫條＋100% 堆疊）。

---

## 1. 適用範圍

方向 × 模式共 8 種：

| 方向 | DIMP `amChartConfig.type` | 分類軸 | 數值軸 |
| --- | --- | --- | --- |
| 直條（vertical） | `column` | X 軸（`AxisRendererX`） | Y 軸（`AxisRendererY`） |
| 橫條（horizontal） | `bar` | Y 軸（`AxisRendererY`） | X 軸（`AxisRendererX`） |

| 模式 | `amChartConfig.stackType` | 判定來源（`ChartConfig.chart`） |
| --- | --- | --- |
| 單一／多序列並排 | `multiple` | `stacked = false`（數值欄位 1 個即單一） |
| 堆疊 | `stacked` | `stacked = true` 且 `stackType = "normal"` |
| 100% 堆疊 | `full-stacked` | `stacked = true` 且 `stackType = "100%"` |

## 2. 輸入參數

```json
{
  "orientation": "vertical",
  "stackType": "multiple",
  "categoryField": "<分類欄位>",
  "categoryType": "string",
  "valueFields": ["<數值欄位 1>", "<數值欄位 2>"],
  "oppositeFields": [],
  "axisTitles": { "x": "<X 軸標題>", "y": "<Y 軸標題>", "yRight": "<右軸標題>" }
}
```

樣式參數（色盤、字級、數字格式、標籤位置、圖例位置）一律來自 `ChartConfig`，對應見共用說明第 3 節。

| 參數 | 說明 |
| --- | --- |
| `categoryType` | `string` 或 `date`；`date` 時分類軸套日期格式（共用說明第 9 節） |
| `valueFields` | 每個欄位產生 1 個 `ColumnSeries`，順序即配色順序 |
| `oppositeFields` | 掛右側數值軸的欄位；**僅直條的 `multiple` 有效**，其餘一律視為空陣列 |

## 3. 資料整形

| 模式 | 規則 |
| --- | --- |
| `multiple` | 每列的數值欄位轉數字，無法轉換者為 `0` |
| `stacked`／`full-stacked` | 依 `categoryField` 分組，數值欄位加總，每個分類只留一列 |

## 4. 建構

共用流程與 XY 預設見共用說明第 2、9 節。本家族特有的設定：

### 4.1 數值軸

| 設定 | `multiple` | `stacked` | `full-stacked` |
| --- | --- | --- | --- |
| `min` | 有負值時不設，否則 `0` | 同左 | `0` |
| `max`／`strictMinMax` | 不設／`false` | 不設／`false` | `100`／`true` |
| `extraMax` | `0.2` | `0.2` | `0` |
| `calculateTotals` | `false` | 直條 `true`；橫條 `false` | `true` |
| renderer | 共用說明第 9 節「非堆疊」 | 共用說明第 9 節「堆疊」 | 共用說明第 9 節「堆疊」 |

右側數值軸：僅直條 `multiple` 且 `oppositeFields` 非空時建立（共用說明第 9 節）。

### 4.2 序列

每個數值欄位一個 `ColumnSeries`（`V` 代表數值方向：直條 `Y`、橫條 `X`；`C` 代表分類方向）：

| 設定 | 值 |
| --- | --- |
| `name`／`value{V}Field` | 欄位名稱 |
| `category{C}Field` | `categoryField` |
| `sequencedInterpolation` | `true` |
| `clustered` | `true`（預設值） |
| `stacked` | 堆疊模式 `true` |
| `value{V}Show` | `stacked` → `"value{V}"`；`full-stacked` → `"value{V}TotalPercent"` |
| `tooltip.labelText` | `"{name}: {value{V}}"`；`full-stacked` 為 `"{name}: {value{V}TotalPercent.formatNumber('#.#')}%"`；`pointerOrientation: "vertical"` |
| 柱寬（直條 `width`、橫條 `height`） | `100%`（分類格內 0.1～0.9 已留間距） |

數值標籤、柱子顯示與提示框文字見共用說明第 6.1 節。

## 5. 方向差異總表

| 項目 | 直條 | 橫條 |
| --- | --- | --- |
| `wheelX`／`wheelY` | `panX`／`zoomX` | `panY`／`zoomY` |
| 圖表 padding（左／上／右／下） | `20`／`16`／`20`／`16` | `0`／`30`／`20`／`20` |
| 游標綁定 | `xAxis` = 分類軸 | `yAxis` = 分類軸 |
| 分類標籤旋轉 | 名稱超過 12 字或標籤寬 > 80px 時 `-60` | 不旋轉 |
| 標籤隱藏門檻 | 柱高 < 5px 或值 ≤ 0 | 柱長 < 20px 或值 ≤ 0 |
| 右側數值軸 | `multiple` 可用 | 不可用 |
| 分類順序 | 由左至右 | 分類軸未設 `inversed`，第一筆在**最下方** |

## 6. 已裁決事項

下列 DIMP 現行做法與 amCharts 官方建議不同，**已於 2026-10-15 裁決照 DIMP 現行**，產生圖表時不得改採官方建議：

| 項目 | DIMP 現行（照做） | amCharts 官方建議（不採用） |
| --- | --- | --- |
| 橫條分類順序 | 不設 `inversed`，第一筆在最下方 | `AxisRendererY` 設 `inversed: true` |
| 小柱子處理 | 柱子一律繪製，只依門檻隱藏標籤（直條與橫條門檻不同） | 以柱子尺寸判斷標籤顯示，不分方向 |
| 橫條一般堆疊 | 不設 `calculateTotals` | 顯示總計時才需要 |

> 修正紀錄：前一版寫「直條柱高 < 20px 時隱藏柱子」，是依序列管理器的程式碼推論；實測（1534：柱高 6px、17px 的柱子照常顯示）證實該 adapter 對畫面沒有作用，且標籤由樣式層重建、門檻為 5px，已改為上表的實際行為。
