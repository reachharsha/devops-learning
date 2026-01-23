# 11 - Environment Variables

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What environment variables are
- Viewing and setting variables
- PATH and how it works
- Shell configuration files (.bashrc, .profile)
- Creating persistent custom variables

---

## 🌍 What are Environment Variables?

**Real-World Analogy:**
```
Environment Variables = Global Settings

Like your phone settings:
- Language: English
- Timezone: PST
- Theme: Dark Mode

Every app can access these settings
You set them once, all apps use them
```

**Technical Definition:**
```
Environment Variable = Named value accessible to processes

Example:
HOME=/home/john
PATH=/usr/bin:/bin
LANG=en_US.UTF-8

Programs can read these values
Influences program behavior
Persists across commands
```

---

## 👀 Viewing Environment Variables

```bash
# Show all environment variables
env

# Or
printenv

# Show specific variable
echo $HOME
echo $PATH
echo $USER

# Show variable with printenv
printenv HOME
printenv PATH

# List all variables (including shell variables)
set

# Show variable with details
declare -p HOME
```

---

## 📋 Common Environment Variables

```bash
# User information
$USER           Current username
$HOME           User's home directory
$SHELL          Current shell (/bin/bash, /bin/zsh)
$HOSTNAME       Computer name

# System paths
$PATH           Where to find commands
$PWD            Current directory
$OLDPWD         Previous directory

# Terminal
$TERM           Terminal type
$LANG           System language
$LC_ALL         Locale settings

# Development
$JAVA_HOME      Java installation directory
$PYTHON_PATH    Python module search path
$NODE_PATH      Node.js module path

# Examples:
echo "My username is: $USER"
echo "My home directory is: $HOME"
echo "Current directory: $PWD"
echo "My shell is: $SHELL"
```

---

## 🛣️ Understanding PATH

**PATH = Where Linux looks for commands**

```bash
# View PATH
echo $PATH

# Output:
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/home/john/.local/bin
│              │         │    │          │     │
│              │         │    │          │     └─ Personal bin
│              │         │    │          └─ System admin binaries
│              │         │    └─ System binaries
│              │         └─ Basic commands
│              └─ User programs
└─ Locally installed programs

Separated by colons (:)
Searched left to right
First match wins
```

**How PATH works:**

```bash
# When you type:
nginx

# Linux searches in order:
1. /usr/local/bin/nginx     (not found)
2. /usr/bin/nginx           (not found)
3. /bin/nginx               (not found)
4. /usr/sbin/nginx          (FOUND! ✅ Execute this)

# If not in PATH, you need full path:
/usr/sbin/nginx

# Or add directory to PATH:
export PATH=$PATH:/usr/sbin
nginx    # Now works!
```

---

## ➕ Adding to PATH

### **Temporary (current session only)**

```bash
# Add to end of PATH
export PATH=$PATH:/new/directory

# Add to beginning of PATH (higher priority)
export PATH=/new/directory:$PATH

# Example: Add custom bin directory
export PATH=$PATH:$HOME/bin

# Verify
echo $PATH

# Test
which mycommand
```

### **Permanent (survives reboots)**

```bash
# Edit .bashrc (for bash)
nano ~/.bashrc

# Add at end of file:
export PATH=$PATH:$HOME/bin

# Save and reload
source ~/.bashrc

# Or
. ~/.bashrc

# Verify
echo $PATH
```

---

## ⚙️ Setting Environment Variables

### **Temporary Variables**

```bash
# Set for current session
export MY_VAR="hello"

# Use the variable
echo $MY_VAR

# Set multiple
export VAR1="value1"
export VAR2="value2"

# Use in commands
export DATABASE_URL="mysql://localhost/mydb"
python manage.py migrate

# Unset variable
unset MY_VAR
```

### **One-Time Use**

```bash
# Set for single command only
MY_VAR="value" command

# Example:
PORT=3000 node server.js
DEBUG=true python app.py

# Multiple variables
VAR1=val1 VAR2=val2 command
```

---

## 📝 Shell Configuration Files

**Different files, different purposes:**

