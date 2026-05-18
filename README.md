# ele-autopilot

React Router v7 (Framework mode) Web 应用 — QA 任务管理后台. 文件夹层级组织任务, Cloudflare D1 持久化 (prepared statement, async), Cloudflare R2 存截图, Ant Design + Tailwind UI. Node 22+ + Bun. 运行时 Cloudflare Workers (V8 isolate). 改完代码 → 在 `CHANGELOG.md` 顶部新增版本段 → 按 [deploy.md](./deploy.md) 发布 (push `v*` tag → Actions → `wrangler deploy`).

## 部署

push `v*` tag 触发 GitHub Actions: 校验版本 → build → `wrangler d1 migrations apply --remote` → `wrangler deploy`. 部署成功后 Worker URL: `https://ele-autopilot.<account-subdomain>.workers.dev`.

前置 (一次性):

- GitHub Secrets: `CLOUDFLARE_API_TOKEN` (Workers / D1 / R2 编辑权限) + `CLOUDFLARE_ACCOUNT_ID`.
- D1 database (`ele-autopilot`) / R2 bucket (`ele-autopilot-screenshots`) 手动一次性创建, `database_id` 写入 `wrangler.jsonc`.

详见 [deploy.md](./deploy.md).

## 开发

```bash
bun install
bunx wrangler d1 migrations apply ele-autopilot --local   # 首次: 初始化 miniflare D1 schema
bun dev                                                    # http://localhost:3000 (Vite + cloudflare 插件 + miniflare D1/R2)
bun run lint
bun run format
bun run typecheck                                          # wrangler types + react-router typegen + tsc
bun run build && bun run preview                           # wrangler dev 模拟生产产物
```

React DevTools 独立窗口: 必须先 `bunx react-devtools` 再 `bun dev`, 反序无效.

## 发布

详见 [deploy.md](./deploy.md). 一条线:

```bash
# 1. 改 package.json#version (与即将打的 tag 不含 v 保持一致)
# 2. CHANGELOG.md 顶部新增 ## [X.Y.Z] - YYYY-MM-DD 段
git commit -am "release: vX.Y.Z"
git tag -a vX.Y.Z -m "vX.Y.Z"
git push origin <branch> vX.Y.Z
```
