# 部署到 Cloudflare Pages（GitHub + Edge）

> 这份文档只讲发布。网站本身的设计说明在 `README.md`。

## 0. 结论：这个网站适合 Cloudflare Pages 吗？

**适合，而且是最省事的一类：纯静态站点。**

| 检查项 | 结果 |
| --- | --- |
| 形态 | 1 个 `public/index.html`（HTML + 内联 CSS/JS）+ 5 张 JPG + 2 个文本文件 |
| 体积 | 整站 **1.68 MB**（Pages 限制：单文件 25 MiB、20,000 个文件） |
| 构建 | **不需要**：没有 npm、没有打包器、没有框架 |
| 后端 | 不需要：没有任何 API、表单提交走 `mailto:`、无数据库 |
| 路由 | 单页，不需要 SPA 回退规则 |
| 路径 | 资源全是相对路径（`assets/...`），挂在 pages.dev 根域或自定义域名都正常 |
| 外部依赖 | Google Fonts（`fonts.googleapis.com` / `fonts.gstatic.com`）。已写系统字体回退，字体拉不到时版面不变 |
| 安全/缓存 | 已加 `public/_headers`（nosniff、Referrer-Policy、图片长缓存）与 `public/robots.txt` |

两个已知代价（不影响发布，只影响体验）：

1. **中国大陆访问**：`*.pages.dev` 和 Google Fonts 在大陆都不稳定，可能需要代理；页面本身会回退到系统中文字体，不会白屏。
2. **概念页标识**：页面用的是 DeepSeek 的名字和鲸鱼意象，页脚已写「本页是 DeepSeek 4.1 Flash 的概念发布页」。如果要公开传播，建议再加「非官方」三个字，避免被当成官方发布（要加的话告诉我）。

---

## 1. 当前状态（我已经做好的部分）

- 站点已整理成「发布目录 + 文档」结构：**只有 `public/` 会被发布**
  ```
  deepseek-4.1-flash-v2/
    public/                 ← Cloudflare Pages 的输出目录
      index.html
      _headers
      robots.txt
      assets/
        img/*.jpg
        manifest.json
        CREDITS.md
    README.md
    COPY.md
    DEPLOY.md
    .gitignore              ← _shots/、.taste-skill/ 不进仓库
  ```
- 本地已初始化 git 仓库，分支 `main`，完成首次提交。
- 远程地址已配置为：`https://github.com/zacharypan87-spec/ds-4.1-flash-1017_web_test.git`
  （如果这不是你的 GitHub 用户名/想要的仓库名，改掉即可，见 §2.4）
- **已推送并验证**：代码已经在 **<https://github.com/zacharypan87-spec/ds-4.1-flash-1017_web_test>** 上（公开仓库，`main` 分支，提交 `00ee167`）。我逐文件核对过 raw 内容：`public/index.html` 200 / 65,927 字节 / 标题正确，5 张图片都是 `image/jpeg` 200，`_headers`、`robots.txt` 也都是 200。
- 本地已配置好 upstream：`git status` 显示 `## main...origin/main`，以后 `git push` 即可更新。

### 已发布（2026-10-20）

- **线上地址：<https://ds-4-1-flash-1017-web.pages.dev>**（Cloudflare Pages，免费）
- 项目名：`ds-4-1-flash-1017-web`（Cloudflare 账号：Zacharypan87@gmail.com）
- 发布方式：**Wrangler Direct Upload**（`wrangler pages deploy public`）。Cloudflare 2026 新界面把 Pages 的 Git 导入收进了 Workers 流程，界面里“导入 Git 仓库”建出来的是 Worker，而 `*.workers.dev` 在本机网络被解析到 127.0.0.1 / 502，所以改用官方 CLI 直接发 Pages。
- 发布检查：首页 200 · 65,927 字节 · 标题正确；5 张图片 200 `image/jpeg`；`_headers` 生效（nosniff / Referrer-Policy / 图片 immutable 缓存）；Edge 1440 与 390 两个宽度 26 个请求、0 个非 200、0 失败、0 横向溢出、字体已加载。截图：`_shots/live-check/live-1440.jpg`、`live-390.jpg`。

### 已用 Edge 做过的本地验收（发布前的基线）

我在这台机器上用 **Microsoft Edge（headless，155.0.4283.62）** 分别以 HTTP 服务、`file://` 两种方式打开 `public/index.html`，1440×900 与 390×844 两个尺寸：

