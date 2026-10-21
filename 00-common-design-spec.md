# 設計說明（共用）：DIMP 圖表的 amCharts 5 建構與樣式規則

> 用途：所有 DIMP 圖表家族共用的規則。各家族說明只寫該家族特有的部分，**共用規則一律以本檔為準**，產生任何家族的圖表前都要先讀本檔。
> 本文件不含任何資料列。
> 依據：DIMP 原始碼 `wwwroot/DIMP2/js/ChartTool/src/`（`styles/`、`managers/`、`data-processors/`）＋ 2026-10-15 在本機站台實際渲染 22 張圖表、讀取 amCharts 執行期物件的實測結果（見 `index.md`）＋ amCharts MCP server 文件。
> 執行環境：amCharts **5.20.0**。

---

## 1. 兩份存檔設定

每張圖表在資料庫（`DIMP2_ChartsSetting`）存兩份 JSON，AI 產生設計時兩份都要考慮：

| 欄位 | 角色 | 主要內容 |
| --- | --- | --- |
| `DataLogicConfig` | **結構**：決定建出哪些 amCharts 物件 | `amChartConfig.type`（家族）、`amChartConfig.stackType`（模式）、`amChartConfig.categoryField`、`amChartConfig.valueFields`、`xSeries`／`ySeries`（每個欄位的序列型別、是否掛右軸）、`calMethod`（量測圖）、`trending`（KPI）、`mapSubType`（地圖） |
| `ChartConfig` | **樣式**：建好後由樣式層套用 | 沿用 ApexCharts 形狀的物件：`colors`、`xaxis`、`yaxis`、`yaxisRight`、`legend`、`dataLabels`、`plotOptions`、`grid`、`tooltip`、`title`、`subtitle`、`chart` |

`ChartConfig` 雖然是 ApexCharts 的形狀，但**大部分欄位會被轉成 amCharts 設定**（見第 3 節）。只有沒列在第 3 節的欄位（例如 `fill.type = "gradient"`、`stroke`、`plotOptions.bar.dataLabels.total`）在 amCharts 執行期沒有作用。

## 2. 建構流程（所有家族相同）

1. 建 `Root`。
2. 建圖表物件（依家族：`XYChart`／`PieChart`／`RadarChart`／`MapChart`／`Treemap`）。
3. **資料整形**：依家族把原始資料列轉成序列需要的形狀（各家族說明的「資料整形」一節）。
4. **軸管理器**建軸與游標。
5. **序列管理器**建序列，並設定序列層級的預設值。
6. **樣式層**依 `ChartConfig` 套用樣式（第 3 節）。⚠️ 樣式層會**先清掉序列既有的 bullets 再重建**數值標籤，所以序列管理器裡寫的標籤設定**不是最終結果**，以第 6 節為準。
7. 建圖例，資料為 `chart.series.values`。

## 3. `ChartConfig` → amCharts 對應（樣式層）

| `ChartConfig` 路徑 | 套用對象 | 轉成的 amCharts 設定 |
| --- | --- | --- |
| `colors` | 圖表 `colors`、各序列 | 見第 5 節 |
| `xaxis.labels.style` | `chart.xAxes[0]` 的刻度標籤 | `fontSize`、`fontWeight`、`fill = colors[0]`（預設 `#333333`）、`show` → 可見性；`scales`／`thousands`／`unit` → 該軸 `numberFormat`（第 4 節） |
| `yaxis.labels.style` | `chart.yAxes[0]` 的刻度標籤 | 同上 |
| `yaxisRight.labels.style` | `renderer.opposite = true` 的 Y 軸 | 同上 |
| `xaxis.title.style`／`yaxis.title.style`／`yaxisRight.title.style` | 對應軸的標題 Label | `text`（空字串時保留原標題）、`fontSize`、`fontWeight`、`fill = color`（預設 `#333333`）、`visible` |
| `legend` | 圖例 | `position`（`top`／`bottom`，預設 `bottom`）、`horizontalAlign`（預設 `center`）、`fontSize`、`fontWeight`、`labels.colors[0]` → 文字色、`show` → 可見性 |
| `dataLabels` | 各序列的數值標籤 | `enabled`、`enabledOnSeries[i]`（逐序列開關）、`style.fontSize`、`style.fontWeight`、`style.colors[i]`（逐序列文字色，預設 `#333333`）、`style.scales`／`thousands`／`unit` |
| `plotOptions.bar` | 長條的標籤位置 | `dataLabels.position`（`top`／`center`／`bottom`，預設 `center`）、`horizontal` |
| `plotOptions.pie` | 圓餅標籤 | `dataLabels.offset`：`0` → 標籤畫在扇形內（`radial`），其他值 → 畫在外側並對齊 |
| `plotOptions.radialBar` | 量測圖角度 | `startAngle`、`endAngle` |
| `grid` | 格線 | `borderColor` → 格線 `stroke`、`strokeDashArray`、`xaxis.lines.show`／`yaxis.lines.show` |
| `grid.row`／`grid.column` | 交錯底色 | `show`、`colors`、`opacity` |
| `markers` | 參考線／區間 | `show`、`ranges` |
| `xaxis.strictMinMax`／`yaxis.strictMinMax` | 軸範圍 | `visible`、`min`、`max` |
| `tooltip` | 各序列提示框 | `enabled`、`style.fontSize`；`fontWeight` 固定 `600` |
| `title`／`subtitle` | 圖表外的主／副標題 | `text`、`style.fontSize`／`fontWeight`／`color`、`visible` |
| `chart` | 外框 | `width`、`height`、`background`（`transparent` 視為白色）、`enabledBackground` |

