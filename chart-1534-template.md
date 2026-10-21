# amCharts 5 圖表設定範本：群組直條圖（圖表 1534）

> 產出日期：2026-10-15
> 測試來源：`https://localhost:20145/DIMP2_PublicCharts/ViewDashboardChart/0/1534`
> 查證管道：amCharts MCP server（`amcharts5` 1.7.1，`https://mcp.amcharts.com/mcp`；文件涵蓋至 amCharts 5.21.0）
> 執行環境：站台實際載入 amCharts **5.20.0**；設定名稱另以本機原始碼 `D:\library-doc\amcharts5\src`（5.20.1）核對
> 同目錄的 `chart-1534-template.json` 是同一張圖的 JSON 版本（`am5plugins_json.JsonParser` 格式）

---

## 1. 圖表摘要

| 項目 | 值 |
| --- | --- |
| DIMP 圖表類型 | `datalogicConfig.amChartConfig.type = "column"`、`stackType = "multiple"`（群組，不堆疊） |
| amCharts 結構 | `XYChart` ＋ `CategoryAxis`（X）＋ `ValueAxis`（Y）＋ 3 個 `ColumnSeries` ＋ `Legend` ＋ `XYCursor` |
| 資料來源 | `sourceSN = 1387`、`sourceType = 3`，共 6 列 |
| 分類欄位 | `轄區分局` |
| 數值欄位 | `SUM(A1)`、`SUM(A2)`、`SUM(A3)` |
| 主題 | `themeC04`（10 色調色盤，見第 3 節） |

資料樣本（實際渲染時取得）：

| 轄區分局 | SUM(A1) | SUM(A2) | SUM(A3) |
| --- | ---: | ---: | ---: |
| 大雅分局 | 0 | 16 | 126 |
| 太平分局 | 0 | 11 | 4 |
| 烏日分局 | 0 | 4 | 5 |
| 第一分局 | 0 | 11 | 8 |
| 第二分局 | 0 | 96 | 272 |
| 第三分局 | 0 | 48 | 81 |

## 2. 設定項目對照

「實際值」取自瀏覽器中執行期的 amCharts 物件，**不是**從 DIMP 的存檔設定推論。

| 物件 | 設定 | 實際值 | 對應的 DIMP 設定 |
| --- | --- | --- | --- |
| XYChart | `panX`／`panY` | `false`／`false` | — |
| XYChart | `wheelX`／`wheelY` | `"panX"`／`"zoomX"` | — |
| XYChart | `layout` | `root.verticalLayout` | — |
| XYChart | `paddingTop／Right／Bottom／Left` | `16／20／16／20` | — |
| XYChart | `colors` | `ColorSet`（10 色） | `chartConfig.colors` |
| XYCursor | `behavior`、`xAxis` | `"none"`、X 軸 | — |
| CategoryAxis | `categoryField` | `"轄區分局"` | `amChartConfig.categoryField` |
| CategoryAxis | `maxDeviation` | `0.1` | — |
| AxisRendererX | `minGridDistance` | `15` | — |
| AxisRendererX | `cellStartLocation`／`cellEndLocation` | `0.1`／`0.9` | — |
| AxisRendererX labels | `fontSize`、`oversizedBehavior`、`maxWidth` | `"12px"`、`"truncate"`、`80` | `chartConfig.xaxis.labels.style` |
| X 軸標題（Label） | `text`、`fontWeight` | `"轄區分局"`、`"bold"` | `chartConfig.xaxis.title.style` |
| ValueAxis | `min`、`extraMax`、`maxDeviation` | `0`、`0.2`、`0.1` | — |
| ValueAxis | `numberFormat` | `"#."`（整數） | `datalogicConfig.yLabel.scales = 0` |
| AxisRendererY | `minGridDistance` | `50` | — |
| Y 軸標題（Label） | `text`、`rotation` | `"數值"`、`-90` | `chartConfig.yaxis.title.style` |
| 格線 | `stroke` | `#90A4AE` | `chartConfig.grid.borderColor` |
| ColumnSeries | `clustered`、`stacked` | `true`、`false` | `amChartConfig.stackType = "multiple"` |
| ColumnSeries | `sequencedInterpolation` | `true` | — |
| ColumnSeries | `name`／`valueYField` | 數值欄位名稱 | `amChartConfig.valueFields` |
| columns.template | `width` | `100%` | — |
| Tooltip | `pointerOrientation`、`labelText` | `"vertical"`、`"{name}:{valueY.formatNumber('#.')}"` | `chartConfig.tooltip` |
| 數值標籤（Bullet＋Label） | `locationX/Y`、`fontSize`、`fontWeight`、`fill` | `0.5/0.5`、`"14px"`、`"bold"`、`#333333` | `chartConfig.dataLabels` |
| Legend | `x`／`centerX`、`layout` | `50%`／`50%`、`root.gridLayout` | `chartConfig.legend.position = "bottom"` |
| Legend labels | `fontSize`、`fill` | `"14px"`、`#333333` | `chartConfig.legend` |

