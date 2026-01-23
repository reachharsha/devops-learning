# 08 - Package Management

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What package managers are and why they matter
- Installing, updating, and removing software
- Using apt (Ubuntu/Debian) and yum/dnf (CentOS/RHEL)
- Troubleshooting common package issues
- Maintaining a healthy system

---

## 📦 What is Package Management?

**Real-World Analogy:**
```
Package Manager = App Store for Linux

iPhone:  App Store
Android: Google Play
Windows: Microsoft Store
Linux:   apt, yum, dnf (depending on distribution)

Difference: Linux package managers are:
- ✅ Free
- ✅ Terminal-based (though GUIs exist)
- ✅ More powerful
- ✅ Handle dependencies automatically
```

**What is a Package?**
```
Package = Software + Metadata

Contains:
- Compiled program files
- Configuration files
- Documentation
- List of dependencies
- Installation scripts

Example package: nginx
Includes:
- nginx binary
- Config files (/etc/nginx/)
- Man pages
- Systemd service file
- Dependencies (openssl, zlib, etc.)
```

---

## 🎯 Why Package Managers Matter for DevOps

```
Without Package Manager:
1. Google "install nginx linux"
2. Download from website
3. Find dependencies manually
4. Compile from source
5. Configure paths
6. Create startup scripts
7. Hope it works
8. No easy updates
9. No security patches

With Package Manager:
1. sudo apt install nginx
2. Done! ✅

Auto-handles:
✅ Downloads
✅ Dependencies
✅ Installation
✅ Configuration
✅ Updates
✅ Security patches
✅ Uninstallation
```

---

## 🐧 Different Package Managers

**By Linux Distribution:**

```
Distribution          Package Manager    Package Format
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ubuntu/Debian         apt / apt-get      .deb
Mint/Pop!_OS          apt / apt-get      .deb

CentOS/RHEL 7         yum                .rpm
CentOS/RHEL 8+        dnf                .rpm
Fedora                dnf                .rpm

Arch Linux            pacman             .pkg.tar.zst
Alpine Linux          apk                .apk

Note: We'll focus on apt and yum/dnf (most common in DevOps)
```

---

## 📦 APT (Ubuntu/Debian)

**APT = Advanced Package Tool**

### **Package Database**

```bash
# Update package database (do this first!)
sudo apt update

# What it does:
# - Contacts package repositories
# - Downloads latest package lists
# - Updates local cache

# Output:
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
Reading package lists... Done
Building dependency tree... Done

# Always run before installing!
```

### **Installing Packages**

```bash
# Install single package
sudo apt install nginx

# Install multiple packages
sudo apt install nginx mysql-server php

# Install specific version
sudo apt install nginx=1.18.0-0ubuntu1

# Install without confirmation prompt
sudo apt install -y nginx

# Reinstall package
sudo apt reinstall nginx

# Download but don't install
sudo apt download nginx
```

**Installation example:**
```bash
$ sudo apt install nginx

Reading package lists... Done
Building dependency tree... Done
The following additional packages will be installed:
  nginx-common nginx-core
Suggested packages:
  nginx-doc
The following NEW packages will be installed:
  nginx nginx-common nginx-core
0 upgraded, 3 newly installed, 0 to remove
Need to get 604 kB of archives.
After this operation, 2,118 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

### **Removing Packages**

```bash
# Remove package (keep config files)
sudo apt remove nginx

# Remove package + config files
sudo apt purge nginx

# Remove package + unused dependencies
sudo apt autoremove nginx

# Remove orphaned packages
sudo apt autoremove

# Clean up
sudo apt autoclean     # Remove old package files
sudo apt clean         # Remove all package files from cache
```

### **Updating System**

```bash
# Update package list
sudo apt update

# Upgrade all packages (safe)
sudo apt upgrade

# Upgrade + remove old packages if needed
sudo apt full-upgrade

# Upgrade specific package
sudo apt install --only-upgrade nginx

# Complete update workflow:
sudo apt update && sudo apt upgrade -y
```

**Maintenance schedule:**
```bash
# Daily/Weekly: Update package lists
sudo apt update

# Weekly/Monthly: Upgrade packages
sudo apt upgrade -y

# Monthly: Clean up
sudo apt autoremove -y
sudo apt autoclean
```

### **Searching Packages**

```bash
# Search for package
apt search nginx
apt search python

# Search in package names only
apt search --names-only docker

# Show package details
apt show nginx

# Check if package is installed
apt list --installed | grep nginx

# List all installed packages
apt list --installed

# List upgradable packages
apt list --upgradable

# Which package provides a file?
apt-file search /usr/bin/python3
dpkg -S /usr/bin/python3
```

### **Package Information**

```bash
# Detailed package info
apt show nginx

