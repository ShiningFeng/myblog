+++
date = '2025-06-07T15:42:12+08:00'
draft = false
title = '从零搭建个人博客'
tags = ["Memos", "教程"]
summary = "从零搭建个人博客，全面讲解 Hugo 和 GitHub Pages 的部署全过程。"

+++

# 从零搭建个人博客：Hugo + GitHub Pages 全流程指南



## 1. 安装 Hugo（静态网站生成器）

Windows/macOS/Linux 安装 Hugo：

👉 打开 Hugo 官方下载页：https://gohugo.io/getting-started/installing/

后续步骤以 Windows 为例：

1. 下载对应版本的 .zip 解压 → 添加环境变量：

- 按 `Win + S`，搜索 `环境变量`
- 找到「系统变量」里的 `Path`，点击「编辑」
- 点击右侧「新建」，添加 Hugo 的路径（**注意**：添加的是目录，不是 hugo.exe 本身！）

2. 打开命令提示符（CMD）或 PowerShell，输入命令：`hugo version`，如果看到类似输出：

```python
hugo v0.125.4-xxx-xxx Windows/amd64 BuildDate=...
```

说明 Hugo 已成功加入 PATH。



## 2. 配置 Git SSH 推送

配置到 GitHub 的 **SSH 协议**，避免 HTTPS 的登录验证问题，并且更稳定。



### 2.1 生成 SSH 密钥

打开 PowerShell 或 Git Bash：

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

按回车，一路默认。

会在 `C:\Users\你的用户名\.ssh` 目录下生成两份文件：

- `id_rsa`（私钥，别泄露）
- `id_rsa.pub`（公钥）



### 2.2 添加 SSH 公钥到 GitHub

1. 打开 GitHub：
    👉 https://github.com/settings/keys
2. 点击 "New SSH Key"
3. Title 可以随意填，比如：`my-laptop`
4. 将 `id_rsa.pub` 文件内容粘贴进去（可以用记事本打开复制）
5. 点 Save



### 2.3 更换远程地址为 SSH

在 `public/` 目录中运行：

```python
git remote set-url origin git@github.com:YOUR/PATH.github.io.git
```



## 3. 本地新建 Hugo 项目

打开终端（CMD / PowerShell），执行下面的命令：

### 3.1 任选磁盘位置建根目录

```powershell
mkdir D:\hugo-myblog
cd D:\hugo-myblog
```



### 3.2 创建站点

```powershell
hugo new site myblog
```

会看到提示：

```cpp
Congratulations! Your new Hugo site was created...
```



### 3.3 初始化 Git

```powershell
cd myblog
git init
```



### 3.4 添加主题（推荐 PaperModX）

```powershell
git submodule add git@github.com:adityatelange/hugo-PaperMod.git themes/PaperMod
```



### 3.5 配置 `config.toml`

打开终端（CMD / PowerShell），执行下面的命令：

```powershell
notepad config.toml
```



会提示你新建文件，在打开的窗口输入下面的内容：

```toml
baseURL = "http://localhost:1313/"
languageCode = "zh-CN"
title = "Your Blog"
theme = "PaperMod"
enableRobotsTXT = true

[pagination]
  pagerSize = 10

[params]
  author = "Your name"
  showReadingTime = true
  ShowShareButtons = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true
  ShowToc = true
  TocOpen = true
  defaultTheme = "auto"

[menu]
[[menu.main]]
  name = "主页"
  url = "/"
  weight = 1

[[menu.main]]
  name = "归档"
  url = "/archives"
  weight = 2

[[menu.main]]
  name = "标签"
  url = "/tags"
  weight = 3

[markup]
  [markup.highlight]
    noClasses = false
```



**注意**：如果出现问题，用记事本打开检查，重新保存为「UTF-8 无 BOM」编码格式。



### 3.6 写一篇测试文章

```powershell
hugo new posts/hello-world.md
notepad content/posts/hello.md
```



编辑 `content/posts/hello-world.md`：

```md
---
title: "你好，博客世界！"
date: 2025-06-06
tags: ["本地上传", "测试"]
draft: false
---

这是我的第一篇博客文章！欢迎来到我的 Hugo 博客。
```

保存并关闭。



### 3.7 本地预览博客效果

```powershell
hugo server -D
```

浏览器访问 http://localhost:1313/，确保页面能正常显示并加载样式（有标题、菜单、文章列表）。



## 4 GitHub 仓库配置

### 4.1 创建双仓库

| 仓库名         | 用途                                              |
| -------------- | ------------------------------------------------- |
| **myblog**     | 源码仓库（存放  `config.toml`、Markdown、主题等） |
| **.github.io** | 发布仓库（存放 Hugo 构建后的静态页面）            |



