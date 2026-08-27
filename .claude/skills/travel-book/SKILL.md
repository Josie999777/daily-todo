---
name: travel-book
description: 创建或更新 Josie Travel Book 旅行手册页面（新城市攻略、行程调整、酒店/预订信息回填），并通过 Git 连接的 Cloudflare Worker 发布。Use when the user asks for a new city travel book, itinerary changes, filling in confirmed bookings, or publishing/deploying a travel page.
---

# Josie Travel Book 工作流

一个城市 = 一个目录 = 一个 Worker，`https://<city>.josietravelbook.workers.dev`。
参考实现：`texas/index.html`（做新城市时先读它，直接复用其 CSS 与结构）。

## 页面规范（自上而下的固定结构）

1. **Hero**：emoji + 标题、日期/人数/进出机场一行小字、状态徽章
   （机票/租车/酒店 n/4/门票），外加一条「下一步」动态条（JS 按日期自动切换：
   出发前显示倒计时+下一个里程碑 → 旅途中显示「今天 · D几」→ 结束后显示收尾语）。
   **页面永远保持终稿可读状态：不放修改日志、版本对比、「V2 改了什么」之类的
   元信息**（用户明确要求）——改动说明只写在 git 提交信息和对话里，页面本身
   任何时刻打开都是一份可以直接呈现的最新攻略。
2. **Sticky 锚点导航**：总览 / 航班 / 每日行程 / 待办清单 / 贴士。
3. **🗺️ 行程总览（必须有，用户明确要求）**：内联 SVG 州轮廓地图 +
   按天着色的路线 + 过夜城市实心点（标「夜N」）+ 航班虚线弧 + 图例（每天一行，
   含车程时长）。
4. **航班卡片**：已订航班的时间/航班号/时长；**确认号只写「见邮件」，
   绝不放到公开页面**（workers.dev 任何人可访问，凭确认号+姓氏可改签）。
5. **每日行程卡**：左侧时间列 + 内容，🎫/⭐ 标签标注需订票与亮点，
   底部虚线框「🏨 夜N · 待定/酒店名」。
6. **待办清单**：静态 HTML 行 + `data-id`，JS 勾选并存 localStorage
   （key 形如 `<city>Trip.todos.v1`），顶部进度条；已完成事项用 `data-default="1"`
   预勾选；酒店勾选数同步到 Hero 徽章。
7. **贴士** + footer（注明更新日期）。

样式：暖纸色调色板（抄 `texas/index.html` 的 `:root` 变量），
`prefers-color-scheme` 深浅双主题，单文件零依赖，移动端优先（420px 宽验证）。

## 地图画法

- 等距圆柱投影：`x=(lon-lonMin)*S*cos(中心纬度)`, `y=(latMax-lat)*S`。
- 州轮廓：手工挑 40~50 个边界经纬点；城市/途经点用真实经纬度投影。
- 写个 Python 脚本批量算坐标再粘进 SVG（参考 texas 的做法），
  不要在页面里用 JS 运行时计算。
- 共线的往返路段把 x 偏移 4~5px 避免重叠；右缘标签用 `text-anchor="end"` 防裁切。
- 完成后用 Playwright + `/opt/pw-browsers/chromium` 截图检查
  浅色/深色两种主题下的标签碰撞与裁切。

## 行程编排原则

- **以已出票的航班为锚**：到达/起飞时刻变了就整体平移行程，宁可重排也不硬塞
  （例：下午落地→当晚就近安排大项目；清晨航班→前一晚完成所有游览并住机场方向）。
- 避免深夜赶路；长途日标注轮换驾驶与休息点（德州线的 Buc-ee's 类地标写进行程）。
- 核对开门时间/闭馆日（如 DMA 周一闭馆）、节假日售罄风险、需要 timed entry 的
  免费景点；边境方向注明内陆检查站带证件。
- 酒店未订时在行程卡里放「待定」占位并进待办清单；用户发来预订确认后，
  把酒店名/航班等回填行程卡并把对应待办改为 `data-default="1"`。

## 发布

- 部署走 **Git 连接（Workers Builds）**：改动合入 `main` 即自动发布。
  永远不要尝试 `wrangler login`；不要把 Cloudflare 凭证写进仓库。
- push 前跑 `npx wrangler deploy --dry-run --config <该城市的 wrangler 配置>` 验证。
- 新城市上线（一次性）：建 `<city>/index.html` + `wrangler-<city>.jsonc`
  （`name: <city>`，`assets.directory: ./<city>`），合入 main 后由用户在
  Dashboard「Workers → Create → Import a repository」再连一个项目，
  Deploy command 用 `npx wrangler deploy --config wrangler-<city>.jsonc`。
  （现有 `wrangler.jsonc` 属于 texas，不要动。）
- 远程环境访问不了 `workers.dev` 和 `api.cloudflare.com`（代理封锁），
  验证用 Cloudflare MCP 的 `workers_list` 看 worker 的 `modified_on` 是否更新。