# Output:
Package: nginx
Version: 1.18.0-0ubuntu1
Priority: optional
Section: web
Maintainer: Ubuntu Developers
Installed-Size: 3,588 kB
Provides: httpd, httpd-cgi, nginx
Depends: libc6, libpcre3, libssl1.1, zlib1g
Suggests: nginx-doc
Homepage: https://nginx.org

# List files in package
dpkg -L nginx

# Which package owns this file?
dpkg -S /usr/sbin/nginx
```

---

## 🎯 YUM/DNF (CentOS/RHEL/Fedora)

**YUM = Yellowdog Updater Modified**  
**DNF = Dandified YUM (newer, faster)**

```bash
# CentOS/RHEL 7:    use yum
# CentOS/RHEL 8+:   use dnf
# Fedora:           use dnf

# Commands are nearly identical!
```

### **Installing Packages**

```bash
# Install package
sudo yum install nginx      # CentOS 7
sudo dnf install nginx      # CentOS 8+, Fedora

# Install multiple
sudo yum install nginx mysql-server php

# Install without confirmation
sudo yum install -y nginx

# Reinstall
sudo yum reinstall nginx

# Install local RPM
sudo yum localinstall package.rpm
```

### **Removing Packages**

```bash
# Remove package
sudo yum remove nginx

# Remove + dependencies
sudo yum autoremove nginx

# Clean up
sudo yum clean all
```

### **Updating System**

```bash
# Check for updates
sudo yum check-update

# Update specific package
sudo yum update nginx

# Update all packages
sudo yum update

# Update all (no prompts)
sudo yum update -y

# Complete update:
sudo yum update -y && sudo yum autoremove -y
```

### **Searching Packages**

```bash
# Search for package
yum search nginx

# Show package info
yum info nginx

# List installed packages
yum list installed

# List available packages
yum list available

# Which package provides a file?
yum provides /usr/sbin/nginx
yum whatprovides */nginx

# List package files
rpm -ql nginx
```

---

## 🔧 Installing Common Software

### **Web Servers**

```bash
# Nginx
sudo apt install nginx              # Ubuntu
sudo yum install nginx              # CentOS

# Apache
sudo apt install apache2            # Ubuntu
sudo yum install httpd              # CentOS
```

### **Databases**

```bash
# MySQL
sudo apt install mysql-server       # Ubuntu
sudo yum install mysql-server       # CentOS

# PostgreSQL
sudo apt install postgresql         # Ubuntu
sudo yum install postgresql-server  # CentOS

# MongoDB
sudo apt install mongodb            # Ubuntu (add repo first)
```

### **Programming Languages**

```bash
# Python
sudo apt install python3 python3-pip
sudo yum install python3 python3-pip

# Node.js
sudo apt install nodejs npm
sudo yum install nodejs npm

# Java
sudo apt install openjdk-11-jdk
sudo yum install java-11-openjdk

# Go
sudo apt install golang
sudo yum install golang
```

### **Development Tools**

```bash
# Git
sudo apt install git
sudo yum install git

# Docker (requires repository setup)
sudo apt install docker.io          # Ubuntu
sudo yum install docker              # CentOS

# Build tools
sudo apt install build-essential    # Ubuntu
sudo yum groupinstall "Development Tools"  # CentOS

# Curl & Wget
sudo apt install curl wget
sudo yum install curl wget
```

### **System Tools**

```bash
# htop (better top)
sudo apt install htop
sudo yum install htop

# ncdu (disk usage)
sudo apt install ncdu
sudo yum install ncdu

# tmux (terminal multiplexer)
sudo apt install tmux
sudo yum install tmux

# net-tools (netstat, ifconfig)
sudo apt install net-tools
sudo yum install net-tools
```

---

## 🚨 Troubleshooting

### **Problem: "Unable to locate package"**

```bash
# Solution 1: Update package list
sudo apt update

# Solution 2: Package might have different name
apt search package-name

# Solution 3: Might need to add repository
# Example for Docker:
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce
```

### **Problem: "dpkg was interrupted"**

```bash
# Fix interrupted installation
sudo dpkg --configure -a
sudo apt install -f
```

### **Problem: "Could not get lock"**

```bash
# Error:
E: Could not get lock /var/lib/dpkg/lock-frontend

# Cause: Another process is using apt

# Solution 1: Wait for it to finish
ps aux | grep apt

# Solution 2: Kill the process
sudo killall apt apt-get

# Solution 3: Remove locks (if system crashed)
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock*
sudo dpkg --configure -a
```

### **Problem: Broken dependencies**

```bash
# Fix broken dependencies
sudo apt install -f

