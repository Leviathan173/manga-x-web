# manga-x-web

自托管漫画在线阅读站前端。纯静态单文件，无后端、无数据库。

## 功能

- 单页 / 连续两种阅读模式；右→左 / 左→右翻页方向；键盘方向键与触摸手势翻页。
- 目录由服务器的 JSON 目录接口提供（`/books/`），前端据此列出系列、章节与图片。
- 章节按名字自动分组：**普通连载话**（`第N话`、`S2-N`）、**单卷**（`第N卷`）、**SP**（含 `附录`/`动画化`/`特别`/`番外`/`sp`）。
- 记住上次阅读的系列与章节。

## 内容目录约定

前端通过服务器的 JSON 目录接口读取内容，按以下结构放置即可：

```
<站点根>/books/
└── <系列>/
    └── <章节>/
        ├── 1.jpg
        ├── 2.jpg
        └── ...
```

- 图片按**文件名自然排序**（`1`、`2`、…、`10`），支持 `jpg` / `jpeg` / `png` / `webp` / `gif` / `avif`。
- 目录名即系列名 / 章节名；章节分组规则见上。

## 部署

适合自己或合作者从零起一份站点（需要一台装好 nginx 的机器和 root）。

1. 准备好文件和目录：

   ```
   /srv/manga/
   ├── index.html          # 本仓库的 index.html
   └── books/<系列>/<章节>/  # 漫画图片
   ```

2. 安装 nginx 配置（示例见仓库根目录 `nginx.example.conf`）：

   ```sh
   sudo cp nginx.example.conf /etc/nginx/sites-available/manga
   sudo ln -s /etc/nginx/sites-available/manga /etc/nginx/sites-enabled/manga
   sudo nginx -t && sudo systemctl reload nginx
   ```

3. 浏览器访问 `http://<你的域名或 IP>/`。

## nginx 说明

本前端完全依赖服务器提供的 JSON 目录接口，配置要点：

- `root` 指向 `index.html` 所在目录；`/books/` 映射到图片目录。
- `location /books/` 必须开启 `autoindex on;` 和 `autoindex_format json;`。前端请求 `/books/<系列>/`、`/books/<系列>/<章节>/` 并解析返回的 JSON 目录列表；缺了它页面只能显示外壳、列不出内容。
- 只放行 `GET/HEAD`（示例里的 `limit_except`）。
- `index.html` 建议 `no-cache`；图片是静态文件，可按需长缓存。
- 生产环境建议加 TLS，并按自身情况加访问控制与限速。

完整可复制配置见 [`nginx.example.conf`](./nginx.example.conf)。

## 本地开发

前端无构建步骤。但 `fetch('/books/...')` 依赖服务器的 JSON 目录接口，直接起普通静态服务器只能看到页面外壳；要完整预览，可用一份带 `autoindex_format json` 的 nginx 指向本地图片目录。