# daily-todo · Josie Travel Book

个人仓库，两类东西：

1. `index.html` —— 「今日便签」四象限待办应用（Supabase 云同步），独立存在，改动需谨慎。
2. `<city>/index.html` —— Josie Travel Book 系列旅行手册，每个城市一个目录、一个
   Cloudflare Worker，域名规律 `https://<city>.josietravelbook.workers.dev`。
   目前有：`texas/`（2026 劳工节德州 4 人自驾）。`chicago` worker 是在 Dashboard
   手工粘贴的，不在本仓库管理，但可以 curl 其线上页面参考样式。

## 做旅行手册 / 改行程 / 发布

**一切按 `.claude/skills/travel-book/SKILL.md` 执行**——那份是完整操作手册，
涵盖页面规范、SVG 地图画法、行程编排原则、Git 连接部署流程、工具边界。

三条最关键的约定，先记住：
- **不要 `wrangler login`**：部署走 Git 连接，push 到 `main` 自动发布。
- **航司确认号等隐私信息不上公开页面**（workers.dev 任何人可访问）。
- **页面永远保持终稿状态**：不放修改日志/版本说明，变更只写在 git 提交里。

## 换个对话继续工作

本文件与 `.claude/skills/` 会在这个仓库的每个新 Claude Code 会话里自动加载，
所以在 claude.ai/code 对这个仓库开新会话即可无缝接续，不需要复述历史。
要在 claude.ai 普通对话里也拥有同样的上下文，把
`.claude/skills/travel-book/SKILL.md` 上传到账号的 Skills（设置 → Capabilities → Skills）。