# Or
sudo apt --fix-broken install

# Nuclear option (careful!)
sudo dpkg --configure -a
sudo apt clean
sudo apt update
sudo apt upgrade
```

### **Problem: Repository errors**

```bash
# Disable problematic repository
sudo nano /etc/apt/sources.list

# Comment out broken line with #

# Or remove from sources.list.d
ls /etc/apt/sources.list.d/
sudo rm /etc/apt/sources.list.d/broken-repo.list
```

---

## 🎯 Package Management Best Practices

### **1. Always Update Before Installing**

```bash
# Good ✅
sudo apt update
sudo apt install nginx

# Bad ❌
sudo apt install nginx    # Might install old version!
```

### **2. Regular System Updates**

```bash
# Weekly maintenance
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
sudo apt autoclean
```

### **3. Don't Mix Package Managers**

```bash
# Pick ONE method per package

# Bad ❌
sudo apt install python3      # System package manager
pip install requests          # Python package manager (global)

# Better ✅
Use virtual environments for Python:
python3 -m venv myenv
source myenv/bin/activate
pip install requests

# Or use system packages only:
sudo apt install python3-requests
```

### **4. Clean Up Regularly**

```bash
# Remove unused packages
sudo apt autoremove

# Clear package cache
sudo apt autoclean

# See cache size
du -sh /var/cache/apt/archives

# Nuclear clean (recovers most space)
sudo apt clean
```

### **5. Hold Critical Packages**

```bash
# Prevent package from updating
sudo apt-mark hold package-name

# Allow updates again
sudo apt-mark unhold package-name

# List held packages
apt-mark showhold
```

---

## 🎯 Quick Check: Do You Understand?

1. **Before installing any package, what should you run first?**
   <details>
   <summary>Answer</summary>
   sudo apt update (to refresh package lists)
   </details>

2. **What's the difference between `apt remove` and `apt purge`?**
   <details>
   <summary>Answer</summary>
   remove keeps config files, purge removes everything
   </details>

3. **How do you update all packages on Ubuntu?**
   <details>
   <summary>Answer</summary>
   sudo apt update && sudo apt upgrade
   </details>

4. **How do you search for a package?**
   <details>
   <summary>Answer</summary>
   apt search package-name
   </details>

5. **What command removes orphaned dependencies?**
   <details>
   <summary>Answer</summary>
   sudo apt autoremove
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Update package lists
sudo apt update

# 2. Search for htop
apt search htop

# 3. Show htop info
apt show htop

# 4. Install htop
sudo apt install -y htop

# 5. Verify installation
which htop
htop --version

# 6. Run htop (press q to quit)
htop

# 7. List installed packages
apt list --installed | grep htop

# 8. See what files were installed
dpkg -L htop

# 9. Remove htop
sudo apt remove htop

# 10. Verify removal
which htop    # Should return nothing

# 11. Clean up
sudo apt autoremove
sudo apt autoclean
```

---

## 📝 Key Takeaways

✅ **sudo apt update** - refresh package lists (do this first!)  
✅ **sudo apt install package** - install software  
✅ **sudo apt remove package** - uninstall (keeps config)  
✅ **sudo apt purge package** - uninstall (removes everything)  
✅ **sudo apt upgrade** - update all packages  
✅ **sudo apt autoremove** - clean up orphaned packages  
✅ **apt search** - find packages  
✅ **apt show** - package details  
✅ **Update regularly** - security and stability  
✅ **Clean up** - save disk space  

---

## 🚀 Next Steps

You can now manage software like a Linux pro!

**Next lesson:** [09-network-commands.md](09-network-commands.md) - Network commands and troubleshooting

---

## 💡 Pro Tips

**One-liner system maintenance:**
```bash
# Complete system update & cleanup
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && sudo apt autoclean

# Make it an alias (add to ~/.bashrc)
alias update='sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y'
```

**Check what will be upgraded:**
```bash
# See upgradable packages
apt list --upgradable

# See what upgrade will do (without doing it)
sudo apt upgrade --dry-run
```

**Install from official website vs package manager:**
```bash
# Package Manager ✅ (preferred)
sudo apt install docker.io

Pros:
✅ Automatic updates
✅ Dependency management
✅ Easy uninstall
✅ Security patches

# Manual Install ❌ (avoid if possible)
wget https://example.com/software.deb
sudo dpkg -i software.deb

Cons:
❌ Manual updates
❌ Dependency hell
❌ Harder to remove
❌ No automatic security patches
```

**Repository management:**
```bash
# List repositories
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Add repository (example: Docker)
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Remove repository
sudo add-apt-repository --remove "repo-url"
```

**Remember:**  
Package manager = your best friend in Linux! Master it! 🚀
