## 目的
为自动化编码代理（AI 助手/Co‑pilot）提供一份小而精的入门说明，帮助快速定位本仓库的关键点、常见改动和本地调试步骤，使得对 Vega/Vega‑Lite 規格的修改能立即运行并通过基本验收。

## 一句概览（大图）
这是一个基于 Vega‑Lite 的州/省 choropleth 演示：`index.html` 使用 CDN 加载 Vega 家族并通过 `vegaEmbed()` 加载 `js/state_energy_rate.vg.json`，该 JSON 用 topojson 做地理形状并用 CSV 数据做 lookup 来给地物着色（字段：`renewable_percent`）。

## 关键文件（直接查看）
- `index.html` — 页面入口与 CDN 引入。查这里先判断是否使用在线 CDN 或已被改为本地库。
- `js/state_energy_rate.vg.json` — 主要工作区：topojson source、lookup.transform(from CSV)、encoding (color、tooltip 可编辑)。
- `js/*.topojson` — 地理形状文件（请确认 `feature` 名称与 vg 规格中一致）。
- `data/energy_state_fy_2023.csv` — lookup 数据源（主要字段：`state_code`, `state`, `renewable_percent`）。

## 常见、可直接修改的任务（举例）
- 切换到本地数据：把 `state_energy_rate.vg.json` 中的远程 `data.url` 改为 `js/ne_10m_admin_1_states_provinces.topojson`（topojson）和 `data/energy_state_fy_2023.csv`（CSV）。
- 添加 tooltip：在 `encoding` 中加 `"tooltip": [{"field":"state","type":"nominal"},{"field":"renewable_percent","type":"quantitative"}]`。
- 修改配色/断点：编辑 `encoding.color.scale.scheme`（例如 `"greens"`）或将 scale 改为 `threshold`/`quantile` 并设置 `domain`。
- topojson 注意：`format.feature` 必须与 topojson 文件内部 feature 名称匹配（常见错误点）。

## 本地运行 / 调试（PowerShell）
（不要用 file:// 打开，浏览器会因 fetch/CORS 失败）
推荐：在仓库根目录运行 Python 简单服务器：
```powershell
py -3 -m http.server 8000; # 或 python -m http.server 8000
```
然后在浏览器打开 `http://localhost:8000/index.html`。打开 DevTools 的 Console 和 Network，检查 `state_energy_rate.vg.json`, topojson, csv 三个请求是否返回 200，并查看 Vega 的 runtime 错误。

## 项目约定 / 小规则
- 小改动（路径、tooltip、色带）直接修改 `js/state_energy_rate.vg.json`。
- commit message 约定：前缀使用 `vega: `，例如 `vega: use local topojson and csv for offline dev`。
- 避免更改 `index.html` 的 CDN 版本，除非明确需要脱网或锁定版本；若改动请在 PR 描述中说明原因和版本号。

## 验收（代理在提交前应自动做）
1. 启动本地服务器并加载页面，无 Console 错误。  
2. Network 面板中 `state_energy_rate.vg.json`, topojson, csv 返回 200。  
3. 地图按 `renewable_percent` 着色且（如已加）tooltip 显示 `state` 与 `renewable_percent`。

## 额外说明（供 AI 参考）
- 本仓库没有构建系统或测试框架；修改后只需做浏览器 smoke test。  
- 如果需要离线构建或锁定依赖，可在 issue/PR 中提出并说明受影响文件（`index.html` CDN 行，或新增本地 libs 目录）。

请检查这份指南是否覆盖你的常见工作流，告诉我是否要把示例替换补丁（将远程 URL 替换为相对路径）、或添加一个小的 smoke‑test 脚本/README。 
