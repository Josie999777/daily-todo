# daily-todo · Josie Travel Book

个人仓库，两类东西：

1. `index.html` —— 「今日便签」四象限待办应用（Supabase 云同步），独立存在，改动需谨慎。
2. `<city>/index.html` —— Josie Travel Book 系列旅行手册，每个城市一个目录、一个
   Cloudflare Worker，域名规律 `https://<city>.josietravelbook.workers.dev`。
   目前有：`texas/`（2026 劳工节德州 4 人自驾）。早期的 `chicago`/`josiechicago`
   worker 是在 Dashboard 手工粘贴的，不在本仓库管理。

## 部署模型（重要）

- Worker 用 **Git 连接部署**（Workers Builds）：push 到 `main` 即自动发布，
  不要尝试 `wrangler login`（远程会话没有浏览器），也不要在 Dashboard 之外找部署凭证。
- `wrangler.jsonc` 是 `texas` worker 的配置（静态资产模式，目录 `./texas`）。
- 改完页面用 `npx wrangler deploy --dry-run` 验证配置，再 push。
- 本仓库的远程 Claude 环境网络代理封锁 `workers.dev` 与 `api.cloudflare.com`，
  无法 curl 线上页面；验证部署结果用 Cloudflare MCP 的 `workers_list`。

## 做旅行手册 / 改行程

按 `.claude/skills/travel-book/SKILL.md` 的流程和页面规范执行。
关键约定：中文页面、暖纸色深浅双主题、顶部必须有 SVG 地图行程总览、
待办清单可勾选且存 localStorage、**航司确认号等隐私信息不上公开页面**。
