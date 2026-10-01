# Cloudflare 部署与域名托管手册

适用对象：本部署包里的“文件管理”静态网站。全程不需要服务器，不需要写代码。
说明：Cloudflare 控制台的菜单名称会随版本调整，下文同时给出中英文名称，遇到对不上时以官方文档为准：https://developers.cloudflare.com/workers/static-assets/

---

## 一、先看这几点

1. **推荐用 Workers 静态资源（Workers Static Assets）部署。** Cloudflare 目前对新项目推荐这种方式，Pages 对已有项目仍可用。纯静态网站的静态资源请求不计费，但额度和价格以 Cloudflare 官方价格页为准。
2. **域名可以没有。** 不绑定域名也能得到一个免费地址：`https://file-manager.你的子域名.workers.dev`。
3. **换地址会丢本地数据。** 本网站的账号和文件存在访问者浏览器里，而浏览器按“网址”隔离数据。从 `workers.dev` 改成自己的域名后，原来上传的文件和账号不会跟过去。请在正式绑定域名之后再开始正式使用。

## 二、准备工作

| 项目 | 说明 |
|---|---|
| Cloudflare 账号 | 到 https://dash.cloudflare.com/sign-up 免费注册，并验证邮箱 |
| Node.js 20 或更高版本 | 仅命令行部署需要，到 https://nodejs.org 安装，装好后在终端输入 `node -v` 检查 |
| 域名（可选） | 已有的，或直接在 Cloudflare 购买 |
| 本部署包 | 解压 `file-manager-deploy.zip`，网站文件在 `public/`，配置在 `wrangler.jsonc` |

## 三、部署网站（三选一）

### 方式 A：命令行直接上传（最简单，推荐）

1. 打开终端，进入解压后的文件夹：
   ```bash
   cd file-manager-deploy
   ```
2. 登录 Cloudflare（会自动打开浏览器，点“允许”）：
   ```bash
   npx wrangler login
   ```
3. 部署：
   ```bash
   npx wrangler deploy
   ```
4. 终端会显示形如 `https://file-manager.xxx.workers.dev` 的地址，打开就能访问。
5. **以后更新网站**：修改 `public/index.html`，再执行一次 `npx wrangler deploy`。

如果项目名 `file-manager` 想改，编辑 `wrangler.jsonc` 里的 `name`。

### 方式 B：连接 GitHub 自动部署

适合以后会反复修改、想“推送即上线”的情况。

1. 把整个文件夹上传到一个 GitHub 仓库。
2. 登录控制台，进入 **Workers 和 Pages（Workers & Pages）** → **创建（Create）** → 选择导入 Git 仓库（Import a repository），授权并选择仓库。
3. 构建命令（Build command）留空；部署命令（Deploy command）保持默认的 `npx wrangler deploy`。
4. 点击部署。之后每次推送到主分支，都会自动更新网站。

### 方式 C：已有 Pages 项目

如果你的账号里已经有 Cloudflare Pages 项目，也可以继续用：项目里选择上传文件夹或连接 Git，构建命令留空，输出目录填 `public`。新项目请优先用方式 A 或 B。

## 四、域名托管

“域名托管”指让 Cloudflare 接管域名的 DNS 解析，这样才能把域名绑定到网站，并获得免费的 HTTPS 证书。

### 情况 1：你已经有域名（在阿里云、腾讯云、GoDaddy 等购买）

1. 控制台首页点击 **添加域（Add a domain）**，输入你的域名，例如 `example.com`。
2. 选择 **免费（Free）** 套餐。
3. Cloudflare 会自动扫描现有的 DNS 记录，检查一遍是否齐全（尤其是邮箱使用的 MX、TXT 记录），缺的手动补上，然后继续。
4. 页面会给出**两个 Cloudflare 名称服务器（Nameserver）**，形如 `xxx.ns.cloudflare.com`。
5. 到你购买域名的注册商后台，找到“修改 DNS 服务器 / 名称服务器”，把原来的服务器**全部替换**为这两个，保存。
6. 回到 Cloudflare 点击检查。通常几分钟到几小时生效，最长可能 24 小时以上。状态变成 **有效（Active）** 就完成了。

> 提示：切换前如果域名正在被网站或邮箱使用，请先把原有 DNS 记录原样迁过来，避免中断服务。

### 情况 2：直接在 Cloudflare 买域名

控制台左侧 **域注册（Domain Registration）** → **注册域（Register Domains）**，搜索并购买。Cloudflare 按成本价销售，购买后域名自动由 Cloudflare 托管，不需要改名称服务器。

### 绑定到网站

1. 进入 **Workers 和 Pages** → 点开 `file-manager`。
2. **设置（Settings）** → **域和路由（Domains & Routes）** → **添加（Add）** → **自定义域（Custom domain）**。
3. 输入要使用的域名：
   - 子域名，如 `files.example.com`（推荐，最稳妥）
   - 或根域名 `example.com`
4. 确认。Cloudflare 会自动创建 DNS 记录并签发 HTTPS 证书，稍等几分钟，状态变为有效后，用 `https://你的域名` 就能访问。

### 同时支持 www 和根域名