⚠️ `xaxis`／`yaxis` 指的是**畫面上的實體軸**，不是分類軸／數值軸。橫條圖的 `xaxis` 對應的是數值軸。

## 4. 數字格式

軸刻度與數值標籤共用同一套規則，由 `scales`（小數位數）、`thousands`（千分位）、`unit`（單位）組成：

| 輸入 | 格式字串 |
| --- | --- |
| 基底 | `thousands = true` → `"#,###"`；否則 `"#"` |
| `scales > 0` | 基底後補 `"."` 加 `scales` 個 `0`（例：`scales = 2` → `"#.00"`） |
| `scales = 0` | 基底後補 `"."`（例：`"#."`，取整數） |
| `scales` 為 `-1` 或未設定 | 不補（原值照印） |
| `unit` | 接在最後（例：`"#,###.0萬"`） |

數值標籤的文字為 `{<欄位>.formatNumber("<格式字串>")}`。100% 堆疊模式固定為 `{value?TotalPercent.formatNumber('#.#')}%`，不套上表。

## 5. 配色

| 家族 | 規則 |
| --- | --- |
| XY 類（長條、折線、區域、組合、氣泡、KPI 迷你圖） | 第 i 個序列的 `fill`／`stroke`（長條另含 `columns.template` 的 `fill`／`stroke`）= `ChartConfig.colors[i]`，超出長度時循環 |
| 圓餅／環形 | 第 i 個扇形 = `colors[i]` |
| 樹狀圖 | 第 i 個第一層節點底下的所有子節點 = `colors[i % colors.length]` |
| 量測圖 | **不套用** `colors`；色帶為固定 5 色（見量測圖說明） |
| 熱力圖 | 填色為固定色階（見熱力圖說明）；格子框線仍取 `colors[0]` |
| 地圖 | 熱力色階起訖色優先讀地圖樣式設定，再退回 `colors` 頭尾（見地圖說明） |

## 6. 數值標籤（樣式層重建後的實際結果）

### 6.1 長條（`ColumnSeries`）

| 項目 | 值 |
| --- | --- |
| Bullet 位置 | 依 `plotOptions.bar.dataLabels.position`：`top` → `1`、`center` → `0.5`、`bottom` → `0`；直條設 `locationY`，橫條設 `locationX`（另一方向為 `0.5`） |
| Label | `populateText: true`、`fontSize`／`fontWeight`／`fill` 依 `dataLabels.style`、`padding` 上下 `8`／左右 `10`、`centerX`／`centerY` 依位置（置中時皆 `50%`） |
| 顯示條件 | 直條：柱高 ≥ 5px 且值 > 0；橫條：柱長 ≥ 20px 且值 > 0。不符合時只隱藏**標籤** |
| 柱子 | 一律繪製。長條序列管理器在柱子上掛了 `visible` adapter，但實測對畫面沒有作用（柱高 6px 的柱子照常顯示），**不要**據此隱藏柱子 |
| 提示框 | `columns.template.tooltipText = "{name}:<數值標籤文字>"` |

### 6.2 線型（`LineSeries`／`SmoothedXLineSeries`／`SmoothedXYLineSeries`）

每個資料點兩個 bullet：

| Bullet | 設定 |
| --- | --- |
| 圓點 | `Circle`：`radius: 5`、`strokeWidth: 3`、`stroke` = 序列色、`fill: #FFFFFF`；`locationY: 1` |
| 數值 | `Label`：`centerX: 50%`、`centerY: 20 + fontSize`（畫在點的上方）、`padding` 上下 `4`／左右 `6`；背景 `RoundedRectangle`：四角 `cornerRadius 3`、`fill`／`stroke` = 序列色、`fillOpacity: 0.1`；`locationX: 0.5`、`locationY: 1` |

`dataLabels.enabled = false` 或該序列在 `enabledOnSeries` 為 `false` 時只留圓點。

### 6.3 圓餅（`PieSeries`）

| `plotOptions.pie.dataLabels.offset` | 標籤 |
| --- | --- |
| `0` | `textType: "radial"`、`radius: 0`、`centerX: 100%`，畫在扇形內，不顯示引線 |
| 其他（預設） | `textType: "aligned"`、`radius: 10`、`alignLabels: true`，畫在圓外並依剩餘寬度自動換行（最小 40px） |

文字預設為 `{valuePercentTotal.formatNumber('#.0')}%`；佔比 < 3% 的扇形隱藏標籤與引線。

## 7. 圖例

