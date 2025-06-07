+++
date = '2024-10-24T08:00:00+08:00'
draft = true
title = "解决训练中 PermissionError 报错：Ubuntu /tmp 缓存空间占满引发的多进程失败"
tags = ["Memos", "经验"]
summary = "一次通过 SSH 远程连接 Ubuntu 服务器训练模型时遇到的 PermissionError 报错，追踪发现是 /tmp 缓存目录空间占满导致多进程通信失败。详细记录诊断和解决过程。"
+++

# 解决训练中 PermissionError 报错：Ubuntu `/tmp` 缓存占满引发的多进程失败

近期在通过 SSH 远程连接 Ubuntu 服务器运行深度学习训练脚本时，遇到了如下报错：

```text
PermissionError: [Errno 13] Permission denied
...
self._listener = Listener(authkey=process.current_process().authkey)
...
```

表面看似是多进程通信（如 multiprocessing、DataLoader）权限问题，但实际原因并非网络端口或 SELinux，而是 **系统 `/tmp` 缓存目录被占满**，导致多进程无法创建 socket 临时文件。

---

## 1. 报错背景

* 环境：远程 Ubuntu 服务器，SSH 登录执行训练脚本
* 任务：使用 PyTorch，DataLoader 设置了 `num_workers > 0`
* 报错点：多进程通信（如 socket）初始化阶段
* 初步尝试：修改端口、关闭防火墙、调整权限，均无效

最终通过排查发现是 **`/tmp` 目录空间耗尽** 所致。

---

## 2.  排查步骤

### 2.1 查看 /tmp 使用情况

```bash
df -h /tmp
```

输出示例：

```bash
Filesystem      Size  Used Avail Use% Mounted on
tmpfs            32G   32G     0 100% /tmp
```

* Use% 高达 100%，说明 /tmp 分区空间被占满，无法继续写入
* 这会导致多进程无法创建 IPC socket、FIFO 文件，从而报 PermissionError

---

### 2.2 找出占用空间的文件/目录

```bash
sudo du -sh /tmp/* | sort -hr | head -n 20
```

* `du -sh /tmp/*`：统计 `/tmp` 下各文件/目录大小
* `sort -hr`：按大小从大到小排序
* `head -n 20`：只显示前 20 个最大文件/目录

输出示例：

```bash
34G /tmp/JDPL5kiW7A
27G /tmp/ronT5OO6Dl
...
```
这些目录可能是之前数据解压/缓存/中断任务遗留的临时文件。

---

### 2.3 检查是否有进程仍在使用这些目录
为了防止误删被使用中的临时文件，可以用 lsof 排查：

```bash
sudo lsof | grep /tmp/JDPL5kiW7A
sudo lsof | grep /tmp/ronT5OO6Dl
```

* 无输出 表示该目录已无进程占用，可考虑删除
* 若有输出，说明还有任务在使用，请暂缓处理或重启后操作

---

### 2.4 安全删除占用目录
确保无进程使用后，执行清理操作：

```bash
sudo rm -rf /tmp/JDPL5kiW7A
sudo rm -rf /tmp/ronT5OO6Dl
```

* `-r` (recursive) 递归删除，`-f` (force) 不询问。
* 确保没有进程正在使用这些目录，一旦删除不可恢复。

然后再次检查 /tmp 使用情况：

```bash
df -h /tmp
```

此时应看到可用空间大幅回升。

---

### 2.5 重试训练任务
恢复后再次运行你的训练脚本：

```bash
python train.py
```

* 若使用 PyTorch，请确保 num_workers > 0；
* 正常情况下，不再报 PermissionError，说明多进程 socket 成功初始化。

---

##  3. 参考建议

* 可通过以下命令设置每周自动清理 `/tmp`：

```bash
sudo systemctl enable tmp.mount
```

* 或将临时目录指向另一个挂载点：

```bash
export TMPDIR=/data/tmp
```

然后将 PyTorch 的缓存、临时写入目录也指向此处。

---

如你也在训练中遇到类似的多进程失败问题，不妨也从系统 `/tmp` 空间使用情况入手排查 —— 很多“神秘报错”，背后其实是存储瓶颈。