| 检查 | 结果 |
| --- | --- |
| 页面标题 | `DeepSeek 4.1 Flash 1017：思考不必等待` ✅ |
| 5 张图片 | 全部加载（naturalWidth > 0）✅ |
| 横向溢出 | 1440/390 均为 0px ✅ |
| 请求 | 26 个请求，非 200：0，失败：0 ✅ |
| 控制台错误 | 0 ✅ |
| Google Fonts | 已加载（Noto Sans SC）；断网时会回退系统字体 |
| 截图 | `_shots/edge-check/http-desktop.jpg`、`http-mobile.jpg`、`file-desktop.jpg` |

> 也就是说：**这一份文件直接传到任何静态托管都能跑**；剩下要解决的只是 GitHub 授权本身。

---

## 2. 第一步：把代码放到 GitHub（在 Edge 里操作）

### 2.1 创建一个空仓库

1. 用 **Edge** 打开 <https://github.com/new>（没登录就先登录，账号应是 `zacharypan87-spec`）。
2. **Repository name** 填：`deepseek-4.1-flash-1017`
3. 可见性：**Public**（私有仓库也能接 Cloudflare Pages，但公开仓库授权更简单）
4. **不要**勾选 “Add a README file / Add .gitignore / Choose a license” —— 仓库必须是空的，否则第一次推送会冲突。
5. 点 **Create repository**，复制页面上显示的 HTTPS 地址（形如 `https://github.com/zacharypan87-spec/ds-4.1-flash-1017_web_test.git`）。

### 2.2 推送（在 PowerShell 里执行一条命令）

如果仓库名/用户名与上面一致，本地已经配好远程，直接：

```powershell
cd C:\Users\张锦鹏\Desktop\Mv\deepseek-4.1-flash-v2
git push -u origin main
```

- 第一次推送会弹出 **Git Credential Manager** 窗口，选 **Sign in with your browser** → 登录 GitHub → **Authorize**。
  - 这台机器的默认浏览器是 Chrome，所以弹窗可能打开 Chrome；在 Chrome 里登录 GitHub 同样可以完成授权（不影响之后用 Edge 打开 GitHub/Cloudflare）。
  - 想强制走 Edge：先临时把 Edge 设为默认浏览器；或者在 GCM 窗口里选 **Use a token**，粘贴 §2.5 的 PAT。
  - 授权成功后回到终端，看到 `main -> main` 即完成。
- 看到 `main -> main` 就成功了。

如果用户名/仓库名不同，先改远程地址再推送：

```powershell
git remote set-url origin https://github.com/<你的用户名>/<你的仓库名>.git
git push -u origin main
```

### 2.3 确认真的在 GitHub 上

刷新仓库页面，应该能看到 `public/`、`README.md`、`COPY.md`、`DEPLOY.md`、`.gitignore`。
也可以把仓库地址发给我，我直接查 GitHub API 确认提交存在。

### 2.5 备选：用 Personal Access Token（PAT）推送

如果不想在 GCM 里走浏览器授权：

1. 在 **Edge** 打开 <https://github.com/settings/tokens> → **Generate new token (classic)**。
2. Note 随便写，Expiration 选 7 天，勾选 **`repo`**（或只勾 `public_repo`）→ Generate → 复制 `ghp_...`。
3. 运行 `git push -u origin main`，在 GCM 弹窗里选 **Use a token**，用户名填 `zacharypan87-spec`，密码粘贴 PAT。
4. 推送完成后建议立刻在 GitHub 上 Revoke 这个 token（或等它 7 天过期）。

### 2.4 备选：不想配 git 凭据（Edge 网页上传）

在空仓库页面 → **Add file → Upload files** → 把下面这些拖进去 → **Commit changes**：

- `public/` 整个文件夹（必须保留 `public/` 这一层）
- `README.md`、`COPY.md`、`DEPLOY.md`、`.gitignore`

> 网页上传也能用，但以后更新要重复拖拽；推荐还是走 §2.2 的 git 推送。
> 如果愿意装 GitHub CLI：`winget install --id GitHub.cli -e`，然后 `gh auth login`（选 GitHub.com → HTTPS → Login with a web browser），告诉我一声，我可以直接用 `gh` 建仓库并推送。

---

## 3. 连接 Cloudflare Pages

> **实际采用的方式（已发布）**：Cloudflare 新 UI 的 Git 导入会生成 Worker（`workers.dev` 在部分网络不可达），因此本项目用官方 CLI 直传 Pages。以后更新执行：
>
> ```powershell
> cd C:\Users\张锦鹏\Desktop\Mv\deepseek-4.1-flash-v2
> npx --yes wrangler@4 pages deploy public --project-name=ds-4-1-flash-1017-web --branch=main --commit-dirty=true
> ```
>
> 第一次在这台机器上需要 `npx wrangler@4 login`（浏览器里点 Allow 即可，已做过）。
>
> 下面 §3 的界面步骤仅作参考（如果你的 Cloudflare 账号仍能看到传统 Pages Git 表单，也可以按它走）。

