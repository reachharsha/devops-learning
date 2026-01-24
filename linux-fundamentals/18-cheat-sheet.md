---
render_with_liquid: false
---
# 18 - Linux Command Cheat Sheet

## 🎯 Quick Reference Guide

**Your go-to reference for all essential Linux commands!**

---

## 📁 Navigation & Files

### **Navigation**
```bash
pwd                    Print current directory
cd /path              Change directory
cd ~                  Go to home directory
cd ..                 Go up one directory
cd -                  Go to previous directory
ls                    List files
ls -la                List all files (detailed, including hidden)
tree                  Show directory tree
```

### **File Operations**
```bash
touch file.txt        Create empty file
mkdir dir             Create directory
mkdir -p a/b/c        Create nested directories
cp file1 file2        Copy file
cp -r dir1 dir2       Copy directory
mv old new            Move/rename
rm file               Delete file
rm -rf dir            Delete directory (DANGEROUS!)
find / -name file     Find file by name
locate file           Find file (uses database)
which command         Find command location
```

### **File Viewing**
```bash
cat file              Show entire file
less file             View file (paginated)
head file             First 10 lines
head -n 20 file       First 20 lines
tail file             Last 10 lines
tail -f file          Follow file (live updates)
tail -n 50 file       Last 50 lines
```

---

## 🔐 Permissions

### **View Permissions**
```bash
ls -l                 List with permissions
ls -ld dir/           Show directory permissions
stat file             Detailed file info
```

### **Change Permissions**
```bash
chmod 644 file        rw-r--r-- (files)
chmod 755 file        rwxr-xr-x (executables/dirs)
chmod 600 file        rw------- (private files)
chmod 700 dir         rwx------ (private dirs)

chmod +x file         Add execute permission
chmod -w file         Remove write permission
chmod u+x file        Add execute for user
chmod g-w file        Remove write for group
chmod o-r file        Remove read for others

chown user file       Change owner
chown user:group file Change owner and group
chown -R user:group dir  Recursive
```

### **Common Permission Patterns**
```bash
600    -rw-------   Private files (SSH keys)
644    -rw-r--r--   Public files (configs)
700    -rwx------   Private executables/dirs
755    -rwxr-xr-x   Public executables/dirs
777    -rwxrwxrwx   DANGEROUS! Never use!
```

---

## 🔍 Search & Text Processing

### **Search in Files**
```bash
grep pattern file             Search in file
grep -r pattern dir/          Search recursively
grep -i pattern file          Case-insensitive
grep -v pattern file          Invert match
grep -n pattern file          Show line numbers
grep -A 3 pattern file        3 lines after
grep -B 3 pattern file        3 lines before
grep -C 3 pattern file        3 lines before and after

find / -name "*.txt"          Find by name
find / -type f -name file     Find files
find / -type d -name dir      Find directories
find / -size +100M            Find large files
find / -mtime -7              Modified in last 7 days
find / -user john             Files owned by john
```

### **Text Processing**
```bash
sort file                  Sort lines
sort -r file               Reverse sort
sort -n file               Numeric sort
uniq file                  Remove duplicates
uniq -c file               Count duplicates
wc -l file                 Count lines
wc -w file                 Count words
cut -d',' -f1 file         Cut field 1 (CSV)
tr 'a-z' 'A-Z'             Translate (uppercase)
awk '{print $1}' file      Print first column
sed 's/old/new/g' file     Replace text
```

---

## ⚙️ Process Management

### **View Processes**
```bash
ps aux                        All processes
ps -ef                        All processes (alternative)
ps -u username                User's processes
ps aux | grep nginx           Find specific process
top                           Real-time processes
htop                          Better top
pstree                        Process tree
```

### **Manage Processes**
```bash
kill PID                      Graceful kill
kill -9 PID                   Force kill
kill -15 PID                  Graceful (SIGTERM)
killall process               Kill by name
pkill process                 Pattern kill
pkill -u username             Kill user's processes

bg                            Resume in background
fg                            Bring to foreground
jobs                          List background jobs
nohup command &               Run after logout

nice -n 10 command            Run with low priority
renice +10 PID                Change priority
```

