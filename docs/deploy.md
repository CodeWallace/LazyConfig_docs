# 发布到 GitHub Pages

本文档介绍如何将 LazyConfig 文档发布到 GitHub Pages。

## 前置条件

- 已安装 Python 3.8+
- 已安装 Git
- 拥有 GitHub 账号和仓库权限

## 方式一：使用 mkdocs gh-deploy（推荐）

`mkdocs gh-deploy` 是 MkDocs 内置的发布命令，它会自动构建文档并推送到 `gh-pages` 分支。

### 1. 安装依赖

```bash
pip install mkdocs mkdocs-material
```

### 2. 配置 GitHub 仓库

确保你的项目已关联到 GitHub 远程仓库：

```bash
git remote add origin https://github.com/你的用户名/LazyConfig_docs.git
```

### 3. 执行发布

在项目根目录下运行：

```bash
mkdocs gh-deploy
```

该命令会：

1. 自动构建文档到 `site/` 目录
2. 将构建产物推送到 `gh-pages` 分支
3. 几分钟后即可通过 `https://你的用户名.github.io/LazyConfig_docs/` 访问

### 4. 启用 GitHub Pages

在 GitHub 仓库的 **Settings → Pages** 中，确认：

- Source 设置为 `Deploy from a branch`
- Branch 选择 `gh-pages` 分支，目录为 `/ (root)`

## 方式二：使用 GitHub Actions 自动发布

通过 GitHub Actions，每次推送到 main 分支时自动构建并发布。

### 1. 创建工作流文件

在项目根目录创建 `.github/workflows/ci.yml`：

```yaml
name: ci
on:
  push:
    branches:
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure Git Credentials
        run: |
          git config user.name github-actions[bot]
          git config user.email 41898282+github-actions[bot]@users.noreply.github.com
      - uses: actions/setup-python@v5
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV
      - uses: actions/cache@v4
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

### 2. 推送代码

将工作流文件推送到 main 分支：

```bash
git add .github/workflows/ci.yml
git commit -m "Add GitHub Actions deployment"
git push origin main
```

之后每次推送 main 分支，GitHub Actions 会自动构建并发布文档。

## 本地预览

在发布前，可以先在本地预览效果：

```bash
mkdocs serve
```

然后访问 `http://127.0.0.1:8000/` 查看文档。

## 常见问题

### 1. 发布后页面 404？

- 检查 GitHub Pages 设置是否正确指向 `gh-pages` 分支
- 等待几分钟，GitHub Pages 部署需要时间
- 确认仓库名称与 URL 路径一致

### 2. 样式不显示？

- 确保 `mkdocs.yml` 中 `site_url` 配置正确
- 检查仓库名是否与路径匹配（子路径项目需要正确的 base URL）

### 3. 如何自定义域名？

在 `docs/` 目录下创建 `CNAME` 文件，内容为你的域名，例如：

```
docs.lazyconfig.com
```

然后在你的 DNS 服务商处配置 CNAME 记录指向 GitHub Pages。
