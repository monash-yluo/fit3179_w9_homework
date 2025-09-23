## 目标
为自动化编码代理（Copilot / AI agent）提供一份小而精的指南，使其能快速在本仓库中定位任务、修改 Vega/Lite 可视化并在本地运行或调试。

## 项目大体结构（重要文件）
- `index.html` — 页面入口，使用 CDN 加载 `vega`, `vega-lite`, `vega-embed`，并把 `js/state_energy_rate.vg.json` 作为 spec 加载到 `#choropleth_map`。
- `js/state_energy_rate.vg.json` — Vega-Lite 规格文件（主要工作区），包含 topojson lookup、transform 与 encoding（color 使用 `renewable_percent`）。
- `js/ne_10m_admin_1_states_provinces.topojson` 和 `js/ne_10m_admin_1_states_provinces1.topojson` — 地图形状（已存在于仓库 `js/`）。
- `data/energy_state_fy_2023.csv` — 与 topojson 做 lookup 的数据源（字段中含 `state_code`, `renewable_percent`, `state`）。

## 大图（设计/数据流）
1. `index.html` 调用 `vegaEmbed('#choropleth_map', 'js/state_energy_rate.vg.json')`。
2. `state_energy_rate.vg.json` 定义了 topojson 的来源和一个 `lookup` transform：
   - 从 topojson 的 feature 中读取 `properties.postal`。
   - 在 CSV（lookup key: `state_code`）中查找并附加字段 `renewable_percent`, `state`。
3. 使用 `geoshape` 标记和 `color` 编码（字段 `renewable_percent`，配色使用 `greens`）渲染州/省的着色。

示例（关键字段）：
- transform lookup: `properties.postal` -> CSV key `state_code`
- color field: `renewable_percent`（标题为“清洁能源比率 (%)”）

## 立刻可运行（开发者/代理操作步骤）
说明：直接用 `file://` 打开 `index.html` 往往会触发浏览器的 fetch/CORS 限制，推荐使用本地静态服务器。提供两种常见方式（Windows PowerShell）：

1) 使用 Python 内置简易服务器（推荐）：
```powershell
# Python 3
py -3 -m http.server 8000
# 或
python -m http.server 8000
```
然后在浏览器打开 `http://localhost:8000/index.html`。

2) 使用 VS Code 的 Live Server 扩展：右键 `index.html` → Open with Live Server。

调试提示：打开浏览器 DevTools 的 Console 和 Network 面板，观察 `state_energy_rate.vg.json`、topojson、CSV 的请求与错误信息；Vega 在运行时也会把错误打印到控制台。

## 常见修改点与范例（对 AI 非常重要）
- 若需要在本地使用仓库内的数据，请把 `state_energy_rate.vg.json` 中的远程 `data.url` 替换为相对路径，例如：
  - topojson: `js/ne_10m_admin_1_states_provinces.topojson`
  - csv: `data/energy_state_fy_2023.csv`
- 添加 tooltip：在 `encoding` 中加入 `tooltip` 字段（示例：`{"field":"state","type":"nominal"}` 或 `[{"field":"state"},{"field":"renewable_percent"}]`）。
- 修改配色或断点：编辑 `encoding.color.scale.scheme` 或将 `type: "quantitative"` 改为 `bin`/`threshold` 并添加 `domain`。

示例替换（可直接写入 `js/state_energy_rate.vg.json`）:
```json
"data": { "url": "js/ne_10m_admin_1_states_provinces.topojson", "format": { "type": "topojson", "feature": "ne_10m_admin_1_states_provinces" } }
// 在 lookup.from.data.url 改为 "data/energy_state_fy_2023.csv"
```

## 集成点、外部依赖与注意事项
- CDN：`vega`, `vega-lite`, `vega-embed` 通过 CDN 加载（在 `index.html`），若无网络或需固定版本，可改为本地包或锁定 CDN 版本。
- Remote data URLs：目前 `state_energy_rate.vg.json` 中的示例使用 raw.githubusercontent 的 URL（会跨域），在离线开发或 CI 中应改为仓库内相对路径。
- TopoJSON `feature` 名称必须匹配 topojson 文件内部的 feature（目前使用 `ne_10m_admin_1_states_provinces`）。修改 topojson 请确认该名称。

## 编辑/提交约定（可被 AI 直接遵循）
- 小改动（如配色、tooltip、路径替换）：在 `js/state_energy_rate.vg.json` 做原子提交，并在 commit message 中写明“vega: <简述>”（例如 `vega: use local topojson and csv for offline dev`）。

## 质量检查（AI 应执行的小型验收）
- 启动本地服务器并在浏览器打开 `index.html`，确保地图能加载且不在 Console 报错。
- 检查 Network 面板，确认 `state_energy_rate.vg.json`、topojson 和 CSV 的请求返回 200。

如果需要，我可以把 `state_energy_rate.vg.json` 的 remote URLs 自动替换为仓库内的相对路径并提交为一个 PR。请告诉我是否要我继续。