### **System Monitoring**
```bash
top                           CPU/Memory usage
htop                          Better top
uptime                        System uptime & load
free -h                       Memory usage
df -h                         Disk usage
du -h dir                     Directory size
du -sh *                      Size of each item
iostat                        I/O statistics
vmstat                        Virtual memory stats
```

---

## 📦 Package Management

### **APT (Ubuntu/Debian)**
```bash
sudo apt update               Update package list
sudo apt upgrade              Upgrade packages
sudo apt install package      Install package
sudo apt remove package       Remove package
sudo apt purge package        Remove package + config
sudo apt autoremove           Remove unused packages
sudo apt search package       Search for package
apt show package              Package info
apt list --installed          List installed
apt list --upgradable         List upgradable
```

### **YUM/DNF (CentOS/RHEL)**
```bash
sudo yum update               Update packages
sudo yum install package      Install package
sudo yum remove package       Remove package
sudo yum search package       Search for package
yum info package              Package info
yum list installed            List installed
```

---

## 🌐 Network Commands

### **Network Configuration**
```bash
ip addr                       Show IP addresses
ip a                          Short form
ifconfig                      Network interfaces (old)
ip route                      Show routing table
ip link                       Show network interfaces
hostname                      Show hostname
hostname -I                   Show IP addresses
```

### **Connectivity**
```bash
ping host                     Test reachability
ping -c 4 host                4 pings only
traceroute host               Trace route
mtr host                      Better traceroute
curl https://site.com         HTTP request
wget https://site.com/file    Download file
```

### **Network Diagnostics**
```bash
netstat -tuln                 Show listening ports
ss -tuln                      Better netstat
ss -tulnp                     Show programs
lsof -i :80                   What's using port 80
nslookup domain               DNS lookup
dig domain                    Detailed DNS lookup
host domain                   Simple DNS lookup
```

### **File Transfer**
```bash
scp file user@host:/path      Copy to remote
scp user@host:/file .         Copy from remote
scp -r dir user@host:/path    Copy directory
rsync -avz source dest        Advanced sync
rsync -avzP local/ remote:/   Sync with progress
```

---

## 🔧 System Services

### **SystemD (Modern Linux)**
```bash
sudo systemctl start service        Start service
sudo systemctl stop service         Stop service
sudo systemctl restart service      Restart service
sudo systemctl reload service       Reload config
sudo systemctl status service       Check status
sudo systemctl enable service       Start at boot
sudo systemctl disable service      Don't start at boot
sudo systemctl is-active service    Check if running
sudo systemctl is-enabled service   Check if enabled

systemctl list-units --type=service           List services
systemctl list-units --state=running          Running services
systemctl list-units --state=failed           Failed services
```

### **Logs**
```bash
journalctl                          All logs
journalctl -f                       Follow logs
journalctl -u service               Service logs
journalctl -u service -f            Follow service logs
journalctl -n 50                    Last 50 lines
journalctl --since "1 hour ago"     Time-based
journalctl -p err                   Errors only
journalctl -b                       Current boot
journalctl -k                       Kernel messages

tail -f /var/log/syslog             Follow syslog
grep ERROR /var/log/app.log         Search logs
```

---

## 🔐 User & Security

### **User Management**
```bash
whoami                           Current user
id                               User ID info
who                              Logged in users
w                                Detailed who
last                             Login history

sudo useradd -m username         Create user
sudo passwd username             Set password
sudo usermod -aG group user      Add to group
sudo userdel -r username         Delete user
groups username                  Show groups
```

### **sudo**
```bash
sudo command                     Run as root
sudo -i                          Root shell
sudo -u user command             Run as user
sudo !!                          Previous command with sudo
visudo                           Edit sudo config
```

### **SSH**
```bash
ssh user@host                    Connect to server
ssh -p 2222 user@host            Custom port
ssh-keygen                       Generate key pair
ssh-copy-id user@host            Copy key to server
scp file user@host:/path         Copy file over SSH
```

---

## 📂 Archives & Compression

