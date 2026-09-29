---
layout: post
title: "Bugku CTF - POST 题目"
date: 2026-09-29
categories: [CTF, WriteUp]
---

> **题目类型：** Web 基础 / HTTP 请求方法  
> **使用工具：** 浏览器（Chrome/Edge）开发者工具 (F12)  
> **核心方法：** 利用浏览器控制台发送 POST 请求进行传参  

---

## 一、 题目分析

访问题目地址，查看网页源代码会发现一段 PHP 代码：

```php
$what=$_POST['what'];
echo $what;
if($what=='flag')
    echo 'flag{****}';
```
## 二、 解题步骤

### 步骤 1：打开开发者工具
在题目页面，按下键盘上的 **`F12`** 键，打开开发者工具。

### 步骤 2：在 Console 中构造 POST 请求
切换到 **Console（控制台）** 标签页。输入以下 JavaScript 代码（利用 fetch API 发送 POST 请求），并将 URL 替换为当前题目的实际地址：

```javascript
fetch('http://160.202.254.160:15762', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: 'what=flag'
})
.then(response => response.text())
.then(data => console.log(data));
```
### 步骤 3：获取 Flag
代码执行后，控制台会打印出服务器返回的响应内容。在输出结果中即可找到真实的 Flag，
最后，将得到的 flag 提交到平台即可通关。
