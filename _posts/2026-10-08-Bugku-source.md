---
layout: post
title: "Bugku CTF - source 题目"
date: 2026-10-08
categories: [CTF, WriteUp]
---

> **题目类型：** Web 基础 / Git 源码泄露  
> **使用工具：** 浏览器（Chrome/Edge）、Git Bash、git-dumper  
> **核心方法：** 利用 Git 源码泄露漏洞（.git 目录泄露），还原网站源码，通过 Git 历史提交记录寻找 Flag

---

## 一、 题目分析

访问题目地址，页面只显示了简单的 `Hello world`。右键查看网页源代码（`Ctrl + U`），发现了一段注释和编码：

```html
<p>This is my friend :<!--tig--></p>
<!-- flag{Zmxhz19ub3RfaGvyzSEHIQ==} -->
```

1. 尝试解码 `Zmxhz19ub3RfaGvyzSEHIQ==`（Base64），得到结果 `flag_is_not_here!`。这是一个迷惑项，提示真正的 Flag 不在这里。
2. 注意到注释里的 `<!--tig-->`，结合题目名 `source`，猜测这是提示 Git 相关的线索（`tig` 是 `git` 的变形）。
3. 尝试访问 `http://靶场IP:端口/.git/`，页面返回了目录列表（`Index of /.git/`），确认存在 Git 源码泄露漏洞。

---

## 二、 解题步骤

### 步骤 1：确认泄露并查看配置
访问 `http://160.202.254.160:10418/.git/config`，看到了 Git 配置信息，其中包含了用户信息 `email=flag@flag.com`，证实了这是一个完整的 Git 仓库。

### 步骤 2：尝试获取源码
* 尝试在本地使用 `git clone http://160.202.254.160:10418/.git/` 克隆，提示 `repository not found`，说明服务器拒绝了标准的 Git 协议请求。
* 访问 `.git/logs/HEAD` 文件，发现历史记录中多次出现 `commit: flag is here?` 的字样，说明 Flag 藏在某次提交记录中。
* 尝试手动下载 `objects` 目录下的压缩文件并用 `git cat-file -p` 解压，但由于文件路径结构不符合 Git 本地仓库规范，报错 `fatal: Not a valid object name`，手动解压极其繁琐。

### 步骤 3：改用 git-dumper 工具（最优解法）
放弃手动下载，改用专门针对此类漏洞的自动化工具 `git-dumper`，直接通过 HTTP 协议抓取并还原源码：

```bash
git-dumper http://160.202.254.160:10418/.git/ ./source
```

### 步骤 4：查看历史提交与文件差异
由于 `git-dumper` 已经帮我们在本地建立了一个完整的 Git 仓库（位于 `./source` 目录），我们可以直接用 Git 命令探查之前的提交记录。

进入源码目录，并查看 Git 的操作日志（reflog）：
```bash
cd source
git reflog
```

在输出中，我们会找到那几条熟悉的记录，其中有一条哈希值（如 `40c6d51`）对应的信息是 `commit: flag is here?`。

直接查看这个具体提交修改了哪些文件内容：
```bash
git show 40c6d51
```
**结果：** 通过查看 diff（差异对比），发现该次提交将 `flag.txt` 文件中的假 Flag `flag{nonono}` 修改成了真正的 Flag。

### 步骤 5：获取最终 Flag
在 `git show` 的输出中，提取出被修改后的真实内容：
`flag{git_is_good_distributed_version_control_system}`

将其提交到平台即可通关。

---

## 三、 关键知识点总结

1. **Git 源码泄露：** 网站部署时，若未删除 `.git` 目录且 Web 服务器配置不当，攻击者可直接下载该目录，还原出网站的全部源码和提交历史。
2. **`git-dumper` 工具：** 当 `git clone` 被服务器拒绝时，`git-dumper` 可以通过遍历 HTTP 请求，强行下载并重建本地仓库，是自动化利用此类漏洞的利器。
3. **`git reflog` 与 `git show`：** `reflog` 记录了仓库的所有操作历史（包括回退），`git show` 可以查看某次提交的具体代码变更。这是审计 Git 历史、寻找敏感信息（如被删除的 Flag）的核心命令。