### **TAR**
```bash
tar -czvf archive.tar.gz dir/    Create compressed archive
tar -xzvf archive.tar.gz         Extract archive
tar -tzvf archive.tar.gz         List contents
tar -xzvf file.tar.gz -C /path   Extract to path

# Flags:
c = create, x = extract, t = list
z = gzip, j = bzip2, J = xz
v = verbose, f = file
```

### **Compression**
```bash
gzip file                        Compress (creates file.gz)
gunzip file.gz                   Decompress
bzip2 file                       Better compression
bunzip2 file.bz2                 Decompress
xz file                          Best compression
unxz file.xz                     Decompress

zip -r archive.zip dir/          Create zip
unzip archive.zip                Extract zip
unzip -l archive.zip             List contents
```

---

## 🔀 I/O Redirection & Pipes

### **Redirection**
```bash
command > file                   Redirect output (overwrite)
command >> file                  Append output
command 2> file                  Redirect errors
command &> file                  Redirect all output
command < file                   Input from file
command > /dev/null              Discard output
command 2>&1                     Redirect errors to output
```

### **Pipes**
```bash
command1 | command2              Pipe output
command | tee file               Output to file AND screen
command | grep pattern           Filter output
command | sort                   Sort output
command | uniq                   Remove duplicates
command | wc -l                  Count lines
command | head -10               First 10 lines
command | tail -10               Last 10 lines
```

### **Practical Combinations**
```bash
cat file | grep error | wc -l                Count errors
ps aux | grep nginx | awk '{print $2}'       Get PIDs
ls -la | sort -k5 -rh | head -10              10 largest files
history | grep ssh                            Search history
du -sh * | sort -rh                           Directories by size
```

---

## 🌍 Environment Variables

### **View Variables**
```bash
env                              All variables
printenv                         All variables
echo $VAR                        Specific variable
echo $PATH                       PATH variable
echo $HOME                       Home directory
```

### **Set Variables**
```bash
export VAR="value"               Set variable
export PATH=$PATH:/new/path      Add to PATH
unset VAR                        Remove variable
```

### **Common Variables**
```bash
$HOME                            Home directory
$USER                            Current user
$PATH                            Command search path
$SHELL                           Current shell
$PWD                             Current directory
```

### **Shell Configuration**
```bash
nano ~/.bashrc                   Edit bash config
source ~/.bashrc                 Reload config
nano ~/.bash_profile             Login shell config
```

---

## 📝 Text Editors

### **Nano (Beginner-Friendly)**
```bash
nano file                        Edit file
Ctrl+O                           Save
Ctrl+X                           Exit
Ctrl+W                           Search
Ctrl+K                           Cut line
Ctrl+U                           Paste
Ctrl+_                           Go to line
```

### **Vim (Power User)**
```bash
vim file                         Edit file

# Modes:
i                                Insert mode
Esc                              Normal mode
:                                Command mode

# Commands (Normal mode):
dd                               Delete line
yy                               Copy line
p                                Paste
u                                Undo
/pattern                         Search

# Command mode:
:w                               Save
:q                               Quit
:wq                              Save and quit
:q!                              Quit without saving
```

---

## 🔥 Firewall

### **UFW (Ubuntu)**
```bash
sudo ufw enable                  Enable firewall
sudo ufw disable                 Disable firewall
sudo ufw status                  Check status
sudo ufw allow 22                Allow port
sudo ufw deny 23                 Deny port
sudo ufw allow ssh               Allow service
sudo ufw delete allow 80         Remove rule
sudo ufw reset                   Reset firewall
```

### **firewalld (CentOS)**
```bash
sudo systemctl start firewalld         Start firewall
sudo firewall-cmd --state              Check status
sudo firewall-cmd --add-port=80/tcp    Add port
sudo firewall-cmd --add-service=http   Add service
sudo firewall-cmd --reload             Reload
```

---

## 🛠️ Shell Scripting Basics

### **Script Structure**
```bash
#!/bin/bash                      Shebang
# Comment                        Comment

VAR="value"                      Variable
echo $VAR                        Use variable
```

