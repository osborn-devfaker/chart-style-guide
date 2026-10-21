# DIMP 圖表設計說明（amCharts 5）總覽

> 產出日期：2026-10-15（分支 `develop-workitem-20145`）
> 用途：讓 AI 依 DIMP 現行做法產生或檢查各類圖表的 amCharts 5 設定。所有說明**不含資料列**。
> 使用方式：**先讀 `00-common-design-spec.md`，再讀目標家族的說明**。

## 文件清單

| 檔案 | 涵蓋圖表 |
| --- | --- |
| `style-guide.md` | **只含樣式與風格**：色盤、中性色、固定語意色、文字層級、線條形狀、數值標籤、圖例、提示框、間距、動畫（不含資料與建構邏輯） |
| `00-common-design-spec.md` | 共用：兩份存檔設定、建構流程、`ChartConfig` 對應、數字格式、配色、數值標籤、圖例、軸標題、XY 共用預設、JSON 化規則、禁止事項 |
| `bar-chart-design-spec.md` | 長條圖 8 種（直／橫 × 單一／多序列／堆疊／100% 堆疊） |
| `line-area-design-spec.md` | 折線圖、區域圖、堆疊區域圖 |
| `combo-design-spec.md` | 組合圖 4 種（折線＋長條／堆疊長條／區域／長條＋區域） |
| `pie-donut-design-spec.md` | 圓餅圖、環形圖 |
| `gauge-design-spec.md` | 量測圖 |
| `kpi-sparkline-design-spec.md` | KPI 指標圖（HTML 指標文字＋迷你趨勢圖） |
| `bubble-heatmap-treemap-design-spec.md` | 氣泡圖、熱力圖、樹狀圖 |
| `map-design-spec.md` | 地圖 4 種（色塊／＋氣泡／＋環形／經緯度點位） |
| `chart-1534-template.md`／`.json` | 單張圖表（1534）的完整範本，含可執行的 JS 與 JsonParser 格式 |

不在範圍內：文字方塊（`textarea`）與表格（`grid`）——兩者不經過 amCharts，由 HTML 直接繪製。

## 驗證方式

1. **讀原始碼**：`wwwroot/DIMP2/js/ChartTool/src/` 的軸／序列管理器、資料整形、樣式層，以及 `dashboard/src/core/legacy-chart-renderer.js`（KPI 文字）。
2. **實際渲染實測**：在本機站台 `https://localhost:20145/DIMP2_PublicCharts/ViewDashboardChart/0/{SN}` 逐張載入，以 `tools/am5-probe.js` 讀出 amCharts 執行期物件樹，`tools/digest.py` 整理成摘要，並截圖存於 `screens/`。原始結果：`tools/probe-round1～3.json`。
3. **官方文件**：amCharts MCP server（`https://mcp.amcharts.com/mcp`，涵蓋 5.21.0）；設定名稱另對照本機 amCharts 5.20.1 原始碼。

說明中寫「實測」的數值來自第 2 步；兩者衝突時以實測為準，並在說明內註明。

## 實測樣本

| 家族 | SN |
| --- | --- |
| 長條（直） | 1534、2311（多序列）、2312（堆疊）、2313（100% 堆疊） |
| 長條（橫） | 2558（多序列）、2309（堆疊）、2310（100% 堆疊） |
| 折線／區域 | 2258（折線＋右軸）、2314（區域）、2315（堆疊區域） |
| 組合 | 2257、2250、2316、2319 |
| 圓餅／環形 | 2320、2397 |
| 量測圖 | 2323 |
| KPI | 2522 |
| 氣泡／熱力／樹狀 | 2307、2322、2398 |
| 地圖 | 2534（色塊）、2531（＋氣泡）、2525（＋環形）、2529（點位） |

> 本機只有部分圖表的資料來源有靜態化檔案；較新的 SN（如 2494、2508、2513）會回 `GetChartDatas` 404，屬本機環境條件，不是程式問題。

## 已知未涵蓋

- 圓餅的「外部對齊標籤」模式：兩個樣本都是標籤在扇形內（`offset = 0`），外部模式只依原始碼撰寫。
- 量測圖只有 `180`～`360` 度一種角度樣本。
- 100% 堆疊區域圖：圖表類型清單沒有這一種，未撰寫。
- 地圖的視角設定（`viewportEnabled`）、聚焦單一行政區、圖例合併等互動邏輯未展開。
- KPI 指標文字的計算只讀原始碼，未以不同資料逐一驗算。

## 裁決紀錄

- 2026-10-15：DIMP 現行做法與 amCharts 官方建議不同時，一律照 DIMP 現行。
- 2026-10-15 修正：長條「直條柱高 < 20px 隱藏柱子」為依程式碼推論的錯誤敘述；實測柱子一律繪製，只有**標籤**依門檻隱藏（直條 < 5px、橫條 < 20px，或值 ≤ 0）。`bar-chart-design-spec.md` 與 `chart-1534-template.md` 已更正。
