---
name: github-pages
description: GitHub Pages 静态网站部署。用于将 HTML/CSS/JS 静态文件部署到 GitHub Pages 公网访问。触发场景：部署静态网站到 GitHub Pages、创建 GitHub Pages 项目、配置 Pages 工作流、更新 Pages 网站。
---

# GitHub Pages 部署 Skill

## 完整部署流程

### 1. 创建仓库

```bash
curl -X POST "https://api.github.com/user/repos" \
  -H "Authorization: token <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"name": "<repo>", "private": false}'
```

### 2. 推送静态文件

```bash
git init
git remote add origin https://<TOKEN>@github.com/<user>/<repo>.git
git add -A
git commit -m "feat: initial site"
git branch -M main
git push -u origin main
```

### 3. 开启 Pages

```bash
curl -X POST "https://api.github.com/repos/<user>/<repo>/pages" \
  -H "Authorization: token <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"build_type": "workflow", "source": {"branch": "main", "path": "/"}}'
```

### 4. 添加部署工作流

创建 `.github/workflows/pages.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

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
          path: .
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

推送触发部署：
```bash
git add -A && git commit -m "feat: add Pages workflow" && git push
```

### 5. 验证

访问地址：`https://<username>.github.io/<repo>/`

## 关键配置

- `on.push.branches`: 自动触发的分支
- `workflow_dispatch`: 允许手动触发
- `path`: 部署目录（`.` = 根目录）
- `permissions`: 必须包含 `pages: write` 和 `id-token: write`

## 国内加速

```bash
git remote set-url origin https://gh-proxy.com/https://github.com/<user>/<repo>.git
```

## 自定义域名

根目录创建 `CNAME` 文件，DNS 添加 CNAME 指向 `<username>.github.io`。
