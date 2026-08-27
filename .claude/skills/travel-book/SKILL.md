---
name: travel-book
description: 创建或更新 Josie Travel Book 旅行手册页面（新城市攻略、行程调整、回填酒店/预订信息、发布上线），并通过 Git 连接的 Cloudflare Worker 自动部署。Use when the user asks for a new city travel book, itinerary changes, filling in confirmed bookings, or publishing a travel page.
---

# Josie Travel Book 操作手册

Josie 的个人旅行手册系列：一个城市 = 一个目录 = 一个 Cloudflare Worker，
地址规律 `https://<city>.josietravelbook.workers.dev`。
仓库 `Josie999777/daily-todo`。参考实现：`texas/index.html`（写新城市前先通读它，直接复用其 CSS 与结构）。

已有：`chicago`（2026.7 芝加哥+大瀑布，深蓝夜景风）、`texas`（2026 劳工节德州自驾，暖纸色风）。
仓库根目录的 `index.html` 是另一个独立产品「今日便签」待办应用，与旅行手册无关，不要误改。

## 1. 页面规范（自上而下的固定结构）

1. **Hero**：emoji + 标题、日期/人数/进出机场一行小字、状态徽章（机票/租车/酒店 n/4/门票）。
2. **NOW 卡片**（紧跟 Hero，两个城市统一用这个模式）：
   - 脉冲圆点 + `此刻关注 · NOW` 小标签
   - **下一个行程事件**的标题 + 一行 meta
   - 大号等宽倒计时 `8天 09:29:16 后`，每秒跳动，`font-variant-numeric:tabular-nums`
   - 底部一行「当前待办」= 清单里第一条未勾选项，点击跳 `#todo`
   - 由一张 `EVENTS` 表驱动（每条 `{t: ISO 带时区, title, meta}`），
     所以整趟旅程会自动推进；全部结束后显示收尾语。
     **时间戳必须带时区偏移**（`-07:00` 太平洋 / `-05:00` 中部），否则跨时区看会错。
3. **Sticky 锚点导航**：总览 / 航班 / 每日行程 / 待办清单 / 贴士。
4. **🗺️ 行程总览（必须有，用户明确要求）**：内联 SVG 州/区域轮廓地图 + 按天着色的路线 +
   过夜城市实心点（标「夜N」）+ 航班虚线弧 + 图例（每天一行，含车程时长）。
5. **航班卡片**：时间/航班号/时长。**确认号只写「见邮件」，绝不放公开页面**
   （workers.dev 任何人可访问，凭确认号 + 姓氏即可改签他人航班）。
6. **每日行程卡**：左侧时间列 + 内容，🎫/⭐ 标签标注需订票与亮点，
   底部虚线框「🏨 夜N · 待定/酒店名」。
7. **待办清单**：静态 HTML 行 + `data-id`，JS 勾选存 localStorage（key `<city>Trip.todos.v1`），
   顶部进度条；已完成项用 `data-default="1"` 预勾选；酒店勾选数同步到 Hero 徽章与 NOW 卡片。
8. **贴士** + footer。

样式：暖纸色调色板（抄 `texas/index.html` 的 `:root` 变量），`prefers-color-scheme` 深浅双主题，
单文件零依赖，移动端优先（420px 宽验证）。

**页面永远保持终稿可读状态**：不放修改日志、版本对比、「V2 改了什么」之类的元信息（用户明确要求）。
变更说明只写在 git 提交信息和对话里——页面任何时刻打开都是一份能直接呈现的最新攻略。

## 2. 地图画法

- 等距圆柱投影：`x=(lon-lonMin)*S*cos(中心纬度)`, `y=(latMax-lat)*S`。
- 轮廓：手工挑 40~50 个边界经纬点；城市/途经点用真实经纬度投影。
- 写个 Python 脚本批量算坐标再把结果粘进 SVG，**不要**在页面里用 JS 运行时计算。
- 共线的往返路段把 x 偏移 4~5px 避免重叠；右缘标签用 `text-anchor="end"` 防裁切。
- 完成后用 Playwright + `/opt/pw-browsers/chromium` 截图，检查浅色/深色下的标签碰撞与裁切。

## 3. 行程编排原则