> `chartConfig` 是沿用 ApexCharts 格式的舊設定，其中 `fill.type = "gradient"`、`plotOptions.bar.dataLabels.total`
> 等項目在 amCharts 執行期**沒有生效**（實測 `fillGradient` 未設定），範本不納入。

## 3. 調色盤（themeC04）

`#6DE082`、`#3BC5C5`、`#10847C`、`#88DFCB`、`#48CE98`、`#1FBACC`、`#00AAA9`、`#0A94BE`、`#5FB4CE`、`#B9EEE3`

序列依序取色：`SUM(A1)` → `#6DE082`、`SUM(A2)` → `#3BC5C5`、`SUM(A3)` → `#10847C`。

## 4. JavaScript 範本（script 載入版）

需要的腳本：`index.js`、`xy.js`（DIMP 的位置在 `/DIMP2/js/amChart/`）。

```javascript
// 群組直條圖（clustered column）：以分類欄位為 X 軸、多個數值欄位各成一組柱子
const renderClusteredColumn = (divId, data, opts) => {
  const root = am5.Root.new(divId);

  const chart = root.container.children.push(am5xy.XYChart.new(root, {
    panX: false,
    panY: false,
    wheelX: "panX",
    wheelY: "zoomX",
    layout: root.verticalLayout,
    paddingTop: 16,
    paddingRight: 20,
    paddingBottom: 16,
    paddingLeft: 20,
    colors: am5.ColorSet.new(root, { colors: opts.colors.map((c) => am5.color(c)) })
  }));

  // X 軸：分類軸
  const xRenderer = am5xy.AxisRendererX.new(root, {
    minGridDistance: 15,
    cellStartLocation: 0.1,
    cellEndLocation: 0.9
  });
  xRenderer.labels.template.setAll({
    fontSize: "12px",
    fontFamily: "Helvetica, Arial, sans-serif",
    fill: am5.color("#333333"),
    oversizedBehavior: "truncate",
    maxWidth: 80
  });
  xRenderer.grid.template.set("stroke", am5.color("#90A4AE"));

  const xAxis = chart.xAxes.push(am5xy.CategoryAxis.new(root, {
    categoryField: opts.categoryField,
    maxDeviation: 0.1,
    renderer: xRenderer,
    tooltip: am5.Tooltip.new(root, {})
  }));
  xAxis.children.push(am5.Label.new(root, {
    text: opts.xTitle,
    x: am5.p50,
    centerX: am5.p50,
    fontSize: "12px",
    fontWeight: "bold"
  }));
  xAxis.data.setAll(data);

  // Y 軸：數值軸
  const yRenderer = am5xy.AxisRendererY.new(root, { minGridDistance: 50 });
  yRenderer.labels.template.setAll({ fontSize: "12px", fontFamily: "Helvetica, Arial, sans-serif" });
  yRenderer.grid.template.set("stroke", am5.color("#90A4AE"));

  const yAxis = chart.yAxes.push(am5xy.ValueAxis.new(root, {
    min: 0,
    extraMax: 0.2,
    maxDeviation: 0.1,
    numberFormat: "#.",
    renderer: yRenderer
  }));
  yAxis.children.unshift(am5.Label.new(root, {
    text: opts.yTitle,
    rotation: -90,
    y: am5.p50,
    centerX: am5.p50,
    fontSize: "12px",
    fontWeight: "bold"
  }));

  // 序列：每個數值欄位一個 ColumnSeries
  opts.valueFields.forEach((field) => {
    const series = chart.series.push(am5xy.ColumnSeries.new(root, {
      name: field,
      xAxis,
      yAxis,
      valueYField: field,
      categoryXField: opts.categoryField,
      clustered: true,
      sequencedInterpolation: true,
      tooltip: am5.Tooltip.new(root, {
        pointerOrientation: "vertical",
        labelText: "{name}:{valueY.formatNumber('#.')}"
      })
    }));
    series.columns.template.set("width", am5.percent(100));

    series.bullets.push(() => {
      const label = am5.Label.new(root, {
        text: "{valueY.formatNumber('#.')}",
        populateText: true,
        centerX: am5.p50,
        centerY: am5.p50,
        paddingTop: 8,
        paddingBottom: 8,
        paddingLeft: 10,
        paddingRight: 10,
        fontSize: "14px",
        fontWeight: "bold",
        fill: am5.color("#333333")
      });
      // 柱高不足 5px 或值 ≤ 0 時只隱藏標籤，柱子照畫（與 DIMP 樣式層相同）
      label.adapters.add("visible", (visible, target) => {
        const dataItem = target.dataItem;
        if (!dataItem) return false;
        return dataItem.get("bottom") - dataItem.get("top") >= 5 && dataItem.get("valueY") > 0;
      });
      return am5.Bullet.new(root, { locationX: 0.5, locationY: 0.5, sprite: label });
    });

    series.data.setAll(data);
    series.appear();
  });

  chart.set("cursor", am5xy.XYCursor.new(root, { behavior: "none", xAxis }));

  const legend = chart.children.push(am5.Legend.new(root, {
    x: am5.p50,
    centerX: am5.p50,
    layout: root.gridLayout
  }));
  legend.labels.template.setAll({ fontSize: "14px", fill: am5.color("#333333") });
  legend.data.setAll(chart.series.values);

  chart.appear(1000, 100);
  return root;
};
```