```bash
~/.bashrc           Executed for interactive shells
                    ✅ Use for: aliases, functions, PATH
                    ✅ Loaded: Every new terminal

~/.bash_profile     Executed for login shells
~/.profile          Alternative to .bash_profile
                    ✅ Use for: environment variables
                    ✅ Loaded: When you login

~/.bash_logout      Executed when you logout
                    ✅ Use for: cleanup tasks

/etc/profile        System-wide settings
/etc/bash.bashrc    System-wide bashrc
                    ✅ Applied to all users
                    ⚠️  Need sudo to edit
```

**Which file to use?**

```bash
# For environment variables (PATH, JAVA_HOME, etc.)
~/.bash_profile     # or ~/.profile

# For aliases and shell functions
~/.bashrc

# Best practice: Have .bash_profile source .bashrc
# Add to ~/.bash_profile:
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

---

## 🔧 Practical Examples

### **Common .bashrc additions:**

```bash
# Edit .bashrc
nano ~/.bashrc

# Add these at the end:

# ---------- Aliases ----------
alias ll='ls -lah'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias update='sudo apt update && sudo apt upgrade -y'
alias ports='sudo netstat -tulanp'

# ---------- Environment Variables ----------
export EDITOR=nano
export VISUAL=nano

# ---------- Custom PATH ----------
export PATH=$PATH:$HOME/bin
export PATH=$PATH:$HOME/.local/bin

# ---------- Custom Prompt ----------
export PS1='\u@\h:\w\$ '

# ---------- History Settings ----------
export HISTSIZE=10000
export HISTFILESIZE=20000

# ---------- Functions ----------
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Save and reload:
source ~/.bashrc
```

### **Development environment variables:**

```bash
# Add to ~/.bashrc or ~/.profile

# Java
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin

# Python
export PYTHONPATH=$HOME/python-libs:$PYTHONPATH

# Node.js
export NODE_ENV=development

# Go
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Docker
export DOCKER_HOST=unix:///var/run/docker.sock

# AWS
export AWS_PROFILE=production
export AWS_REGION=us-east-1

# Database
export DATABASE_URL=postgresql://localhost/mydb
export REDIS_URL=redis://localhost:6379
```

### **Project-specific variables:**

```bash
# Create .env file in project
nano .env

DATABASE_URL=postgresql://localhost/mydb
SECRET_KEY=your-secret-key
DEBUG=true
API_KEY=your-api-key

# Load in script
source .env
echo $DATABASE_URL

# Or use in Python (with python-dotenv)
pip install python-dotenv

# In Python:
from dotenv import load_dotenv
load_dotenv()
```

---

## 🎨 Customizing Your Prompt

**PS1 = Primary Prompt String**

```bash
# Basic prompt
export PS1='\u@\h:\w\$ '
# Output: john@laptop:~/projects$

# Colored prompt
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
# Output: john@laptop:~/projects$ (with colors)

# Show time
export PS1='[\t] \u@\h:\w\$ '
# Output: [14:30:15] john@laptop:~/projects$

# Git branch in prompt (requires git-prompt.sh)
source /etc/bash_completion.d/git-prompt
export PS1='\u@\h:\w$(__git_ps1 " (%s)")\$ '
# Output: john@laptop:~/project (main)$
```

**PS1 variables:**
```
\u    Username
\h    Hostname
\w    Current working directory (full path)
\W    Current directory (basename only)
\t    Time (24-hour HH:MM:SS)
\d    Date
\$    $ for user, # for root
\n    Newline
```

---

## 🚀 Advanced Variable Techniques

### **Default values:**

```bash
# Use default if variable not set
echo ${VAR:-default}

# Set variable if not set
: ${VAR:=default}

