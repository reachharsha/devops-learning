---
render_with_liquid: false
---
# 01 - Prerequisites & Installation

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand system requirements for Docker
- Install Docker on different operating systems
- Verify Docker installation
- Understand Docker architecture components
- Configure Docker for non-root access
- Troubleshoot common installation issues

---

## 📋 Prerequisites Check

Before installing Docker, ensure you have:

### **Knowledge Prerequisites:**
- [ ] Basic Linux commands (cd, ls, mkdir, rm)
- [ ] Understanding of what a process is
- [ ] Familiarity with command line/terminal
- [ ] Basic networking concepts (IP, ports)
- [ ] Text editor usage (nano or vim)

### **System Prerequisites:**

**Minimum Requirements:**
- **CPU:** 64-bit processor
- **RAM:** 4GB (8GB recommended)
- **Disk:** 20GB free space
- **OS:** Linux kernel 3.10+, Windows 10/11, macOS 10.15+
- **Internet:** For pulling images

**Check your system:**
```bash
# Linux - Check kernel version
uname -r  # Should be 3.10 or higher

# Check architecture
uname -m  # Should show x86_64 or aarch64

# Check available disk space
df -h /var/lib/docker

# Check memory
free -h
```

---

## 🐧 Installing Docker on Linux

### **Ubuntu / Debian**

**Step 1: Remove old versions (if any)**
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

**Step 2: Update package index**
```bash
sudo apt-get update
```

**Step 3: Install prerequisites**
```bash
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

**Step 4: Add Docker's official GPG key**
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**Step 5: Set up repository**
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Step 6: Install Docker Engine**
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin
```

**Step 7: Verify installation**
```bash
sudo docker run hello-world
```

---

### **CentOS / RHEL / Fedora**

**Step 1: Remove old versions**
```bash
sudo yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
```

**Step 2: Install yum-utils**
```bash
sudo yum install -y yum-utils
```

**Step 3: Add Docker repository**
```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

**Step 4: Install Docker**
```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin
```

**Step 5: Start Docker**
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

**Step 6: Verify**
```bash
sudo docker run hello-world
```

---

## 🪟 Installing Docker on Windows

### **Windows 10/11 Pro/Enterprise (with Hyper-V)**

**Step 1: Enable Hyper-V and WSL 2**
```powershell
# Run in PowerShell as Administrator
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Restart computer
```

**Step 2: Download Docker Desktop**
- Visit: https://www.docker.com/products/docker-desktop
- Download Docker Desktop for Windows
- Run the installer

**Step 3: Configure WSL 2**
```powershell
wsl --set-default-version 2
```

**Step 4: Start Docker Desktop**
- Launch Docker Desktop from Start menu
- Wait for Docker to start (watch system tray icon)

**Step 5: Verify in PowerShell**
```powershell
docker --version
docker run hello-world
```

---

### **Windows with WSL 2** (Recommended Method)

**Step 1: Install WSL 2**
```powershell
# PowerShell as Administrator
wsl --install
```

**Step 2: Install Ubuntu from Microsoft Store**
- Open Microsoft Store
- Search "Ubuntu 22.04 LTS"
- Install and launch

**Step 3: Install Docker in WSL Ubuntu**
```bash
# Inside WSL Ubuntu terminal
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Restart WSL
exit
# Then relaunch Ubuntu
```

---

## 🍎 Installing Docker on macOS

### **macOS (Intel or Apple Silicon)**

**Step 1: Download Docker Desktop**
- Visit: https://www.docker.com/products/docker-desktop
- Download for Mac (Intel or Apple Silicon)

**Step 2: Install**
- Open the .dmg file
- Drag Docker to Applications
- Launch Docker from Applications

**Step 3: Grant permissions**
- Docker will ask for system permissions
- Enter your password
- Allow Docker in System Preferences → Security

**Step 4: Verify**
```bash
docker --version
docker run hello-world
```

---

## 🔧 Post-Installation Setup

### **Run Docker without sudo (Linux)**

**Step 1: Create docker group (if not exists)**
```bash
sudo groupadd docker
```

**Step 2: Add your user to docker group**
```bash
sudo usermod -aG docker $USER
```

**Step 3: Activate changes**
```bash
# Log out and log back in, OR:
newgrp docker
```

**Step 4: Verify**
```bash
# Should work without sudo
docker run hello-world
```

---

### **Configure Docker to Start on Boot**

**Linux (systemd):**
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

**Check status:**
```bash
sudo systemctl status docker
```

---

## 🏗️ Understanding Docker Architecture

### **Docker Components:**

```
┌─────────────────────────────────────────────┐
│           Docker Architecture                │
├─────────────────────────────────────────────┤
│                                              │
│  ┌──────────────┐         ┌──────────────┐ │
│  │ Docker Client│◄────────┤   REST API   │ │
│  │   (docker)   │         └──────┬───────┘ │
│  └──────────────┘                │          │
│                                   │          │
│                         ┌─────────▼────────┐│
│                         │  Docker Daemon   ││
│                         │   (dockerd)      ││
│                         └─────────┬────────┘│
│                                   │          │
│                ┌──────────────────┼────────┐│
│                │                  │        ││
│         ┌──────▼─────┐    ┌──────▼─────┐  ││
│         │   Images   │    │ Containers │  ││
│         └────────────┘    └────────────┘  ││
│                                            ││
│         ┌────────────┐    ┌────────────┐  ││
│         │  Networks  │    │  Volumes   │  ││
│         └────────────┘    └────────────┘  ││
└─────────────────────────────────────────────┘
```

**Components Explained:**

1. **Docker Client (`docker`):**
   - CLI tool you interact with
   - Sends commands to Docker daemon
   - Can connect to remote daemons

2. **Docker Daemon (`dockerd`):**
   - Background service
   - Manages containers, images, networks, volumes
   - Listens for API requests

3. **Docker Engine:**
   - Combination of daemon + CLI
   - Core Docker technology

4. **containerd:**
   - Container runtime
   - Manages container lifecycle
   - Lower-level than Docker daemon

5. **runc:**
   - OCI-compliant runtime
   - Actually runs the containers
   - Lowest level

---

## ✅ Verifying Installation

### **Check Docker Version:**
```bash
docker --version
# Output: Docker version 24.0.7, build afdd53b