| 項目 | 值 |
| --- | --- |
| 加入方式 | 圖表的 `children`；資料為 `chart.series.values`（圓餅為扇形） |
| 位置 | `centerX: 50%`；`legend.position = "bottom"` → `centerY: 100%`，`"top"` → `centerY: 0%` |
| 寬度上限 | `maxWidth: 600` |
| 文字 | `fontSize`、`fontWeight`、`fill` 依 `ChartConfig.legend` |
| 標記 | 圓餅／環形 `useDefaultMarker: true`；其餘 `false` |

## 8. 軸標題

軸標題是加在軸上的 `Label`（`isTtitle: true`）：

| 位置 | 設定 |
| --- | --- |
| 水平軸 | `centerX: 50%` |
| 垂直軸 | `rotation: -90`、`centerY: 50%`，放在刻度外側 |
| 共用 | `padding` 上下 `8`／左右 `10`、`fontSize`／`fontWeight`／`fill` 依 `*.title.style`、`populateText: false` |

## 9. XY 類共用預設

（長條、折線、區域、組合、氣泡、KPI 迷你圖；熱力圖例外見其說明）

| 項目 | 值 |
| --- | --- |
| `XYChart` | `panX: false`、`panY: false`；垂直分類軸：`wheelX: "panX"`、`wheelY: "zoomX"`，`padding` 左右 `20`／上下 `16`；水平分類軸（橫條）：`wheelX: "panY"`、`wheelY: "zoomY"`，`paddingLeft: 0`、`paddingTop: 30`、`paddingRight: 20`、`paddingBottom: 20` |
| `XYCursor` | `behavior: "none"`，綁定分類軸；`lineX`、`lineY` 隱藏（折線只隱藏 `lineY`） |
| 分類軸 | `CategoryAxis`、`tooltip`、`markUnitChange: false`；renderer `cellStartLocation: 0.1`／`cellEndLocation: 0.9`、`minGridDistance`：分類數 n ≤ 25 → `15 + round(n / 25) × 10`，否則 `50` |
| 分類軸標籤 | `maxWidth: 80`、`textAlign: "center"`、`oversizedBehavior`：n ≤ 6 → `"truncate"`，否則 `"fit"`；水平軸上任一名稱超過 12 字或標籤寬度 > 80px → `rotation: -60`、`centerX: 100%`、`centerY: 50%` |
| 日期型分類 | `dateFormats`／`periodChangeDateFormats` = `{ day: "yyyy/MM/dd", month: "yyyy/MM/dd", year: "yyyy" }` |
| 數值軸 | `ValueAxis`；有負值時不設 `min`，否則 `min: 0`；`extraMax: 0.2`（100% 堆疊為 `0`）；`maxDeviation: 0.1`；`numberFormat` 依第 4 節 |
| 數值軸 renderer（非堆疊） | `strokeOpacity: 0.1`、`minGridDistance = max(20 + 10^floor(log10(d)), 50)`，d = 最大值的整數位數；有負值時 `30` |
| 數值軸 renderer（堆疊） | 只設 `minorGridEnabled: false`，其餘用 amCharts 預設（`AxisRendererX` 120、`AxisRendererY` 40） |
| 右側數值軸 | renderer `opposite: true`、`syncWithAxis` = 主數值軸、`extraMax: 0.2`；掛在 `ySeries[i].opposite = true` 的序列 |
| 格線 | `stroke` = `grid.borderColor`、`strokeWidth: 2`、`strokeDasharray` = `grid.strokeDashArray` |

## 10. 序列化成 JSON（`am5plugins_json.JsonParser`）

1. `refs` 用陣列分群組：資料 → 軸 → 序列；同一群組內的物件不能互相參照（5.20.0 會擲出 `Could not find ref`）。
2. 以 `#` 或 `@` 開頭的字串值要重複前綴：`numberFormat: "#."` 寫成 `"##."`。
3. 子元素（軸標題、圖例）寫在物件頂層的 `children`；需要插到最前面時加 `"index": 0`。
4. 顏色寫 `{ "type": "Color", "value": "#RRGGBB" }`，百分比寫 `{ "type": "Percent", "value": 50 }`。
5. **adapter、事件、`heatRules` 以外的動態行為無法寫進 JSON**：第 6.1 節的標籤顯示條件、第 3 節的刻度樣式 adapter、地圖的 tooltip HTML、量測圖指針動畫等，都要在 `parse()` 之後以程式碼補上。
6. 資料不寫進設計檔；執行期以 `series.data.setAll()` 填入。

## 11. 禁止事項

- 不使用 5.20.0 之後才有的功能：`colorByDataItem`（5.20.4）、`@root.interfaceColors` 參照（5.21.0）、`<am5-chart>`（5.21.0）、`autoAppear`（5.21.0）。
- 不用 `new` 建立物件，一律 `.new(root, {...})`。
- 重繪前呼叫既有 `root.dispose()`。
- DIMP 現行做法與 amCharts 官方建議不同時，**一律照 DIMP 現行**（2026-10-15 使用者裁決）。