1. 用 **Edge** 打开 <https://dash.cloudflare.com/>，注册/登录（免费计划即可，**不需要先买域名**）。
2. 左侧菜单 → **Workers & Pages** → 右上 **Create application** → 切到 **Pages** 标签 → **Connect to Git**。
3. 点 **Connect GitHub** → **Install & Authorize**，在 GitHub 授权页里选 **Only select repositories** → 选 `ds-4.1-flash-1017_web_test` → **Install**。
4. 回到 Cloudflare，选中这个仓库 → **Begin setup**。
5. 按下表填写（**这就是针对当前网站的正确值**）：

| 字段 | 填什么 | 说明 |
| --- | --- | --- |
| Project name | `ds-4-1-flash-1017-web` | 会变成 `https://ds-4-1-flash-1017-web.pages.dev`；只能小写字母/数字/连字符 |
| Production branch | `main` | 本地分支已叫 main |
| Framework preset | **None** | 没有框架 |
| Build command | **留空** | 没有构建步骤，不要填 `npm run build` |
| Build output directory | `public` | 站点在 `public/`，这是唯一的必填项 |
| Root directory | 留空（= `/`） | 仓库根目录就是项目目录 |
| Environment variables | 不填 | 没有需要注入的东西 |

6. 点 **Save and Deploy**，等 30–60 秒。
7. 顶部会出现免费网址：`https://ds-4-1-flash-1017-web.pages.dev`（项目名不同则前缀不同）。

---

## 4. 发布后检查（已完成）

线上 URL：<https://ds-4-1-flash-1017-web.pages.dev>。检查结果见 §1「已发布」与 `_shots/live-check/report.json`。以下为检查清单：

把 `https://xxxx.pages.dev` 发给我，我会用 **Edge（headless）+ HTTP** 检查：

- [ ] 首页返回 **200**，HTML 里含 `DeepSeek 4.1 Flash 1017` 标题
- [ ] 5 张图片全部 200 且 `Content-Type: image/jpeg`
- [ ] `assets/manifest.json`、`_headers` 生效（响应头里有 `X-Content-Type-Options`）
- [ ] Google Fonts 是否可达（不可达时应回退系统字体，页面仍完整）
- [ ] Edge 截图首屏 / 玻璃工具栏 / 请求卡片，确认无 404、无控制台报错、无横向溢出
- [ ] HTTPS 证书与 `pages.dev` 解析正常

我也可以顺便跑一遍移动端（390px）截图确认。

---

## 5. 自定义域名（可选）

1. Pages 项目 → **Custom domains** → **Set up a custom domain**。
2. 输入你的域名（如 `flash.example.com`）。
3. 分两种情况：
   - **域名已托管在 Cloudflare**：会自动加 CNAME，等 1–5 分钟即可。
   - **域名在别处**：去域名服务商加记录 —— 子域名加 `CNAME → <project>.pages.dev`；根域（apex）多数服务商不支持 CNAME，建议用 `www` 子域，或把域名的 NS 迁到 Cloudflare。
4. Cloudflare 会自动签发并续期 HTTPS 证书，无需手工配置。
5. 换域名后不需要改代码：页面资源全是相对路径。

---

## 6. 以后更新

```powershell
cd C:\Users\张锦鹏\Desktop\Mv\deepseek-4.1-flash-v2
# 1) 改 public/index.html 或替换 public/assets/img/... 里的图片
# 2) 提交到 GitHub
git add -A
git commit -m "update copy"
git push
# 3) 重新发布到 Cloudflare Pages（Git 自动部署在本项目里没有启用，需要手动这一条）
npx --yes wrangler@4 pages deploy public --project-name=ds-4-1-flash-1017-web --branch=main --commit-dirty=true
```

Cloudflare 检测到 `main` 分支新提交后会自动重新部署，约 30 秒后 pages.dev 域名生效。
需要回滚时，在 Pages 项目的 **Deployments** 里对任一历史版本点 **Rollback**。

---

## 7. 备注

- 不需要 Cloudflare 的构建镜像/Node 版本/环境变量；这个站点没有任何构建产物。
- 不建议把 `_shots/`、`.taste-skill/` 一起发布：`.gitignore` 已经排除，`public/` 里也没有它们。
- 如果以后想把字体也本地化（彻底去掉 Google Fonts 依赖），我可以把字体子集下载进 `public/assets/fonts/` 并改成 `@font-face`，这样中国大陆访问也不会掉字体。