呼叫方式（以圖表 1534 為例）：

```javascript
renderClusteredColumn("chartdiv", rows, {
  categoryField: "轄區分局",
  valueFields: ["SUM(A1)", "SUM(A2)", "SUM(A3)"],
  xTitle: "轄區分局",
  yTitle: "數值",
  colors: ["#6DE082", "#3BC5C5", "#10847C", "#88DFCB", "#48CE98",
           "#1FBACC", "#00AAA9", "#0A94BE", "#5FB4CE", "#B9EEE3"]
});
```

> 範本回傳 `root`；換頁或重繪前呼叫 `root.dispose()` 釋放，避免同一個 div 重複建立 Root。

## 5. JSON 範本的載入方式

需要的腳本：`index.js`、`xy.js`、`plugins/json.js`。

```javascript
const root = am5.Root.new("chartdiv");
const config = await (await fetch("chart-1534-template.json")).json();
const chart = await am5plugins_json.JsonParser.new(root).parse(config, { parent: root.container });
chart.appear(1000, 100);
```

JSON 的結構重點（皆為本次實測踩到、並經原始碼確認的規則）：

| 規則 | 說明 |
| --- | --- |
| `refs` 用陣列分群組 | 5.20.0 中同一群組內的參照**不能互相引用**（`xAxis` 引用 `#data` 會擲出 `Could not find ref #data`）。本範本分三組：`data` → 兩軸 → 三個序列。5.20.7 起同群組可向後引用，但分組仍是保證建立順序的做法 |
| `#`／`@` 開頭的字串要重複前綴 | 字串以 `#` 開頭會被當成參照。`numberFormat: "#."` 必須寫成 `"##."`，解析後才會是 `"#."` |
| 子元素寫在物件頂層的 `children` | 不在 `properties` 內；每個子元素可帶 `index` 決定插入位置。Y 軸標題用 `"index": 0` 對應程式版的 `children.unshift()` |
| 顏色與百分比用物件表示 | `{ "type": "Color", "value": "#333333" }`、`{ "type": "Percent", "value": 50 }` |
| `layout` 可用字串 | `"vertical"`／`"horizontal"`／`"grid"` 會換成 `root.verticalLayout` 等 |
| 樣板設定走 `properties.template` | 例：`renderer.properties.labels.properties.template.settings` |

## 6. 已知差異與限制

1. **JSON 版無法隱藏 0 值與過矮柱子的標籤**：DIMP 以標籤上的 adapter 在柱高 < 5px 或值 ≤ 0 時隱藏標籤（柱子照畫），
   amCharts MCP 文件明載 adapter 不能經由 JSON 往返。JSON 版會顯示 `0` 標籤；需要相同效果時，
   在 `parse()` 之後以程式碼對各序列的標籤補上第 4 節的 adapter。
   （修正紀錄：前一版寫「隱藏柱高不足 20px 的柱子」，經實測柱子實際上都有畫出，已更正。）
2. **圖例項目間距**：原圖圖例較緊湊，範本採 `gridLayout` 預設寬度，三個項目間距較寬；不影響內容。
3. **版本差異**：MCP 文件涵蓋 5.21.0，站台為 5.20.0。下列 5.20.2 之後的功能**本範本未使用**，引入前須先升級：
   `@root.interfaceColors` 參照（5.21.0）、`<am5-chart>` HTML 元素（5.21.0）、`ChartSerializer` 只輸出使用者設定值（5.20.3）。
4. 站台上的 `am5plugins_json.Serializer`（5.20.0）序列化這張圖只輸出 cursor 與背景，不含軸與序列，
   無法用來自動產生範本；JSON 範本為依 MCP 文件手寫後實測解析。

## 7. 驗證紀錄

| 驗證項 | 方式 | 結果 |
| --- | --- | --- |
| 原圖執行期設定 | Playwright 讀取 `am5.registry.rootElements[0]` 的物件設定 | 取得第 2 節的實際值 |
| JSON 範本 | 同頁以 `JsonParser.parse()` 解析 | 1 個 X 軸、1 個 Y 軸、3 個序列，每個 6 筆；序列顏色、標籤值（第二分局 96／272）、圖例 3 項、Y 軸 0～350 皆與原圖相同。截圖：`chart-1534-from-json.png` |
| JS 範本 | 同頁執行本檔第 4 節的 `renderClusteredColumn()`，逐筆比對每個資料點的標籤顯示狀態 | 3 個序列、顏色一致、圖例 3 項；標籤顯示狀態與原圖逐筆相同（`SUM(A1)` 6 個全隱藏，`SUM(A2)`、`SUM(A3)` 12 個全顯示）。截圖：`chart-1534-from-js.png`（2026-10-15 修正後重測） |
| 設定名稱 | 對照本機 5.20.1 原始碼 | `extraMax`、`cellStartLocation`、`oversizedBehavior`、`sequencedInterpolation`、`clustered`、`populateText` 等皆存在 |

原圖截圖：`chart-1534-original.png`
