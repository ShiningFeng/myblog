+++
date = '2023-10-14T08:00:00+08:00'
draft = false
title = 'Typora 使用技巧：PicGo 图床配置、代码块默认语言、快捷键自定义等'
summary = "本篇笔记整理了 Typora 使用过程中的常用技巧，包括如何配置 PicGo + Gitee 图床、更改默认代码语言、自定义快捷键，以及实现图片居中显示等，适用于提高 Markdown 写作效率的用户。"
tags = ["Memos", "经验"]

+++

# Typora 使用技巧汇总

本文总结了日常使用 Typora 中涉及的几个实用技巧，包括：

* 配置 PicGo + Gitee 实现图床自动上传
* 修改默认代码块语言
* 自定义快捷键
* 图片居中显示的写法

---

## 1.  PicGo + Gitee 图床配置指南

为了让插入的图片自动上传到图床并返回外链，建议使用 PicGo 搭配 Gitee 仓库作为图床。

### 1.1 Gitee 图床仓库配置注意事项

* 使用用户名而非昵称：例如 Gitee 昵称为 `可爱风`，但实际用户名为 `YOUR NAME`，PicGo 中填写时应使用用户名

* 仓库路径格式正确：`repo` 应为 `YOUR NAME/typora_img`，不能包含空格或多余斜杠

* `branch` 填 `master`，有其他分支也可按需填写

* 图片存放路径 `path` 建议设置为 `img` 目录

* 仓库必须设置为公开仓库，否则无法通过外链访问


### 2. 避免 Typora 中上传失败的常见原因

* 不要勾选 Typora 的“使用相对路径”
* 图片路径推荐使用 `![]()` 这种标准 Markdown 格式
* 遇到上传报错时，优先检查：

  * 仓库是否为公开
  * 图片路径是否设置正确
  * 是否在 PicGo 中设置为 Gitee 图床并填写正确的 token

---

## 2. 更改代码块默认语言为 Python

Typora 默认的代码块语言为空，可以修改默认语言：

具体步骤：

1. 找到 Typora 安装目录（Windows 一般为）：

   ```
   PATH\TO\YPUR\TYPORA\resources\app\app\window
   ```

2. 打开.json文件

3. 搜索如下代码片段：

   ```js
   S.localize.getString("code language")+"'></span>",n.childNodes[0].textContent=t||"",t=n,i[0].insertChildAtIndex(t,1)
   ```

4. 修改为：

   ```js
   S.localize.getString("code language")+"'></span>",n.childNodes[0].textContent=t||"python",t=n,i[0].insertChildAtIndex(t,1)
   ```

保存后重新启动 Typora，默认语言即为 `python`。

---

## 3. 自定义快捷键配置

Typora 支持通过配置 JSON 文件来自定义快捷键。

### 1. 打开快捷键配置文件

可在官网说明文档中查找路径，也可以在菜单栏中打开：菜单 → 偏好设置 → 通用 → 快捷键  → 自定义快捷键

### 2. 个人配置内容

```json
  "keyBinding": {
    // for example: 
    // "Always on Top": "Ctrl+Shift+P"
    // All other options are the menu items 'text label' displayed from each typora menu
    "Heading 1": "Ctrl+Shift+1",
    "Heading 2": "Ctrl+Shift+2",
    "Heading 3": "Ctrl+Shift+3",
    "Heading 4": "Ctrl+Shift+4",
    "Heading 5": "Ctrl+Shift+5",
    "Heading 6": "Ctrl+Shift+6",
    "Paragraph": "Ctrl+Shift+0",
    "Ordered List": "Ctrl+Shift+I",
    "Unordered List": "Ctrl+Shift+U",
    "Task List": "Ctrl+Shift+Y",
    "Insert Image": "Ctrl+Shift+P",
    "Outline": "Alt+1",
    "Articles": "Alt+2",
    "File Tree": "Alt+3",
    "Code Fences": "Ctrl+Shift+C",
    "Insert Table": "Ctrl+Shift+T",
    "Strike": "Ctrl+T"
  },
```

---

## 3. 图片居中显示方法

在 Markdown 中居中图片需要用到原始 HTML 标签，格式如下：

```html
<div style="text-align:center">
  <img src="你的图片地址" alt="图片描述" />
</div>
```

Typora 支持原始 HTML 渲染，因此该方法在预览中也能正常显示。

---
