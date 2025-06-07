+++
date = '2025-06-06T23:00:00+08:00'
draft = false
title = '从零实现网站自动签到 + 鸿蒙 PUSH 推送 '
tags = ["Memos", "教程"]
summary = "完整实现网站自动签到流程，并通过鸿蒙 MeoW 实现每日签到结果推送。涵盖 Cookie 抓取、sign 参数解析、状态识别与推送机制。"
+++

本文记录了我如何使用 Python 脚本，自动完成网站的每日签到操作，并将签到结果通过鸿蒙系统的 MeoW 应用推送至手机，真正实现“自动签到 + 即时提醒”的全自动流程。

---

## 1. 项目目标

* 每天自动访问 网站签到页面
* 自动提取网页动态参数 `sign`
* 发送模拟点击请求进行签到
* 解析签到响应判断状态（成功、已签到、Cookie 失效等）
* 使用鸿蒙 MeoW 提供的接口推送签到结果
* 支持通过系统计划任务定时执行脚本

---

## 2. 环境准备

### 2.1 安装依赖库

我们使用 `requests` 进行网页请求，`BeautifulSoup` 进行 HTML 解析，命令如下：

```bash
pip install requests beautifulsoup4
```

---

### 2.2 获取登录 Cookie

访问网站并抓取 Cookie 值：

1. 打开浏览器登录网站，并勾选 “记住我”
2. 按下 `F12` → 进入开发者工具 → 切换到「Application」面板
3. 展开 Cookies → 找到域名 `https://www.网站.com`
4. 提取以下两个字段：

```
bbs_sid=xxxx; bbs_token=xxxx
```

⚠️ 必须在登录状态下获取，否则 Cookie 会很快过期。

---

## 3.  获取鸿蒙 PUSH 接口

升级鸿蒙NEXT系统的手机，去应用市场安装 **MeoW App** 后可获得你的推送链接，格式如下：

```
https://api.chuckfang.com/你的用户名/推送内容
```

将推送内容拼接至接口末尾即可实现消息推送。

---

## 4. 签到流程逻辑详解

1. 访问签到页 `https://www.网站.com/sg_sign.htm`
2. 从 JS 中提取动态参数 `sign`
3. 构造 POST 请求，附带 `sign` 参数发起签到
4. 判断响应文本内容：

   * ✅ 签到成功
   * ⚠️ 已签到
   * ❌ Cookie 失效
5. 发送推送通知至鸿蒙手机

---

## 5. 完整代码

保存为 `daily_auto_sign.py`：

```python
import requests
from bs4 import BeautifulSoup
import re
import datetime

# === 配置区域 ===
COOKIE = "bbs_sid=xxxx; bbs_token=xxxx"  # 替换为你的 Cookie
PUSH_API = "https://api.chuckfang.com/你的用户名"

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
    "Referer": "https://www.网站.com/",
    "Cookie": COOKIE,
}

def send_push(content):
    """推送消息到鸿蒙 MeoW"""
    try:
        url = PUSH_API + content
        requests.get(url, timeout=10)
    except Exception as e:
        print("❌ 推送失败：", e)

def get_sign_token(session):
    """提取签到参数 sign"""
    res = session.get("https://www.网站.com/sg_sign.htm", headers=HEADERS)
    soup = BeautifulSoup(res.text, "html.parser")
    for script in soup.find_all("script"):
        match = re.search(r'sign\s*=\s*"([^"]+)"', script.text)
        if match:
            return match.group(1)
    return None

def do_sign(session, sign_token):
    """发送签到 POST 请求"""
    data = {"sign": sign_token}
    res = session.post("https://www.网站.com/sg_sign.htm", headers=HEADERS, data=data)
    return res.text

def sign_in():
    session = requests.Session()
    now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    sign_token = get_sign_token(session)
    if not sign_token:
        msg = f"[{now}] ❌ 获取 sign 参数失败"
        print(msg)
        send_push(msg)
        return

    result = do_sign(session, sign_token)
    if "签到成功" in result or "恭喜获得" in result:
        msg = f"[{now}] ✅ 签到成功"
    elif "已经签到" in result:
        msg = f"[{now}] ⚠️ 今天已签到"
    elif "请先登录" in result:
        msg = f"[{now}] ❌ Cookie 已失效"
    else:
        msg = f"[{now}] ❓ 未知返回"

    print(msg)
    send_push(msg)

if __name__ == "__main__":
    sign_in()
```

---

## 6. 设置定时任务（Windows）

1. 创建 `run_sign.bat` 文件：

```bat
PATH\TO\YOUR\python.exe D:\你的路径\daily_auto_sign.py
```

2. 打开「任务计划程序」：
   * 创建基本任务
   * 触发器：每天 → 设置时间如早上 08:00
   * 操作：启动程序 → 「程序或脚本」中直接输入：cmd.exe → 「添加参数（可选）」中输入：`/c start "" "\YOUR\PATH\run_sign.bat"`