### **Conditionals**
```bash
if [ condition ]; then
    commands
fi

if [ $VAR -eq 10 ]; then        Numeric equal
if [ "$VAR" = "text" ]; then    String equal
if [ -f file ]; then            File exists
if [ -d dir ]; then             Directory exists
```

### **Loops**
```bash
for i in 1 2 3; do
    echo $i
done

while [ condition ]; do
    commands
done
```

---

## 🚨 Troubleshooting

### **Quick Diagnostics**
```bash
df -h                            Disk space
free -h                          Memory
top                              CPU/Memory usage
uptime                           Load average
systemctl status service         Service status
journalctl -xe                   Recent errors
ping 8.8.8.8                     Internet connectivity
sudo ss -tlnp                    Listening ports
ps aux | head                    Running processes
```

### **Common Fixes**
```bash
sudo systemctl restart service   Restart service
sudo systemctl daemon-reload     Reload systemd
sudo apt clean                   Clear package cache
sudo reboot                      Restart system
```

---

## ⌨️ Keyboard Shortcuts

### **Terminal**
```bash
Ctrl+C                           Kill current process
Ctrl+Z                           Suspend process
Ctrl+D                           Logout/exit
Ctrl+L                           Clear screen
Ctrl+A                           Start of line
Ctrl+E                           End of line
Ctrl+K                           Cut to end of line
Ctrl+U                           Cut to start of line
Ctrl+R                           Search history
Ctrl+W                           Delete word
Tab                              Auto-complete
!!                               Previous command
!$                               Last argument
```

---

## 💡 One-Liners Collection

### **System Maintenance**
```bash
# Full system update
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh

# Disk usage by directory
du -h --max-depth=1 / 2>/dev/null | sort -rh | head -20

# Memory usage by process
ps aux --sort=-%mem | head -10

# CPU usage by process
ps aux --sort=-%cpu | head -10

# Check all service status
systemctl list-units --type=service --state=running
```

### **Network**
```bash
# Show all listening ports
sudo ss -tlnp | column -t

# What's using port 80?
sudo lsof -i :80

# Network connectivity test
ping -c 1 8.8.8.8 && echo "OK" || echo "FAIL"

# Current connections
netstat -an | grep ESTABLISHED | wc -l
```

### **Log Analysis**
```bash
# Count errors in log
grep -c ERROR /var/log/app.log

# Show unique errors
cat /var/log/app.log | grep ERROR | sort | uniq -c | sort -rn

# Top 10 IPs in access log
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
```

---

## 📚 Getting Help

```bash
man command                      Manual page
command --help                   Help option
info command                     Info page
whatis command                   Brief description
apropos keyword                  Search man pages
tldr command                     Simplified examples
```

---

## 🎓 Learning Resources

```bash
# Practice commands safely
mkdir ~/practice
cd ~/practice

# Read man pages
man bash
man ls
man chmod

# Try vimtutor
vimtutor

# Explore system
ls -la /etc
cat /etc/os-release
uname -a
```

---

## 🎯 Final Tips

✅ **Always use `sudo` carefully** - it has full power  
✅ **Backup before modifying** - `cp file file.bak`  
✅ **Test commands in test environment** first  
✅ **Read error messages** - they usually tell you the problem  
✅ **Google is your friend** - search error messages  
✅ **Man pages are helpful** - `man command`  
✅ **One change at a time** - easier to troubleshoot  
✅ **Document what works** - save useful commands  
✅ **Practice, practice, practice!**  

---

## 🎉 Congratulations!

You've completed the Linux Fundamentals guide!

**You now have the skills to:**
✅ Navigate and manage files  
✅ Control permissions and security  
✅ Manage processes and services  
✅ Configure networks  
✅ Write shell scripts  
✅ Troubleshoot issues  
✅ Work efficiently in Linux  

**Keep practicing and exploring!** 🚀

---

**Back to:** [00-START-HERE.md](00-START-HERE.md) - Main guide

---

💡 **Pro Tip:** Bookmark this page! You'll reference it constantly as you work with Linux. Every expert still looks up commands - that's completely normal!
