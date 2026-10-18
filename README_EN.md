# Compose Tool 🐳

<p align="center">English ｜ <a href="README.md">简体中文</a></p>

**Docker Compose configuration converter**

Convert Docker Run commands and Docker Compose configs in one click, and enforce a standardized "one container, one directory" layout.

![License](https://img.shields.io/github/license/BeacherZ/compose-tool?style=flat-square)
![Docker](https://img.shields.io/badge/docker-ready-blue?style=flat-square&logo=docker)
![GitHub Stars](https://img.shields.io/github/stars/BeacherZ/compose-tool?style=flat-square)

---

## 🖼️ Preview

![Main UI](https://raw.githubusercontent.com/BeacherZ/compose-tool/main/main.png)

🔗 **Live demo**: [Compose Tool | Docker Compose Converter](http://8.209.214.239:6688)

> The web UI is currently in Chinese only; all conversion logic is language-independent.

---

## 🏗️ Core Philosophy: Blueprint and Materials in the Same Room

### The Problem

Once you run a few Docker projects, bind mounts end up scattered across arbitrary host directories, which quickly becomes a mess:

```
/root/data/          ← some projects live here
/home/user/docker/   ← some here
/var/lib/myapp/      ← some here
/tmp/test/           ← and some are just... wherever
```

### The Solution

This tool keeps every Docker project's "blueprint" (the `compose.yaml` file) and "materials" (the mapped data directory) together in one place:

```
/opt/docker/                        ← single root directory
├── vaultwarden/                    ← one subdirectory per container
│   ├── compose.yaml                ← blueprint (config file)
│   └── vw-data/                    ← materials (container data)
├── portainer/
│   ├── compose.yaml
│   └── portainer-data/
├── compose-tool/                   ← this project follows its own convention
│   └── compose.yaml
└── nextcloud/
    ├── compose.yaml
    └── data/
```

### Benefits

- **Trivial backups**: back up `/opt/docker` and you have backed up every project's config and data
- **Painless migration**: pack → upload to the new server → extract → start, fully restored
- **Clear layout**: `cd` into any subdirectory and run `docker compose up -d`

> The default path is `/opt/docker/{container_name}`. Override it with the `VOLUME_PREFIX` environment variable.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔐 **100% private** | Pure frontend. Everything is processed locally in your browser and never uploaded |
| 🎯 **Automatic path normalization** | Strips `$(pwd)`, `${HOME}`, `$PWD`, and all other shell variables |
| 🛡️ **System path allowlist** | 45+ system-level mounts (e.g. `/var/run/docker.sock`) are detected and preserved |
| 🔄 **Dual input modes** | Accepts both Docker Run commands and Docker Compose YAML |
| 📦 **One-click deploy command** | Outputs a complete deployment command that is ready to copy and run |
| 📱 **Responsive design** | Works on desktop and mobile |

---

## 🚀 Deployment

### Option 1: Docker Run (not recommended)

```bash
docker run -d \
  --name compose-tool \
  --restart unless-stopped \
  -p 6688:80 \
  -e VOLUME_PREFIX=/opt/docker/{container_name} \
  ghcr.io/beacherz/compose-tool:latest
```

### Option 2: Docker Compose (recommended — consistent with this tool's philosophy)

> 💡 Custom path: if you don't want the default `/opt/docker/` prefix, replace every `/opt/docker/` in the commands below (in `mkdir`, `cd`, and the environment variable) with your own path before running.

```bash
mkdir -p /opt/docker/compose-tool && cd /opt/docker/compose-tool && \
cat > compose.yaml << 'EOF'
services:
  compose-tool:
    image: ghcr.io/beacherz/compose-tool:latest
    container_name: compose-tool
    restart: unless-stopped
    ports:
      - "6688:80"  # change the left side to any port you like
    environment:
      - TZ=Asia/Shanghai  # timezone
      - VOLUME_PREFIX=/opt/docker/{container_name}  # default mount path; {container_name} is replaced with the container name
EOF
docker compose up -d
```

### Access: `http://<server-ip>:6688`

> **About HTTPS**: this is a static frontend page and all data stays in your browser, so plain HTTP is perfectly safe.
>
> If you need HTTPS, put a reverse proxy in front of it (Nginx Proxy Manager, Caddy, Traefik),
>
> and point your domain to `server-ip:6688`.

**Common management commands**

```bash
cd /opt/docker/compose-tool      # enter the project directory
docker compose logs -f            # follow logs
docker compose pull               # pull the latest image
docker compose up -d              # redeploy (run after updating)
docker compose down               # stop and remove the container
```

> 💡 This tool is a pure static site with no persistent data, so no volume mounts are required.

---

## 🎯 How It Works

1. **Paste** - copy a Docker Run command or Compose config from official docs or GitHub
2. **Convert** - the tool normalizes shell variables, rewrites paths, and adds standard settings
3. **Deploy** - the result is split into two parts, each with its own copy button:
   - **🚀 One-click deploy command**: directory creation, file generation, and container startup in one block
   - **📄 compose.yaml content**: plain YAML for manual editing

---

## 📖 Conversion Examples

### Example 1: Docker Run → Compose

**Input**

```bash
docker run -d --name vaultwarden \
  -p 8000:80 \
  -v $(pwd)/vw-data:/data \
  -e DOMAIN=https://vw.example.com \
  vaultwarden/server:latest
```

**Output (one-click deploy command)**

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

### Example 2: Upstream Compose → Standardized Paths

**Input**

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    volumes:
      - ./vw-data:/data
    ports:
      - 8000:80
```

**Output**

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

### Example 3: System Paths Are Preserved

**Input**

```bash
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/portainer-data:/data \
  --name portainer \
  portainer/portainer-ce
```

**Output**

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

> `/var/run/docker.sock` is recognized as a system path and left untouched.

---

## 💾 Backup and Migration

### Back up a single project

```bash
# Back up the vaultwarden project (config + data)
tar -czf /root/vaultwarden-backup.tar.gz /opt/docker/vaultwarden
```

**Archive location**: `/root/vaultwarden-backup.tar.gz`

---

### Back up all projects

```bash
# Back up every Docker project at once
tar -czf /root/docker-all-backup.tar.gz /opt/docker
```

**Archive location**: `/root/docker-all-backup.tar.gz`

---

### Download the archive to your computer

Use an FTP/SFTP client such as [FileZilla](https://filezilla-project.org/) or [WinSCP](https://winscp.net/):

1. Connect to the server (IP, username `root`, password)
2. Navigate to `/root/`
3. Download the `.tar.gz` archive

---

### Restore a single project

```bash
# Upload the archive to /root/ on the new server, then run
tar -xzf /root/vaultwarden-backup.tar.gz -C /

# Enter the directory and start the container
cd /opt/docker/vaultwarden
docker compose up -d
```

---

### Migrate all projects to a new server

**Step 1: Install Docker on the new server**

```bash
curl -fsSL https://get.docker.com | sh
```

**Step 2: Upload the archive**

Upload `docker-all-backup.tar.gz` to `/root/` on the new server with your FTP/SFTP client.

**Step 3: Extract and restore the directory structure**

```bash
# Run on the new server
tar -xzf /root/docker-all-backup.tar.gz -C /
```

The directory structure is restored automatically:

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

**Step 4: Start all containers**

```bash
for dir in /opt/docker/*/; do
  echo "Starting: $dir"
  (cd "$dir" && docker compose up -d)
done
```

---

## 🔧 Advanced Configuration

### Change the default mount path

Edit `compose.yaml` on the server and update the `VOLUME_PREFIX` environment variable:

```yaml
- VOLUME_PREFIX=/your/new/path/{container_name}
```

Recreate the container:

```bash
cd /opt/docker/compose-tool
docker compose up -d --force-recreate
```

### Custom port

Edit `compose.yaml`:

```yaml
ports:
  - "your-port:80"
```

### Update to the latest version

```bash
cd /opt/docker/compose-tool
docker compose pull
docker compose up -d
```

---

## 🛡️ System Path Allowlist

The following paths are detected and preserved as-is:

| Category | Example paths |
|---|---|
| **Docker** | `/var/run/docker.sock`, `/var/lib/docker`, `/usr/bin/docker` |
| **Kernel / devices** | `/proc`, `/sys`, `/dev`, `/boot`, `/lib/modules` |
| **System config** | `/etc/localtime`, `/etc/hosts`, `/etc/resolv.conf`, `/etc/timezone` |
| **SSL certificates** | `/etc/ssl`, `/etc/pki`, `/etc/ca-certificates` |
| **Users / groups** | `/etc/passwd`, `/etc/group`, `/etc/shadow`, `/etc/gshadow` |

The full allowlist covers 45+ paths, including all common system-level mounts.

---

## 📜 License

Released under the **MIT License**.

---

## 📧 Feedback

- **Issues**: [report a problem](https://github.com/BeacherZ/compose-tool/issues)
- **Discussions**: [join the discussion](https://github.com/BeacherZ/compose-tool/discussions)

---

<p align="center">
  <sub>Made with ❤️ for Docker enthusiasts</sub>
</p>
