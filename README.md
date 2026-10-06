# manga-x-web

自托管漫画在线阅读站前端。纯静态单文件，无后端、无数据库；`push` 到 `main` 即自动部署。

## 文件

| 文件 | 说明 |
| --- | --- |
| `index.html` | 全部前端（首页 / 系列页 / 阅读器）。 |
| `.github/workflows/deploy.yml` | push 到 `main` 触发部署。 |

## 功能

- 单页 / 连续两种阅读模式；右→左 / 左→右翻页方向；键盘方向键与触摸手势翻页。
- 目录由服务器 `autoindex_format json` 接口提供（`/books/`），前端据此列出系列与章节。
- 章节按名字自动分组：**普通连载话**（`第N话`、`S2-N`）、**单卷**（`第N卷`）、**SP**（含 `附录`/`动画化`/`特别`/`番外`/`sp`）。
- 记住上次阅读的系列与章节。

## 部署

push 到 `main` 触发 GitHub Actions：把 `index.html` 上传到服务器并做探活校验。也可在 Actions 页手动 `workflow_dispatch`。

仓库 Secrets（`Settings → Secrets and variables → Actions`）：`ECS_HOST`、`ECS_USER`、`ECS_PORT`、`ECS_PASSWORD`。

## 本地开发

前端无构建步骤。但 `fetch('/books/...')` 依赖服务器的 JSON autoindex，直接起普通静态服务器只能看到页面外壳，目录列表需要同样的 `autoindex_format json` 接口。