**注意：**

1. 第二个仓库名必须是：`你的用户名.github.io`
2. **不需要初始化 README**（保持空仓库）



### 4.2 创建一个 GitHub 个人访问令牌（PAT）

GitHub 的自动生成 `GITHUB_TOKEN` **不允许跨仓库 push**，所以需要 **PAT（GitHub Personal Access Token）**。

1. 打开链接：https://github.com/settings/tokens?type=beta
2. 点击 `Generate new token` → 选择 **Fine-grained token**
3. 填写信息：
   - **Name**：hugo-pages-token
   - **Expiration**：90 days（或自定义）
   - **Repository access**：只选择 `.github.io`仓库
   - **权限设置**：
     - `Contents` → 勾选 **Read and Write**
4. 最后点击 **Generate token**
5. 拷贝生成的 token（只会显示一次！）



### 4.3 添加 token 到 `myblog` 仓库的 Secrets 中

1. 打开 `myblog` 仓库
2. 点顶部导航栏 → `Settings`
3. 左边栏选择 `Secrets and variables > Actions`
4. 点击 `New repository secret`
   - Name: `PERSONAL_TOKEN`
   - Secret: 粘贴你刚生成的 token



## 5. GitHub Pages 部署

### 5.1 连接远程并首次推送源码

```powershell
git remote add origin git@github.com:<USERNAME>/myblog.git
git add .
git commit -m "🎉 初始化 Hugo 博客"
git push -u origin master
```



### 5.2 设置 GitHub Actions 自动部署

创建部署脚本文件

```
mkdir .github\workflows
notepad .github\workflows\deploy.yml
```



在打开的记事本中，复制并粘贴以下内容：

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches: [master]

permissions:
  contents: write            # 允许推送另一个仓库

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.147.0'
          extended: true

      - name: Build site
        run: hugo --minify --baseURL "https://<USERNAME>.github.io/"   # 换成你的线上域名

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
          external_repository: <USERNAME>/<USERNAME>.github.io
          publish_dir: ./public
          publish_branch: master
```

保存后推送：

```powershell
git add .github/workflows/deploy.yml
git commit -m "添加 GitHub Actions 自动部署脚本"
git push
```



### 5.3 启用GitHub Pages 配置

1. 进入你的仓库 `myblog`
2. 点击 `Settings`
3. 左侧点击 **Pages**
4. 查看是否显示如下内容：

```
Your site is live at https://YOURNAME.github.io/
```

如果没有，确保以下设置正确：

- **Source**: Deploy from a branch ✅
- **Branch**: 选择 `master` ✅
- **/ (root)**: 保持默认 ✅
- 点一次 `Save`



### 5.4 等待 Actions 完成

推送成功后，进入 GitHub：

1. 打开 `myblog` 仓库 → 右上角 "Actions" 出现 **Deploy Hugo site to GitHub Pages** 任务
2. 全绿 ✅ 表示构建 + 推送成功
3. 打开 `https://github.com/<USERNAME>/<USERNAME>.github.io` 应看到 `index.html` 等文件



### 5.5 打开 GitHub Pages

`https://<USERNAME>.github.io/` 刷新几次，样式完整即上线成功！



## 6 日常写作工作流

当你想要上线新的笔记或文章时，按照下面操作即可：

1. **创建新文章**：在你的 Hugo 项目中，使用 `hugo new` 命令创建新的 Markdown 文件。例如：

```bash
hugo new posts/my-note.md
```

2. **编辑文章内容**：打开新创建的 Markdown 文件（例如 `content/posts/my-note.md`），使用 Markdown 语法编写你的内容。确保在文件的开头设置正确的元数据，如标题、日期、标签等。并确保设置 `draft: false` 才会被 Hugo 编译到页面中。

```markdown
---
title: "我的新笔记"
date: 2025-06-08T10:00:00+08:00
tags: ["笔记"]
draft: false
---

笔记内容。
```

3. **本地预览**：启动 Hugo 本地服务后打开浏览器访问：`http://localhost:1313/` 即可实时预览。

```powershell
hugo server -D
```

4. **提交到 Git 仓库**：将修改提交到 Git 仓库并推送到远程仓库。

```bash
git add content/posts/my-note.md
git commit -m "添加新笔记：XXX"
git push origin master
```

5. **等待 Actions 自动部署**：

- GitHub Actions 将会自动检测到你的推送并触发自动部署流程。你的新笔记将被构建并发布到 GitHub Pages 上。

6. **访问网站**：

- 打开你的 GitHub Pages 网站地址（例如 `https://<USERNAME>.github.io/`），查看新笔记是否已经成功发布。

