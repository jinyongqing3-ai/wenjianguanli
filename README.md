# Cloudflare Pages 版（带服务器端登录）

适用于已经用 Pages 部署（地址是 xxx.pages.dev）的项目，保持原来的网址不变。

## 文件
- `public/index.html`：新版网站（含登录联动）
- `public/_worker.js`：登录网关。Pages 发现这个文件，会让每个请求先经过它
- 两个文件必须放在**同一个文件夹**，且该文件夹就是 Pages 的“构建输出目录”

## 步骤
1. 在 GitHub 仓库里，把仓库中原来的 `index.html` 替换成 `public/index.html`，并把 `_worker.js` 放在它旁边。
   （如果原来的 index.html 在仓库根目录，就把两个文件都放在根目录。）
2. Cloudflare：Workers 和 Pages → 你的 Pages 项目 → 设置 → 构建（Builds）
   - 构建命令留空；构建输出目录 = 上一步 index.html 所在的文件夹（根目录填 `/`，放进 public 就填 `public`）
3. 设置 → 变量和机密（Variables and Secrets），环境选“生产（Production）”，添加：
   - `ADMIN_USER`：用户名
   - `ADMIN_PASSWORD`：密码（类型选机密/加密）
   - `SESSION_SECRET`：可选，一串随机长字符
4. 重新部署（在 GitHub 提交一次，或在“部署”页点重试）。变量只对新的部署生效。
5. 无痕窗口打开网址，应先出现登录页。检查：访问 `/api/me` 应显示 `{"user":null}`，不再显示文件管理页面。

## 提示
- 改账号或密码：改变量后重新部署一次，旧登录会失效。
- 网站里的文件仍保存在访问者各自的浏览器里，不在服务器上。
