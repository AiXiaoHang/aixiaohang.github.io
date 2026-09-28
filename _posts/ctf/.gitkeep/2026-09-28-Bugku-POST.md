---
layout: post
title: "Bugku CTF - POST 题目 WriteUp"
date: 2026-09-28
categories: [CTF, WriteUp]
---

> **题目类型：** Web 基础 / PHP 变量传参  
> **使用工具：** 浏览器（Chrome/Edge）开发者工具 (F12)  
> **核心方法：** 利用浏览器控制台 (Console) 发送 POST 请求

---

## 一、 题目分析

访问题目地址后，查看网页源代码，发现是一段 PHP 代码：

```php
1  $what=$_POST['what'];
2  echo $what;
3  if($what=='flag')
4      echo 'flag{****}';