docker version
# Shows client and server versions

docker info
# Detailed system information
```

### **Check Components:**
```bash
# Check Docker daemon
sudo systemctl status docker  # Linux

# Check Docker Compose
docker compose version

# Check BuildKit
docker buildx version
```

### **Run Test Container:**
```bash
docker run hello-world
```

**Expected output:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from Docker Hub.
 3. The Docker daemon created a new container from that image.
 4. The Docker daemon streamed that output to the Docker client.
```

---

## 🎯 Quick Check: Installation Verification

Run these commands to verify everything works:

```bash
# 1. Check Docker is running
docker ps

# 2. Check available images
docker images

# 3. Run a simple container
docker run alpine echo "Docker is working!"

# 4. Check Docker info
docker info | grep "Server Version"

# 5. Verify Compose
docker compose version
```

All commands should work without errors!

---

## ⚠️ Troubleshooting Common Issues

### **Issue 1: Permission Denied**
```
Error: permission denied while trying to connect to Docker daemon
```

**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or:
newgrp docker
```

---

### **Issue 2: Docker Daemon Not Running**
```
Error: Cannot connect to the Docker daemon
```

**Solution:**
```bash
# Linux
sudo systemctl start docker
sudo systemctl status docker

# macOS/Windows
# Start Docker Desktop application
```

---

### **Issue 3: WSL 2 Installation Incomplete**
```
Error: WSL 2 installation is incomplete
```

**Solution:**
```powershell
# Update WSL
wsl --update

# Set default version
wsl --set-default-version 2
```

---

### **Issue 4: Insufficient Disk Space**
```
Error: No space left on device
```

**Solution:**
```bash
# Check disk usage
df -h /var/lib/docker

# Clean up unused resources
docker system prune -a

# Configure different storage location
# Edit /etc/docker/daemon.json
{
  "data-root": "/new/path/docker"
}
```

---

### **Issue 5: Port Already in Use**
```
Error: Bind for 0.0.0.0:80 failed: port is already allocated
```

**Solution:**
```bash
# Find process using the port
sudo lsof -i :80
sudo netstat -tulpn | grep :80

# Kill the process or use different port
docker run -p 8080:80 nginx
```

---

## 🛠️ Docker Configuration

### **Docker Daemon Configuration**

Create/edit `/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "default-address-pools": [
    {
      "base": "172.17.0.0/16",
      "size": 24
    }
  ]
}
```

**Restart Docker after changes:**
```bash
sudo systemctl restart docker
```

---

## 📝 Hands-on Exercise

**Complete these tasks:**

1. **Install Docker** on your system
2. **Verify** installation with `docker --version`
3. **Run** hello-world container
4. **Configure** non-root access (Linux)
5. **Run** these test containers:
   ```bash
   docker run alpine echo "Hello from Alpine!"
   docker run nginx:alpine  # Press Ctrl+C to stop
   docker run -d redis:alpine
   ```
6. **Check** running containers: `docker ps`
7. **Stop** all containers: `docker stop $(docker ps -q)`
8. **Clean up**: `docker system prune`

---

## 📝 Key Takeaways

✅ **Docker requires 64-bit OS with modern kernel**  
✅ **Docker Desktop for Windows/Mac, Docker Engine for Linux**  
✅ **Add user to docker group to avoid sudo**  
✅ **Docker uses client-server architecture**  
✅ **Verify with `docker run hello-world`**  
✅ **Configure daemon via /etc/docker/daemon.json**  
✅ **Check logs if issues occur**  

---

## 🔍 Installation Checklist

Before moving forward, ensure:

- [ ] Docker installed successfully
- [ ] `docker --version` works
- [ ] `docker ps` runs without errors
- [ ] hello-world container runs
- [ ] No sudo required (Linux)
- [ ] Docker starts on boot (configured)
- [ ] Enough disk space allocated
- [ ] Understanding of Docker architecture

---

## 🚀 Next Steps

**You're ready for containers!**

**Next lesson:** [02 - Docker Fundamentals](02-docker-fundamentals.md) - Learn what Docker is and why it matters

---

## 💡 Pro Tips

**Best Practices:**
- Use Docker Desktop on Windows/Mac (easier)
- Use Docker Engine on Linux servers (lightweight)
- Configure log rotation to save disk space
- Set resource limits in Docker Desktop
- Enable experimental features for new tools

**Quick Commands:**
```bash
# Check Docker status
docker info

# View Docker disk usage
docker system df

# Clean up everything
docker system prune -a --volumes

# View logs
journalctl -u docker  # Linux
```

**Installation complete - let's learn Docker!** 🐳
