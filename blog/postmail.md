---
title: "posteio mailserver guide"
description: "个人企业级邮件服务器搭建指南"
pubDate: " Mar 09 2025"
---

# Poste.io 个人邮件服务器搭建指南

## 1. 搭建开始前的准备

- 一个域名（例如 example.com，本文用 nodeseek.com 代替）
- 一台 VPS 或独立服务器（必须有独立 IPv4）
- 服务器为新安装的64位 Linux 系统，系统中不能安装 Nginx、宝塔、1Panel、Apache 等程序或面板
- Poste.io 运行需要约 1.8 GB 内存，建议 2 GB 及以上；关闭杀毒和反垃圾邮件功能时，1GB 服务器也可运行
- 邮件服务器必须开放 25 端口用于邮件收发。部分主机商会阻断此端口，需发工单解除
- 测试端口 25 是否被阻断：

  ```bash
  telnet smtp.aol.com 25
  ```

  若返回类似 `220 smtp.mail.yahoo.com ESMTP ready`，说明 25 端口正常。

---

## 2. 域名 DNS 的设置（第一部分）

DNS 配置对邮件服务器至关重要，错误配置会导致掉信、进垃圾箱等问题。

以下示例使用域名 `nodeseek.com`，服务器 IP 为 `12.34.56.78`，子域名设为 `mx.nodeseek.com`。

请在 DNS 服务商处添加以下记录（替换成自己的域名和 IP）：

| 记录类型 | 主机名/前缀  | 内容/值                         | 备注                   |
|----------|--------------|--------------------------------|------------------------|
| A        | mx           | 12.34.56.78                    | 邮件服务器地址         |
| MX       | @            | mx.nodeseek.com                | 优先级 10              |
| TXT      | @            | v=spf1 a mx -all               | SPF 记录               |
| TXT      | mx           | v=spf1 a -all                  | 子域 SPF（可选）       |
| TXT      | _dmarc       | v=DMARC1; p=reject; rua=mailto:postmaster@nodeseek.com; ruf=mailto:postmaster@nodeseek.com | DMARC 记录，邮箱可自定义 |

> **注意**：若使用 Cloudflare，请关闭上述记录的代理（关闭小云朵），否则可能收不到邮件。

> 若根域名没有指向任何地址，建议设置一条 A 记录指向邮件服务器 IP 或 127.0.0.1，部分收件服务器依赖此记录判断垃圾邮件。

---

## 3. Reverse DNS (rDNS) 的设置

Reverse DNS 将服务器 IP 反解析到子域名，防止邮件被拒收。

- 登录服务器主机商后台（一般在 Network/IP Address 设置里）
- 设置服务器 IP（如 `12.34.56.78`）的 rDNS 指向 `mx.nodeseek.com`
- 若找不到设置入口，请联系主机商客服或提交工单

> **rDNS 的值必须与 MX 记录的主机名一致**

---

## 4. 安装 Poste.io

1. 使用 root 用户登录服务器，设置主机名：

   ```bash
   hostnamectl set-hostname mx.nodeseek.com
   ```

2. 安装 Docker：

   ```bash
   sh <(curl -fsSL https://get.docker.com)
   ```

3. 创建数据目录：

   ```bash
   mkdir /opt/mail
   ```

4. 使用 Docker 运行 Poste.io 容器：

   ```bash
   docker run -d --restart=always      --net=host      -v /opt/mail:/data      --name "mailserver"      -h "mx.nodeseek.com"      -t analogic/poste.io
   ```

等待 5 分钟，安装完成。

---

## 5. 配置 Let’s Encrypt SSL 证书

- 访问 `https://mx.nodeseek.com/admin`（替换成你的域名）
- 按提示创建管理员账号（如 admin@nodeseek.com）
- 登录后，进入 **System settings > TLS Certificate**
- 点击 **issue free letsencrypt.org certificate**，勾选 **enabled**，点击保存
- 等待几分钟，证书部署完成

---

## 6. 域名 DNS 的设置（第二部分）

- 进入后台 **Virtual domains**，选择你的域名
- 在 DKIM 部分点击 **create a new key**
- 系统生成 DKIM 记录，添加到你的 DNS 作为 TXT 记录，例如：

  | 记录类型 | 主机名/前缀                     | 内容/值                             |
  |----------|--------------------------------|------------------------------------|
  | TXT      | s2025xxxxxxx._domainkey        | k=rsa; p=MIIBscxxxxxxxxx...       |

- 注意完整复制 DKIM 内容，否则验证失败导致邮件被拒

---

## 7. 添加用户与其他域名

- 在后台点击 **Email accounts**，点击 **Create a new email** 添加用户
- 点击 **Create a redirect (alias)** 添加别名或转发
- 添加其他域名需重复第6步生成 DKIM 并添加 DNS 记录

示例其他域名 DNS 记录：

| 记录类型 | 主机名/前缀 | 内容/值                           | 备注                    |
|----------|-------------|----------------------------------|-------------------------|
| MX       | @           | mx.nodeseek.com                  | 优先级 10，与主域一致    |
| TXT      | @           | v=spf1 a mx -all                 | SPF 记录                |
| TXT      | _dmarc      | v=DMARC1; p=reject; rua=mailto:postmaster@example.com; ruf=mailto:postmaster@example.com | DMARC 邮箱可自定义       |
| TXT      | s2025xxxxxxx._domainkey | k=rsa; p=Your_domain_key_xxxxxx | 根据系统生成 DKIM 记录   |

---

## 8. 测试邮件收发

- 打开 `https://mx.nodeseek.com/` 登录新建的邮箱
- 发送测试邮件给 `mail-tester.com`
- 理想情况下邮件评分满分 10 分
- 邮件内容过短可能扣分，但只要认证部分（SPF、DKIM、DMARC）全通过即可

---

# 祝贺！至此，您已成功搭建并配置了个人邮件服务器。

如遇问题欢迎留言讨论，我将尽力协助解答。
