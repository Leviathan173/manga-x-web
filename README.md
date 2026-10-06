# manga-x-web

自托管漫画在线阅读站前端。纯静态单文件，无后端、无数据库。

## 功能

- 单页 / 连续两种阅读模式；右→左 / 左→右翻页方向；键盘方向键与触摸手势翻页。
- 目录由服务器的 JSON 目录接口提供（`/books/`），前端据此列出系列、章节与图片。
- 章节按名字自动分组：**普通连载话**（`第N话`、`S2-N`）、**单卷**（`第N卷`）、**SP**（含 `附录`/`动画化`/`特别`/`番外`/`sp`）。
- 记住上次阅读的系列与章节。
- 访问密码登录（`login.html` + nginx 下发的会话 Cookie），未登录只能看到登录页。

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
   ├── login.html          # 本仓库的 login.html
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

## 登录与反爬

登录门和爬虫拦截都由 nginx 完成，无需后端进程：

- **登录**：`login.html` 把密码 POST 到 `/__login`；密码正确时 nginx 下发 `manga_auth` 会话 Cookie（HttpOnly / SameSite=Lax / Secure / 1 天）。`map $cookie_manga_auth` 校验 Cookie，未登录访问任何非登录路径一律 302 跳登录页（含 `/books/` 与图片，不按路径区分，避免泄露资源是否存在）。`/__logout` 清 Cookie 回登录页。
- **脚本 UA**：`map $http_user_agent` 命中 `curl`/`wget`/`python`/`scrapy`/`httrack`/`aria2` 等或空 UA 时，只在访问日志打 `ua_flag=1`，**不拦截**（UA 可伪造，拦截只有误杀与虚假信心）。
- **缓存**：鉴权靠 Cookie，`/books/` 一律 `Cache-Control: private` + `Vary: Cookie`，防将来前置反代/CDN 缓存后喂给未鉴权者。
- **原有**：广西 GeoIP 白名单、每 IP 限速、限并发连接、限带宽保留。

口令与令牌写在服务器 nginx 配置里（`nginx.example.conf` 中的 `<PASSWORD>` / `<TOKEN>` 占位符）：

```sh
openssl rand -hex 16     # 生成令牌，替换 <TOKEN>
```

说明与天花板：这是**共享令牌**，服务端不做会话过期校验，退出仅清浏览器 Cookie，换令牌需改配置并 reload；密码在 root 只读的 nginx 配置中为明文（纯配置无法校验哈希）。需要多用户账号或签名短会话时，再引入后端鉴权（`auth_request` + 小服务或 njs）。

## 本地开发

前端无构建步骤。但 `fetch('/books/...')` 依赖服务器的 JSON 目录接口，直接起普通静态服务器只能看到页面外壳；要完整预览，可用一份带 `autoindex_format json` 的 nginx 指向本地图片目录。