# Example:
PORT=${PORT:-3000}
echo "Server running on port $PORT"
```

### **String manipulation:**

```bash
# Get string length
VAR="hello"
echo ${#VAR}        # Output: 5

# Substring
VAR="hello world"
echo ${VAR:0:5}     # Output: hello
echo ${VAR:6}       # Output: world

# Replace
VAR="hello world"
echo ${VAR/world/universe}    # Output: hello universe

# Uppercase/Lowercase
VAR="hello"
echo ${VAR^^}       # Output: HELLO
echo ${VAR,,}       # Output: hello (if was uppercase)
```

### **Check if variable exists:**

```bash
# Check if set
if [ -z "$VAR" ]; then
    echo "VAR is not set"
fi

# Check if set and not empty
if [ -n "$VAR" ]; then
    echo "VAR is set to: $VAR"
fi
```

---

## 🔍 Debugging Environment

```bash
# Show all environment variables
env | sort
printenv | sort

# Search for specific variable
env | grep PATH
env | grep JAVA

# Show shell variables
set | grep VAR

# Check which config file sets a variable
grep -r "MY_VAR" ~/.bashrc ~/.bash_profile ~/.profile

# See what's loaded at startup
bash -x ~/.bashrc

# Test without loading configs
bash --norc

# Show where command is found
which python
type python
command -v python
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you display all environment variables?**
   <details>
   <summary>Answer</summary>
   env  OR  printenv
   </details>

2. **How do you view the PATH variable?**
   <details>
   <summary>Answer</summary>
   echo $PATH
   </details>

3. **How do you add a directory to PATH permanently?**
   <details>
   <summary>Answer</summary>
   Add to ~/.bashrc: export PATH=$PATH:/new/directory
   Then: source ~/.bashrc
   </details>

4. **How do you set a temporary environment variable?**
   <details>
   <summary>Answer</summary>
   export VAR_NAME="value"
   </details>

5. **How do you reload .bashrc without restarting terminal?**
   <details>
   <summary>Answer</summary>
   source ~/.bashrc  OR  . ~/.bashrc
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. View all environment variables
env | less

# 2. Check your current PATH
echo $PATH

# 3. Check your home directory
echo $HOME

# 4. Set temporary variable
export MY_NAME="YourName"
echo "Hello, $MY_NAME"

# 5. Create custom bin directory
mkdir -p ~/bin

# 6. Create test script
cat > ~/bin/hello << 'EOF'
#!/bin/bash
echo "Hello from custom script!"
EOF

# 7. Make it executable
chmod +x ~/bin/hello

# 8. Try to run (should fail)
hello

# 9. Add to PATH in .bashrc
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc

# 10. Reload .bashrc
source ~/.bashrc

# 11. Now try again (should work!)
hello

# 12. Verify PATH
echo $PATH | grep "$HOME/bin"

# 13. Create an alias
echo 'alias myls="ls -lah"' >> ~/.bashrc
source ~/.bashrc

# 14. Test alias
myls
```

---

## 📝 Key Takeaways

✅ **env** - show all environment variables  
✅ **echo $VAR** - show specific variable  
✅ **export VAR=value** - set temporary variable  
✅ **$PATH** - where Linux finds commands  
✅ **~/.bashrc** - for aliases and interactive shell config  
✅ **~/.bash_profile** - for environment variables  
✅ **source ~/.bashrc** - reload configuration  
✅ **export PATH=$PATH:/dir** - add to PATH  

---

## 🚀 Next Steps

You can now customize your shell environment!

**Next lesson:** [12-archives-compression.md](12-archives-compression.md) - Working with archives and compression

---

## 💡 Pro Tips

**Quick environment setup:**
```bash
# Create organized .bashrc sections
nano ~/.bashrc

# ========== ALIASES ==========
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade'

# ========== ENVIRONMENT ==========
export EDITOR=nano

# ========== PATH ==========
export PATH=$PATH:$HOME/bin

# ========== FUNCTIONS ==========
mkcd() { mkdir -p "$1" && cd "$1"; }

# Save and reload
source ~/.bashrc
```

**Debugging PATH issues:**
```bash
# Which command is being used?
which python
type python

# Show all matches in PATH
which -a python

# Debug PATH
echo $PATH | tr ':' '\n'    # Show one per line
```

**Backup before modifying:**
```bash
# Always backup config files
cp ~/.bashrc ~/.bashrc.backup

# If something breaks:
cp ~/.bashrc.backup ~/.bashrc
source ~/.bashrc
```

**Template .bashrc:**
```bash
# Download a good template
wget https://raw.githubusercontent.com/username/dotfiles/main/.bashrc -O ~/.bashrc

# Or create your own and version control it
cd ~
git init dotfiles
cp .bashrc dotfiles/
cd dotfiles
git add .bashrc
git commit -m "My bashrc config"
```

Master environment variables = Customize your Linux experience! 🚀
