# Repository Guidelines

Next.js (App Router) Web 应用 — QA 任务管理后台. 文件夹层级组织任务, SQLite 持久化 (`better-sqlite3`, 仅服务端), Ant Design + Tailwind UI. Node 20+ + Bun (`bun.lock`). 改完代码 → 在 `CHANGELOG.md` 顶部新增版本段 → 按 [deploy.md](./deploy.md) 发布.

## AI-only 工程声明

本仓库为 **AI-only 项目**, 人类不参与开发流程:

1. 代码编写 / 测试 / 构建 / 部署 / 发布 — 全部由 Claude Code 或 Codex 执行.
2. 设计决策 (架构 / 技术选型 / 目录结构 / 命名 / 依赖) 以 AI 判断为准, 不需要参考人类开发者惯例, 除非该惯例本身就是 AI 的最优解.
3. 非必要不反问用户, 直接决策并执行 (deploy / 技术抉择 / 文档同步 / 版本号 / changelog 等).
4. 所有文档 (`README` / `CHANGELOG` / `deploy.md` / `AGENTS.md` / 注释) 必须简洁精炼 / 重点突出 / 零冗余 — 能一行不写两行, 能用列表不用段落, 不堆砌背景. 宁可信息密度过载, 不要废话填充.

用户角色 = 触发者 + 验收者, 不是协作开发者.

`package.json#version` 与 tag 必须一致 (Actions 校验, 不一致直接 fail). 详见 [deploy.md](./deploy.md).

## Project Structure & Module Organization

- `app/`: 页面与路由 (App Router). `app/api/` 为 Route Handlers (REST 风格).
- `app/api/admin/`: 管理后台 REST 接口. 资源: `folders` / `tasks` / `jobs` / `settings`. 路由文件位于 `app/api/admin/**/route.ts`.
- `app/api/jobs/[id]/callback/`: 外部 runner 回调入口, 上报 `job_task` 执行结果.
- `app/admin/`: 管理后台 UI. 内部模块按 `_components/` / `_data/` / `_hooks/` / `_services/` / `_utils/` / `_theme/` 分层组织; `_types.ts` 为共享类型.
- `lib/db/`: SQLite 数据访问层 (仅服务端使用, `import 'server-only'`; 客户端必须通过 API 访问).
- `public/`: 静态资源; `docs/`: 参考资料 (含上游 llms.txt 镜像); `data/`: 本地 SQLite 文件 (默认 `data/app.sqlite`, 已 gitignore).

## Key Files

- `lib/db/connection.ts`: 数据库连接 / 建表 / 初始化 / 迁移. 单例存于 `globalThis.__eleAutopilotDb`.
- `lib/db/folders.ts` / `tasks.ts` / `jobs.ts` / `settings.ts`: 各资源 CRUD.
- `app/admin/_components/admin-task-explorer.tsx`: 管理后台主页面状态与交互.
- `app/api/admin/_shared.ts`: 管理后台 API 的通用解析 + 分页响应头 (`Content-Range`) 处理.
- `next.config.ts`: `output: 'standalone'` (release artifact 依赖此模式).

## API Conventions

- 路由形态: `/api/admin/{resource}` 与 `/api/admin/{resource}/[id]`, 资源命名: `folders` / `tasks` / `jobs` / `settings`.
- 列表查询参数 (均 JSON 字符串): `sort` / `range` / `filter`.
- 分页通过响应头 `Content-Range` 表达, 浏览器侧通过 `Access-Control-Expose-Headers: Content-Range` 暴露.
- 新增接口保持资源命名一致, route handler 文件路径与 URL path 严格对应.

## Database

- 表: `folders` (`parent_id` 表层级) / `tasks` (关联 `folders`, `sub_ids` JSON 表子任务链) / `jobs` + `job_tasks` (执行记录) / `settings` (全局配置).
- 已在线上运行, 修改 schema **必须**向后兼容: 用 `ALTER TABLE ... ADD COLUMN`, 禁止 `DROP` / `RENAME` 已有列.
- 迁移代码放在 `lib/db/connection.ts#initSchema()`, 用 `try { ALTER TABLE } catch { /* 列已存在 */ }` 模式保证幂等.
- `data/` 仅本地开发; 线上 SQLite 文件路径用 `SQLITE_DB_PATH` 指向持久化目录, 禁止落在 release tarball 解压目录内 (升级覆盖会丢数据).

## Build & Development

```bash
bun install
bun dev                # http://localhost:3000
bun run build          # 输出 .next/ (含 standalone)
bun run start          # 生产模式 (需先 build)
bun run lint
bun run format         # prettier --write .
```

React DevTools 独立窗口: 必须先 `bunx react-devtools` 再 `bun dev`, 反序无效.

## Coding Style

- TypeScript `strict: true`. 优先使用 `@/*` 路径别名 (如 `@/lib/db`).
- 风格以 Prettier 配置为准 (含 `prettier-plugin-tailwindcss`). 调整格式时跑 `bun run format`, 不要手工对齐 / 排序制造噪音 diff.
- React 组件 `PascalCase` 命名 + 导出; 文件名 `kebab-case.tsx`.
- API 路由按 Next 约定 `app/api/**/route.ts`.

## Testing

未配置测试框架. 改 DB schema / API 形态时通过 `bun run build` + `node .next/standalone/server.js` 启动 + 手动点击验证. 后续接入 vitest / playwright 时回填本节.

## Commit & PR

- Conventional Commits: `feat | fix | chore | refactor | docs | test: ...`.
- 发布提交统一: `release: vX.Y.Z`.
- PR 需含: 变更动机 / 影响范围. UI 改动附截图 (尤其 `app/admin`). API 改动列出请求 / 响应示例. DB schema 改动说明迁移策略.

## Release

- 触发: push `v*` tag. workflow: `.github/workflows/release.yml`.
- artifact: `ele-autopilot-vX.Y.Z-linux-x64.tar.gz` + `checksums.txt`.
- 完整步骤 / amend 场景: [deploy.md](./deploy.md).

## Security & Configuration

- 不要提交 `.env*` / `data/` / `.next/` / `node_modules/` / `*.sqlite*`. 敏感配置放 `.env.local`.
- 环境变量: `SQLITE_DB_PATH` 自定义 SQLite 路径 (默认 `data/app.sqlite`); `HOSTNAME` / `PORT` 控制 standalone server 监听.
- 本地首次启动自动建表 + 写示例数据 (10 顶级文件夹 × 5 子文件夹 + 100 条任务). 重置: 删 `data/app.sqlite`. **线上禁止直接删数据库文件.**

## Boundary

- 当前 release 只产 `linux-x64` artifact. 其它平台需 build 时按需扩展 workflow.
- `CLAUDE.md` 为 `AGENTS.md` 的 symlink, 改 `AGENTS.md` 即可, 不要分叉.
