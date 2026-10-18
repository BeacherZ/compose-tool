# Compose Tool 🐳

**Docker Compose 配置转换工具**

一键转换 Docker Run 命令和 Docker Compose 配置，实现"一容器一目录"的标准化管理。

![License](https://img.shields.io/github/license/BeacherZ/compose-tool?style=flat-square)
![Docker](https://img.shields.io/badge/docker-ready-blue?style=flat-square&logo=docker)
![GitHub Stars](https://img.shields.io/github/stars/BeacherZ/compose-tool?style=flat-square)

---

## 🖼️ 界面预览

![主界面](https://raw.githubusercontent.com/BeacherZ/compose-tool/main/main.png)

🔗 **在线演示**：[Compose Tool | Docker 配置转换工具](http://8.209.214.239:6688)

---

## 🏗️ 核心哲学：图纸与材料同屋存放

### 问题
装了几个 Docker 项目后，各种映射卷零零散散躺在宿主机的各个目录里，极其混乱：
```
/root/data/          ← 有些项目装这里
/home/user/docker/   ← 有些装这里
/var/lib/myapp/      ← 有些装这里
/tmp/test/           ← 还有些随手放的
```

### 解决方案
本工具的目标是把所有 Docker 项目的"图纸"（compose.yaml 配置文件）和"材料"（映射的数据目录）统一存放：
```
/opt/docker/                        ← 统一大目录
├── vaultwarden/                    ← 每个容器一个子目录
│   ├── compose.yaml                ← 图纸（配置文件）
│   └── vw-data/                    ← 材料（容器数据）
├── portainer/
│   ├── compose.yaml
│   └── portainer-data/
├── compose-tool/                   ← 本项目自身也遵循此规范
│   └── compose.yaml
└── nextcloud/
    ├── compose.yaml
    └── data/
```

### 好处
- **备份极简**：备份整个 `/opt/docker` 目录即可备份所有项目的配置和数据
- **迁移无痛**：打包 → 上传到新服务器 → 解压 → 一键启动，完整恢复
- **管理清晰**：进入任意子目录执行 `docker compose up -d` 即可启动对应项目

> 本工具默认路径为 `/opt/docker/{container_name}`，可通过 `VOLUME_PREFIX` 环境变量自定义。

---

## ✨ 核心特性

| 特性 | 说明 |
|---|---|
| 🔐 **100% 隐私安全** | 纯前端实现，所有数据仅在浏览器本地处理，绝不上传 |
| 🎯 **自动路径规范化** | 自动清理 `$(pwd)`、`${HOME}`、`$PWD` 等所有 Shell 变量 |
| 🛡️ **系统路径白名单** | 45+ 系统级挂载（如 `/var/run/docker.sock`）自动识别保留 |
| 🔄 **双模式支持** | 同时支持 Docker Run 命令和 Docker Compose YAML 配置转换 |
| 📦 **一键部署命令** | 转换结果自动生成完整部署命令，复制粘贴即可执行 |
| 📱 **响应式设计** | 完美适配桌面端和移动端 |

---

## 🚀 快速部署

### 方式一：Docker Run（不推荐）

```bash
docker run -d \
  --name compose-tool \
  --restart unless-stopped \
  -p 6688:80 \
  -e VOLUME_PREFIX=/opt/docker/{container_name} \
  ghcr.io/beacherz/compose-tool:latest
```

### 方式二：Docker Compose（推荐，符合本工具理念）
> 💡 自定义路径说明：
> 如果不想使用默认的 /opt/docker/ 路径，请在执行前将下面命令中所有的 /opt/docker/ 整体替换为你规划的路径（包括 mkdir、cd 以及环境变量中的路径）。
```bash
mkdir -p /opt/docker/compose-tool && cd /opt/docker/compose-tool && \
cat > compose.yaml << 'EOF'
services:
  compose-tool:
    image: ghcr.io/beacherz/compose-tool:latest
    container_name: compose-tool
    restart: unless-stopped
    ports:
      - "6688:80"  # 左边可改为你想要的端口
    environment:
      - TZ=Asia/Shanghai  # 时区
      - VOLUME_PREFIX=/opt/docker/{container_name}  # 默认卷映射路径，{container_name} 会自动替换为容器名
EOF
docker compose up -d
```

### 访问：`http://服务器IP:6688`

> **关于 HTTPS**：本工具为纯前端静态页面，所有数据在浏览器本地处理，HTTP 访问完全安全。
> 
> 如需HTTPS，可在前面套一层反向代理（如 Nginx Proxy Manager / Caddy / Traefik），
> 
> 将域名指向 `服务器IP:6688` 即可。

**常用管理命令**

```bash
cd /opt/docker/compose-tool      # 进入项目目录
docker compose logs -f            # 查看运行日志
docker compose pull               # 拉取最新镜像
docker compose up -d              # 重新部署（更新后执行）
docker compose down               # 停止并删除容器
```

> 💡 本工具是纯静态网页，无数据持久化需求，无需挂载卷。

---

## 🎯 使用流程

1. **粘贴命令** - 从官方文档或 GitHub 复制 Docker Run 命令或 Compose 配置
2. **点击转换** - 工具自动清理 Shell 变量、转换路径、添加标准配置
3. **一键部署** - 转换结果分为两部分，各有独立复制按钮：
   - **🚀 一键部署命令**：包含目录创建、文件生成、容器启动的完整命令
   - **📄 compose.yaml 内容**：纯 YAML 配置，适合手动编辑

---

## 📖 转换示例

### 示例 1：Docker Run → Compose

**输入**
```bash
docker run -d --name vaultwarden \
  -p 8000:80 \
  -v $(pwd)/vw-data:/data \
  -e DOMAIN=https://vw.example.com \
  vaultwarden/server:latest
```

**输出（一键部署命令）**
```bash
mkdir -p /opt/docker/vaultwarden && cd /opt/docker/vaultwarden && \
cat > compose.yaml << 'EOF'
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    ports:
      - "8000:80"
    volumes:
      - "/opt/docker/vaultwarden/vw-data:/data"
    environment:
      - "DOMAIN=https://vw.example.com"
    restart: unless-stopped
EOF
docker compose up -d
```

---

### 示例 2：官方 Compose → 标准化路径

**输入**
```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    volumes:
      - ./vw-data:/data
    ports:
      - 8000:80
```

**输出**
```bash
mkdir -p /opt/docker/vaultwarden && cd /opt/docker/vaultwarden && \
cat > compose.yaml << 'EOF'
services:
  vaultwarden:
    image: vaultwarden/server:latest
    volumes:
      - "/opt/docker/vaultwarden/vw-data:/data"
    ports:
      - "8000:80"
    restart: unless-stopped
EOF
docker compose up -d
```

---

### 示例 3：系统路径自动保留

**输入**
```bash
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/portainer-data:/data \
  --name portainer \
  portainer/portainer-ce
```

**输出**
```bash
mkdir -p /opt/docker/portainer && cd /opt/docker/portainer && \
cat > compose.yaml << 'EOF'
services:
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "/opt/docker/portainer/portainer-data:/data"
    restart: unless-stopped
EOF
docker compose up -d
```

> `/var/run/docker.sock` 识别为系统路径，自动保留不转换。

---

## 💾 备份与迁移

### 备份单个项目

```bash
# 备份 vaultwarden 项目（配置 + 数据）
tar -czf /root/vaultwarden-backup.tar.gz /opt/docker/vaultwarden
```

**压缩包位置**：`/root/vaultwarden-backup.tar.gz`

---

### 备份所有项目

```bash
# 一次性备份所有 Docker 项目
tar -czf /root/docker-all-backup.tar.gz /opt/docker
```

**压缩包位置**：`/root/docker-all-backup.tar.gz`

---

### 下载备份到本地电脑

使用 [FileZilla](https://filezilla-project.org/) 或 [WinSCP](https://winscp.net/) 等 FTP 工具：

1. 连接到服务器（填写 IP、用户名 `root`、密码）
2. 进入 `/root/` 目录
3. 下载 `.tar.gz` 压缩包到本地电脑

---

### 恢复单个项目

```bash
# 在新服务器上传压缩包到`/root/` 目录后执行
tar -xzf /root/vaultwarden-backup.tar.gz -C /

# 进入目录启动容器
cd /opt/docker/vaultwarden
docker compose up -d
```

---

### 迁移所有项目到新服务器

**第 1 步：新服务器安装 Docker**

```bash
curl -fsSL https://get.docker.com | sh
```

**第 2 步：上传压缩包到新服务器**

使用 FTP 工具将本地的 `docker-all-backup.tar.gz` 上传到新服务器的 `/root/` 目录。

**第 3 步：解压并还原目录结构**

```bash
# 在新服务器执行
tar -xzf /root/docker-all-backup.tar.gz -C /
```

解压后目录结构自动恢复：

```
/opt/docker/
├── vaultwarden/
│   ├── compose.yaml
│   └── vw-data/
├── portainer/
│   ├── compose.yaml
│   └── portainer-data/
├── compose-tool/
│   └── compose.yaml
└── ...
```

**第 4 步：一键启动所有容器**

```bash
for dir in /opt/docker/*/; do
  echo "正在启动: $dir"
  (cd "$dir" && docker compose up -d)
done
```

---

## 🔧 高级配置

### 修改默认卷映射路径

编辑服务器上的 `compose.yaml`，修改 `VOLUME_PREFIX` 环境变量：

```yaml
- VOLUME_PREFIX=/你的新路径/{container_name}
```

重启容器：

```bash
cd /opt/docker/compose-tool
docker compose up -d --force-recreate
```

### 自定义端口

编辑 `compose.yaml`：

```yaml
ports:
  - "你的端口:80"
```

### 更新到最新版本

```bash
cd /opt/docker/compose-tool
docker compose pull
docker compose up -d
```

---

## 🛡️ 系统路径白名单

以下路径自动识别保留，不做转换：

| 分类 | 路径示例 |
|---|---|
| **Docker** | `/var/run/docker.sock`、`/var/lib/docker`、`/usr/bin/docker` |
| **内核/设备** | `/proc`、`/sys`、`/dev`、`/boot`、`/lib/modules` |
| **系统配置** | `/etc/localtime`、`/etc/hosts`、`/etc/resolv.conf`、`/etc/timezone` |
| **SSL 证书** | `/etc/ssl`、`/etc/pki`、`/etc/ca-certificates` |
| **用户/组** | `/etc/passwd`、`/etc/group`、`/etc/shadow`、`/etc/gshadow` |

完整白名单包含 45+ 路径，覆盖所有常见系统级挂载。

---

## 📜 开源协议

本项目采用 **MIT License** 协议开源。

---

## 📧 问题反馈

- **Issues**：[提交问题](https://github.com/BeacherZ/compose-tool/issues)
- **Discussions**：[参与讨论](https://github.com/BeacherZ/compose-tool/discussions)

---

<p align="center">
  <sub>Made with ❤️ for Docker enthusiasts</sub>
</p>
