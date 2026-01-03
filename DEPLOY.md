# 📦 部署指南

详细的部署步骤说明。

## 环境要求

| 项目 | 最低要求 | 推荐配置 |
|------|---------|---------|
| CPU | 1 核 | 2 核+ |
| 内存 | 2 GB | 4 GB+ |
| 硬盘 | 20 GB | 50 GB+ |
| 系统 | CentOS 7+ / Ubuntu 18+ | CentOS 8 / Ubuntu 22 |
| Docker | 20.0+ | 最新版 |
| Docker Compose | 2.0+ | 最新版 |

## 安装 Docker

### CentOS

```bash
# 安装依赖
yum install -y yum-utils

# 添加 Docker 仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker
yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动 Docker
systemctl start docker
systemctl enable docker

# 验证安装
docker --version
docker compose version
```

### Ubuntu

```bash
# 更新包索引
apt update

# 安装依赖
apt install -y apt-transport-https ca-certificates curl software-properties-common

# 添加 Docker GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 Docker 仓库
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 验证安装
docker --version
docker compose version
```

## 部署步骤

### 1. 上传部署包

```bash
# 方式1: scp 上传
scp order-system-offline-*.zip root@your-server:/opt/

# 方式2: wget 下载（如果有下载链接）
cd /opt
wget https://your-download-link/order-system-offline-*.zip
```

### 2. 解压部署包

```bash
cd /opt
unzip order-system-offline-*.zip
cd offline-package
```

### 3. 导入 Docker 镜像

```bash
chmod +x import.sh
./import.sh
```

导入过程可能需要几分钟，请耐心等待。

### 4. 配置环境变量

```bash
cp .env.example .env
vim .env
```

**必须修改的配置项：**

```env
# 应用域名（支付回调必须）
APP_DOMAIN=https://your-domain.com

# 数据库密码
MYSQL_ROOT_PASSWORD=你的强密码
MYSQL_PASSWORD=你的强密码

# Redis 密码
REDIS_PASSWORD=你的强密码

# JWT 密钥（至少32个字符）
JWT_SECRET=your-super-secret-key-at-least-32-chars
```

### 5. 启动服务

```bash
docker compose up -d
```

### 6. 检查服务状态

```bash
docker compose ps
```

所有服务应该显示 `running` 状态。

### 7. 配置防火墙

```bash
# CentOS
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload

# Ubuntu
ufw allow 80/tcp
ufw allow 443/tcp
```

### 8. 激活授权

1. 访问 `http://your-ip/admin`
2. 查看未授权页面显示的服务器 IP
3. 联系开发者获取授权文件（QQ: 1612423062）
4. 上传授权文件完成激活

## HTTPS 配置（可选）

### 使用 Let's Encrypt 免费证书

```bash
# 安装 certbot
apt install -y certbot

# 获取证书
certbot certonly --standalone -d your-domain.com

# 证书位置
# /etc/letsencrypt/live/your-domain.com/fullchain.pem
# /etc/letsencrypt/live/your-domain.com/privkey.pem
```

### 配置 Nginx SSL

编辑 `nginx/conf.d/default.conf`，启用 HTTPS 配置块。

## 常见问题

### Q: 容器启动失败怎么办？

```bash
# 查看详细日志
docker compose logs -f

# 查看特定服务日志
docker compose logs backend
```

### Q: 数据库连接失败？

1. 检查 MySQL 容器是否正常运行
2. 确认 `.env` 中的密码配置正确
3. 等待 MySQL 完全启动（首次启动需要 1-2 分钟）

### Q: 页面显示 502 错误？

1. 检查后端服务是否正常运行
2. 查看后端日志：`docker compose logs backend`
3. 确认端口没有被占用

### Q: 如何备份数据？

```bash
# 备份数据库
docker exec order-mysql mysqldump -uroot -p order_system > backup_$(date +%Y%m%d).sql

# 备份上传文件
tar -czvf uploads_$(date +%Y%m%d).tar.gz uploads/
```

### Q: 如何升级版本？

1. 下载新版本部署包
2. 停止服务：`docker compose down`
3. 备份数据
4. 导入新镜像：`./import.sh`
5. 启动服务：`docker compose up -d`

## 技术支持

如遇到问题，请联系：

- **QQ**：1612423062
- **GitHub Issues**：[提交问题](https://github.com/your-username/order-system/issues)

