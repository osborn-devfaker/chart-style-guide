# DIMP 圖表設計說明（amCharts 5）

依 DIMP 現行做法整理的 amCharts 5 圖表設定說明，供 AI 產生或檢查各類圖表設定時參考。
所有說明**只描述設定與樣式，不含資料列**。

## 閱讀順序

1. [`index.md`](index.md)：總覽、驗證方式、實測樣本、已知未涵蓋項目與裁決紀錄。
2. [`00-common-design-spec.md`](00-common-design-spec.md)：所有圖表共用的建構流程與樣式規則（必讀）。
3. 目標圖表家族的說明（見下表）。
4. 只需要視覺風格時，讀 [`style-guide.md`](style-guide.md) 即可。

## 文件清單

| 檔案 | 內容 |
| --- | --- |
| [`style-guide.md`](style-guide.md) | 樣式與風格：色彩、文字層級、線條、數值標籤、圖例、提示框、間距、數字呈現、動畫 |
| [`00-common-design-spec.md`](00-common-design-spec.md) | 共用：存檔設定、建構流程、`ChartConfig` 對應、數字格式、配色、JSON 化規則、禁止事項 |
| [`bar-chart-design-spec.md`](bar-chart-design-spec.md) | 長條圖（直／橫 × 單一／多序列／堆疊／100% 堆疊） |
| [`line-area-design-spec.md`](line-area-design-spec.md) | 折線圖、區域圖、堆疊區域圖 |
| [`combo-design-spec.md`](combo-design-spec.md) | 組合圖（折線＋長條／堆疊長條／區域／長條＋區域） |
| [`pie-donut-design-spec.md`](pie-donut-design-spec.md) | 圓餅圖、環形圖 |
| [`gauge-design-spec.md`](gauge-design-spec.md) | 量測圖 |
| [`kpi-sparkline-design-spec.md`](kpi-sparkline-design-spec.md) | KPI 指標圖（HTML 指標文字＋迷你趨勢圖） |
| [`bubble-heatmap-treemap-design-spec.md`](bubble-heatmap-treemap-design-spec.md) | 氣泡圖、熱力圖、樹狀圖 |
| [`map-design-spec.md`](map-design-spec.md) | 地圖（色塊／＋氣泡／＋環形／經緯度點位） |
| [`chart-1534-template.md`](chart-1534-template.md)／[`.json`](chart-1534-template.json) | 單張圖表（1534 群組直條圖）完整範本，含 JavaScript 與 `JsonParser` 格式 |

## 範圍

- 不含文字方塊（`textarea`）與表格（`grid`）：兩者由 HTML 直接繪製，不經過 amCharts。
- DIMP 現行做法與 amCharts 官方建議不同時，一律以 DIMP 現行為準。
- 說明中標示「實測」的數值來自本機站台實際渲染；與原始碼推論衝突時以實測為準。
