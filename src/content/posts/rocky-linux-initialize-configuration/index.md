---
title: 'Rocky Linux 服务器初始化配置指南（适用于新手）'
published: 2025-12-4
draft: false
tags: ['linux']
toc: true
coverImage:
  src: './cover.png'
  alt: 'linux 显示系统中各个进程的占用情况。'
---

# Rocky Linux 服务器初始化配置指南（适用于新手）

🎉 **Rocky Linux**！作为一个企业级、极其稳定的 RHEL（Red Hat Enterprise Linux）下游发行版，它非常适合作为服务器操作系统。

拿到一台全新的 Rocky Linux 服务器后，为了保证安全性、稳定性和易用性，以下是你必须立即执行的“初始化清单”。

---

## ✅ 第一步：系统更新

确保所有软件包和安全补丁都是最新的：

```bash
sudo dnf update -y
```

> 💡 如果内核有更新，建议在所有设置完成后执行一次重启：
>
> ```bash
> sudo reboot
> ```

---

## 👤 第二步：创建普通用户并配置 Sudo 权限

永远不要直接使用 `root` 用户进行日常操作！

```bash
# 创建新用户（将 username 替换为你的用户名）
sudo adduser username

# 设置密码
sudo passwd username

# 添加到管理员组
sudo usermod -aG wheel username
```

---

## 🔐 第三步：SSH 安全加固

### A. 配置 SSH 密钥登录（强烈推荐）

在本地电脑生成密钥并上传公钥：

```bash
ssh-keygen
ssh-copy-id username@your_server_ip
```

### B. 修改 SSH 配置文件

编辑 `/etc/ssh/sshd_config`：

```bash
sudo vi /etc/ssh/sshd_config
```

确保以下配置项：

```ini
PermitRootLogin no
PermitEmptyPasswords no
Port 2222                # 可选，修改默认端口
PasswordAuthentication no  # 可选，仅允许密钥登录
```

### C. 重启 SSH 服务

```bash
sudo systemctl restart sshd
```

> ⚠️ 在退出当前终端前，务必新开一个窗口测试新用户能否成功登录！

---

## 🔥 第四步：配置防火墙（Firewalld）

### 安装并启动 firewalld

```bash
sudo dnf install firewalld -y
sudo systemctl enable --now firewalld
```

### 放行必要端口

```bash
# 放行 SSH（默认端口或自定义端口）
sudo firewall-cmd --permanent --add-service=ssh
# 或者
sudo firewall-cmd --permanent --add-port=2222/tcp

# 放行 Web 服务端口
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# 应用更改
sudo firewall-cmd --reload
```

> 💡 云服务器用户请同时配置云平台的“安全组”规则！

---

## 🕒 第五步：设置时区和主机名

```bash
# 查看当前时间设置
timedatectl

# 设置时区为中国标准时间
sudo timedatectl set-timezone Asia/Shanghai

# 设置主机名
sudo hostnamectl set-hostname web-server-01
```

---

## 🧰 第六步：安装常用工具和 EPEL 源

```bash
# 安装 EPEL 源
sudo dnf install epel-release -y

# 安装常用工具
sudo dnf install vim wget curl git htop net-tools unzip -y
```

---

## 🛡️ 第七步：安装并配置 Fail2Ban（防爆破）

```bash
# 安装 Fail2Ban
sudo dnf install fail2ban -y

# 复制配置文件
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# 编辑 jail.local，启用 sshd 部分并设置正确端口
sudo vi /etc/fail2ban/jail.local

# 启动服务
sudo systemctl enable --now fail2ban
```

---

## 💾 第八步：配置 Swap 分区（适用于内存较小的服务器）

```bash
# 检查是否已有 Swap
free -h

# 创建 2G Swap 文件
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 设置开机自动挂载
echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab

# 优化 swappiness
echo 'vm.swappiness = 10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## ✅ 初始化检查清单

| 步骤 | 任务                                   | 状态 |
| ---- | -------------------------------------- | ---- |
| 1    | 系统全量更新                           | ✅   |
| 2    | 创建 Sudo 用户                         | ✅   |
| 3    | SSH 禁止 Root 登录 / 改端口 / 密钥登录 | ✅   |
| 4    | 配置 Firewalld 防火墙                  | ✅   |
| 5    | 设置时区 (Asia/Shanghai)               | ✅   |
| 6    | 安装 EPEL 源和基础工具                 | ✅   |
| 7    | 安装 Fail2Ban 防爆破                   | ✅   |
| 8    | 配置 Swap（如需）                      | ✅   |

---

## 🚀 下一步

你的服务器基础环境已经非常安全且稳固了。接下来可以选择：

- 🅰️ 安装网站/博客（Nginx/Apache + PHP/Python）
- 🅱️ 部署容器化应用（Docker/Podman）
- 🅲️ 搭建数据库服务（MySQL/PostgreSQL）

---