- **以已出票的航班为锚**：到达/起飞时刻变了就整体平移行程，宁可重排也不硬塞。
  （例：下午落地 → 当晚就近安排大项目；清晨航班 → 前一晚完成所有游览并住机场方向。）
- 避免深夜赶路；长途日标注轮换驾驶与休息点（德州线的 Buc-ee's 这类地标写进行程）。
- 核对开门时间与闭馆日（如达拉斯艺术馆周一闭馆）、节假日售罄风险、需 timed entry 的免费景点
  （Alamo）；边境方向注明内陆检查站带证件。
- **热门票要查是否售罄并给替代方案**：例如沃斯堡 Rodeo 售罄 → Billy Bob's Texas
  （周五 20:30/21:30 两场馆内职业骑公牛，普通门票即可看），原项目降级为「刷回流票」候补。
- 酒店未订时行程卡放「待定」占位并进待办清单；用户发来预订确认后，回填酒店名并把对应待办
  改为 `data-default="1"`。

## 4. 发布流程（Git 连接部署）

- Worker 用 **Workers Builds（Git 连接）**：改动合入 `main` 即自动发布，约 1 分钟上线。
  **永远不要尝试 `wrangler login`**（云端会话没有浏览器，OAuth 回跳 localhost 走不通）。
  也不要把 Cloudflare 凭证写进仓库或环境变量（环境变量对所有使用该环境的人可见）。
- push 前跑 `npx wrangler deploy --dry-run --config <配置文件>` 验证配置。
- 流程：改文件 → dry-run → 提交推分支 → 开 PR → 合并进 main → 自动部署 → 用 curl 验证线上。
- **Dashboard 里的 Worker 名必须等于 `wrangler.jsonc` 里的 `name`**，否则构建失败；
  Worker 名同时决定子域名前缀（只能小写字母/数字/连字符，所以「德克萨斯」→ `texas`）。
  在 Dashboard 里 Rename 会保留 Worker ID 与部署历史，是安全操作。
- 新城市上线（一次性）：建 `<city>/index.html` + `wrangler-<city>.jsonc`
  （`name: <city>`，`assets.directory: ./<city>`），合入 main 后由用户在 Dashboard
  「Workers → Create → Import a repository」再连一个项目，
  Deploy command 用 `npx wrangler deploy --config wrangler-<city>.jsonc`。
  现有 `wrangler.jsonc` 属于 texas，不要动。

## 5. 工具边界（谁能做什么）

- **云端 Claude Code 会话（我）**：能改代码、push、开 PR、合并、curl 线上页面验证。
  **不能**操作用户的浏览器，看不到用户屏幕。
- **浏览器插件里的 Claude**：能点 Cloudflare Dashboard（连接仓库、重命名 Worker）。
  凡是必须在 Dashboard 点的事，写一段明确指令让用户转给它。
- **Cloudflare MCP**：`workers_list` 可查 Worker 是否存在及 `modified_on`（验证部署时间）。
  `workers_get_worker_code` 对静态资产 Worker 会报解析错误，别指望它读页面源码——直接 curl。
- **网络出网**：受云环境的 Network access 策略控制。当前环境已设为 **Custom**，放行了
  `*.workers.dev`（可以 curl 线上页面自查），`api.cloudflare.com` 仍被挡（不影响，部署走 Git）。
  策略入口：claude.ai/code → 输入框上方云朵按钮 → 悬停环境 → 齿轮 → Network access
  （None / Trusted / Full / Custom）。

## 6. 常见任务

| 用户说 | 怎么做 |
| --- | --- |
| 酒店订好了（发来确认信息） | 回填对应「夜N」行程卡的酒店名与地址，待办改 `data-default="1"`，push 合并 |
| 某个票买不到 / 改时间 | 找替代方案并核实（WebSearch 查官网场次），改行程卡 + 待办 + 地图图例文案，保持一致 |
| 加一个新城市 | 复制 `texas/index.html` 结构 → 新目录 + wrangler 配置 → 引导用户在 Dashboard 连接 |
| 页面怎么不更新 | 查 PR 是否合进 main；用 `workers_list` 看 `modified_on`；curl 线上页面看内容 |
