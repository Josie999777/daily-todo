# daily-todo · Josie Travel Book

| 路径 | 内容 |
| --- | --- |
| `index.html` | 「今日便签」四象限待办应用（含 Supabase 云同步） |
| `texas/index.html` | 德州劳工节旅行手册（2026.9.4–9.8，4 人自驾） |
| `wrangler.jsonc` | `texas` worker 的部署配置（Workers 静态资产模式） |

## 部署（texas 旅行手册）

一次性配置：Cloudflare Dashboard → **Workers & Pages → Create → Workers → Import a repository** → 选择本仓库，生产分支选 `main`：

- Build command：留空
- Deploy command：`npx wrangler deploy`

之后每次 push 到 `main`，Cloudflare 自动构建发布到
**https://texas.josietravelbook.workers.dev** —— 不需要本地 wrangler login。

> `chicago` / `josiechicago` 两个 worker 是早期在 Dashboard 手工创建的，不由本仓库管理。
