# GitHub Pages 静态网站部署指南

## 概述

GitHub Pages 是 GitHub 提供的免费静态网站托管服务。将 HTML/CSS/JS 文件推送到 GitHub 仓库，即可自动部署为公网可访问的网站。

---

## 完整流程（从零开始）

### 第一步：创建 GitHub 仓库

**方式 A：网页创建**
1. 登录 GitHub → 右上角 `+` → `New repository`
2. 填写仓库名（如 `chensiqi-resume2026`）
3. 选择 **Public**
4. 不要勾选 "Add a README"
5. 点击 **Create repository**

**方式 B：API 创建**
```bash
curl -X POST "https://api.github.com/user/repos" \
  -H "Authorization: token <YOUR_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "chensiqi-resume2026",
    "private": false,
    "auto_init": false
  }'
```

### 第二步：准备静态文件

项目结构：
```
chensiqi-resume2026/
├── index.html                    # 主页面（必须有）
├── style.css                     # 样式文件
├── .github/
│   └── workflows/
│       └── pages.yml             # Pages 部署工作流
└── .gitignore
```

> `index.html` 是入口文件，GitHub Pages 默认加载它。

### 第三步：推送代码到 GitHub

```bash
git init
git config user.name "Your Name"
git config user.email "your@email.com"

git remote add origin https://github.com/<username>/<repo>.git
git add -A
git commit -m "feat: initial site"
git branch -M main
git push -u origin main
```

### 第四步：开启 GitHub Pages

**方式 A：网页操作**
1. 打开仓库 → **Settings** → 左侧 **Pages**
2. Source 选择 **GitHub Actions**
3. 保存

**方式 B：API 开启**
```bash
curl -X POST "https://api.github.com/repos/<username>/<repo>/pages" \
  -H "Authorization: token <YOUR_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "build_type": "workflow",
    "source": {"branch": "main", "path": "/"}
  }'
```

### 第五步：添加 GitHub Actions 工作流

创建文件 `.github/workflows/pages.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]          # push 到 main 时自动触发
  workflow_dispatch:           # 允许手动触发（Actions 页面出现 Run workflow 按钮）

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v5
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .              # 部署整个仓库根目录
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

推送后自动触发首次部署：
```bash
git add -A
git commit -m "feat: add Pages workflow"
git push
```

### 第六步：验证部署

```bash
# 查看部署状态
curl -s "https://api.github.com/repos/<username>/<repo>/actions/runs?per_page=1" \
  -H "Authorization: token <YOUR_TOKEN>" | grep -E '"status"|"conclusion"'

# 查看 Pages 状态
curl -s "https://api.github.com/repos/<username>/<repo>/pages" \
  -H "Authorization: token <YOUR_TOKEN>"
```

访问地址：`https://<username>.github.io/<repo-name>/`

---

## 关键配置说明

### `on` — 触发条件
```yaml
on:
  push:
    branches: [main]       # 推送到 main 分支自动部署
  workflow_dispatch:        # 允许在 Actions 页手动触发
```

### `permissions` — 权限
```yaml
permissions:
  contents: read       # 读取仓库代码
  pages: write         # 写入 Pages 服务
  id-token: write      # OIDC 身份验证（GitHub Actions 部署必须）
```

### `path` — 部署目录
```yaml
with:
  path: .              # 部署根目录所有文件
  # path: ./docs       # 只部署 docs/ 目录
```

### `concurrency` — 并发控制
```yaml
concurrency:
  group: pages
  cancel-in-progress: false   # 不取消正在运行的部署
```

---

## 日常更新

修改文件后：
```bash
git add -A
git commit -m "update: 修改内容"
git push
```

推送后 1-2 分钟自动生效。

---

## 国内加速

GitHub 在国内访问较慢，可通过代理 push：
```bash
git remote set-url origin https://gh-proxy.com/https://github.com/<username>/<repo>.git
```

---

## 自定义域名（可选）

1. 在仓库根目录创建 `CNAME` 文件，内容为你的域名：
   ```
   www.yourdomain.com
   ```
2. 在域名 DNS 添加 CNAME 记录指向 `<username>.github.io`
3. GitHub Settings → Pages → Custom domain 填入域名
4. 勾选 **Enforce HTTPS**

---

## 常见问题

| 问题 | 解决方案 |
|------|---------|
| Pages 404 | 检查是否有 `index.html`，确认 Pages 已开启 |
| 部署失败 | 查看 Actions 页面日志，检查 `permissions` 配置 |
| 样式不生效 | 检查 CSS 路径是否正确（相对路径） |
| 更新未生效 | 等待 1-2 分钟，清除浏览器缓存 |
| Token 权限不足 | Classic Token 需要 `repo` 权限 |
