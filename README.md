# 文件管理网站 · 部署说明

这是一个纯静态网站（只有 index.html），没有后端和数据库。
用户的文件保存在各自浏览器的 IndexedDB 里，不会上传到服务器。

> 注意：浏览器只在 HTTPS 或 localhost 下提供完整的存储功能，正式上线请启用 HTTPS。

## 方式一：Docker（推荐）
```bash
docker compose up -d --build
# 访问 http://服务器IP:8080
```
停止：`docker compose down`

## 方式二：直接用 Nginx
```bash
sudo mkdir -p /var/www/file-manager
sudo cp public/index.html /var/www/file-manager/
sudo cp nginx.conf /etc/nginx/conf.d/file-manager.conf
# 编辑该文件：root 改为 /var/www/file-manager，server_name 改为你的域名
sudo nginx -t && sudo systemctl reload nginx
```
HTTPS：`sudo certbot --nginx -d your-domain.com`

## 方式三：免费托管
- **Vercel / Netlify**：把整个文件夹拖进控制台，或连接 Git 仓库，无需构建命令。
- **GitHub Pages**：推送到 `main` 分支，并在仓库 Settings → Pages 里选择 “GitHub Actions”，工作流已写好。
- **Cloudflare Pages**：上传文件夹，构建命令留空，输出目录填 `public`。

## 本地预览
```bash
python3 -m http.server 8000 --directory public   # 访问 http://localhost:8000
```

## 文件清单
| 文件 | 用途 |
|---|---|
| public/index.html | 网站本体 |
| Dockerfile / docker-compose.yml | 容器部署 |
| nginx.conf | Nginx 配置（含压缩与安全头） |
| wrangler.jsonc | Cloudflare 部署配置 |
| src/worker.js | 服务器端登录网关（读取变量校验账号密码） |
| .dev.vars.example | 本地调试用的变量示例 |
| Cloudflare部署与域名手册.md | Cloudflare 部署与域名绑定指南 |
| vercel.json / netlify.toml | 托管平台配置 |
| .github/workflows/pages.yml | GitHub Pages 自动部署 |

## 账号与权限（重要）
- 首次打开会要求创建管理员账号；管理员可在页面右上角“用户管理”里添加用户、重置密码、禁用或删除用户、设置角色。
- 每个用户只能看到自己的文件。
- 密码经 PBKDF2 加盐哈希后保存，但账号和文件都存在**访问者自己的浏览器**里，没有服务器端校验。
  所以它适合个人使用、演示和内网小范围试用，**不能**当作多人共用的真正权限系统，也不能防止懂技术的人直接读取浏览器数据。
- 需要多人共享、跨设备访问或真正的安全登录，需要再加后端（用户表、会话、文件存储在服务器）。

## Cloudflare 部署（带服务器端登录）
管理员账号和密码用 Cloudflare 变量设置（ADMIN_USER、ADMIN_PASSWORD），详见手册第五章。未设置变量时网站不会开放。
见 `Cloudflare部署与域名手册.md`。最快的方式：`npx wrangler login` 后执行 `npx wrangler deploy`。