两个都想能访问时，可以再添加一个自定义域；或者保留一个主域名，另一个用跳转：
**规则（Rules）** → **重定向规则（Redirect Rules）** → 新建规则，把 `www.example.com` 永久重定向（301）到 `https://example.com`。

### HTTPS 建议设置

在域名页面进入 **SSL/TLS** → **边缘证书（Edge Certificates）**：
- 打开 **始终使用 HTTPS（Always Use HTTPS）**
- 打开 **自动 HTTPS 重写（Automatic HTTPS Rewrites）**

## 五、管理员登录：用变量设置账号和密码

部署包里的 `src/worker.js` 是一个服务器端登录网关：访问网站的任何页面都要先登录，登录页由服务器返回，账号密码**不写在网页里**，而是保存在 Cloudflare 的变量中。

| 变量名 | 类型 | 说明 |
|---|---|---|
| `ADMIN_USER` | 文本或机密 | 管理员用户名 |
| `ADMIN_PASSWORD` | **机密（Secret）** | 管理员密码，请用较长的强密码 |
| `SESSION_SECRET` | 机密，可选 | 登录令牌的签名密钥，填一串随机长字符更稳妥 |

**方法一：在控制台设置（推荐）**
1. 先完成一次部署（第三章），让项目存在。
2. **Workers 和 Pages** → 点开 `file-manager` → **设置（Settings）** → **变量和机密（Variables and Secrets）** → **添加（Add）**。
3. 依次添加上表的变量，密码和密钥的类型选“机密（Secret）”，保存并部署。
4. 刷新网站，会看到登录页，用刚设置的账号密码登录。

**方法二：命令行设置**
```bash
npx wrangler secret put ADMIN_USER
npx wrangler secret put ADMIN_PASSWORD
npx wrangler secret put SESSION_SECRET
```
每条命令执行后按提示输入值。

**以后修改账号或密码**：在同一个位置改变量并保存，原有的登录会立刻失效，所有人需要用新密码重新登录。因为配置里已写了 `keep_vars`，之后再执行 `npx wrangler deploy` 不会把控制台里设置的变量冲掉。

**本地调试**：把 `.dev.vars.example` 复制为 `.dev.vars` 并填写，然后执行 `npx wrangler dev`。`.dev.vars` 已被 `.gitignore` 排除，不要上传到仓库。

**没设置变量会怎样**：网站会显示“尚未设置管理员账号”的提示页，任何内容都不会开放，不会出现默认密码。

**登录的实际保护范围**：它能保证没登录的人打不开网站页面。但网站里的文件仍保存在每个访问者自己的浏览器本地，不在服务器上，换设备看不到。需要多设备同步，要再接入 Cloudflare 的 R2 存储，可以让我继续做。

> 这个登录只在 Cloudflare Workers 部署方式（方式 A、B）下生效。Docker、Netlify、Vercel、GitHub Pages 那几套配置没有这个网关。

## 六、更强的访问保护（可选）：Cloudflare Access

网站自带的登录只在浏览器本地校验，不能防住懂技术的人。如果网站要放在公网，建议用 Cloudflare Access 在网站前面再加一层真正的验证：

1. 控制台进入 **Zero Trust**，首次使用需要设置团队名称，并选择免费套餐（可能需要绑定支付方式，以页面提示为准）。
2. **访问（Access）** → **应用程序（Applications）** → **添加应用程序** → 选择**自托管（Self-hosted）**。
3. 填入你的网站域名（需要是自定义域名）。
4. 新建策略：动作选“允许（Allow）”，规则选“电子邮件（Emails）”，填写允许访问的邮箱。
5. 保存。之后访问网站会先要求输入邮箱，并用邮箱验证码登录，不在名单里的人进不来。

免费额度的人数上限以 Cloudflare 官方页面为准。

## 七、常见问题

| 现象 | 处理办法 |
|---|---|
| 执行 `npx wrangler deploy` 提示找不到入口或配置错误 | 用最新版：`npx wrangler@latest deploy`，并确认 `wrangler.jsonc` 在当前文件夹，且 `src/worker.js` 存在 |
| 部署成功但打开是 404 | 检查 `public/index.html` 是否存在，以及 `wrangler.jsonc` 里 `assets.directory` 是否为 `./public` |
| 域名一直显示“待处理（Pending）” | 名称服务器还没生效，去注册商后台确认已替换，并耐心等待 |
| 自定义域证书一直在“初始化” | 一般几分钟内完成，超过一小时请检查该域名下有没有冲突的 DNS 记录（同名的 A、AAAA、CNAME） |
| 更新后访问还是旧页面 | 强制刷新（Ctrl+F5），或在 Cloudflare 的 **缓存（Caching）** 里选择清除缓存 |
| 换了域名后之前上传的文件看不到 | 正常现象，数据存在原网址对应的浏览器存储里，用回原网址即可看到 |
| 国内访问速度不稳定 | Cloudflare 在中国大陆的访问速度因网络而异，面向国内用户且追求稳定时，请自行实测；使用大陆服务器托管网站则需要办理 ICP 备案 |

## 八、命令速查

```bash
npx wrangler login          # 登录
npx wrangler deploy         # 部署或更新网站
npx wrangler dev            # 本地预览（默认 http://localhost:8787）
npx wrangler deployments list   # 查看部署历史
npx wrangler rollback       # 回滚到上一个版本
```
