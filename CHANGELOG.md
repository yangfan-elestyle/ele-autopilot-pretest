# Changelog

[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) + [SemVer](https://semver.org/).

## [0.1.0] - 2026-05-18

### Added

- 管理后台 UI (`app/admin`): 文件夹树 + 任务列表的层级管理, 支持任务标题 / sub_ids 子任务链 / 多任务批量创建.
- 数据模型: `folders` (`parent_id` 表层级) / `tasks` (关联 folder, `sub_ids` JSON 表子任务链).
- Job 执行模型: `jobs` + `job_tasks` (一次执行展开 `sub_ids` 后 flat 成多个 `job_task`, 各自记录 `status` / `result` / `error`).
- 全局配置: `settings` (key-value, 默认写入 `agent_config` 含 gemini model / max_steps / 各类 timeout 等).
- 管理后台 REST API (`/api/admin/{folders,tasks,jobs,settings}`): 列表支持 `sort` / `range` / `filter` 查询参数 (均 JSON 字符串), 分页通过响应头 `Content-Range` 表达, 浏览器侧通过 `Access-Control-Expose-Headers: Content-Range` 暴露.
- Job 回调入口: `/api/jobs/[id]/callback` 接收外部 runner 上报 job_task 执行结果.
- SQLite 持久化 (`better-sqlite3`, 仅服务端), 首次启动自动建表 + 写入示例数据 (10 个顶级文件夹 × 5 个子文件夹 + 100 条任务模板).
- DB schema 迁移机制: `initSchema` 内 `ALTER TABLE ... ADD COLUMN` (try/catch 包裹) 幂等处理, 保证已有数据不被破坏.
- `tag (v*)` 触发 GitHub Actions: 构建 Next.js `standalone` 产物, 打包 `linux-x64` tarball, 生成 SHA256 `checksums.txt`, 发布 GitHub Release.

[0.1.0]: https://github.com/yangfan-elestyle/ele-autopilot-pretest/releases/tag/v0.1.0
