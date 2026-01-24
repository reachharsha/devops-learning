---
render_with_liquid: false
---
# 05 - File Permissions

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Linux permission system (read, write, execute)
- Understanding permission notation (rwx and numeric)
- Changing permissions with chmod
- Changing ownership with chown
- Security implications and best practices

---

## 🔐 Understanding Linux Permissions

**Real-World Analogy:**
```
File = House
Permissions = Access Rules

Owner (You):        Can enter, renovate, sell
Group (Family):     Can enter, use kitchen
Others (Strangers): Can only see from street

read (r)    = Look inside (view)
write (w)   = Modify/delete
execute (x) = Enter/run
```

---

## 👥 Three Permission Levels

Every file has three permission sets:

```
1. User (u):   File owner
2. Group (g):  Group members
3. Others (o): Everyone else
```

**Example:**
```bash
ls -l file.txt

-rw-r--r--  1 john  developers  1234 Jan 23 10:30 file.txt
 │││ │││ │││    │       │
 │││ │││ │││    │       └─ Group
 │││ │││ │││    └─ Owner
 │││ │││ │└─ Others permissions
 │││ ││└─ Group permissions  
 │││ │└─ User/Owner permissions
 ││└─ File type (- = file, d = directory, l = link)
 └─ Permission string
```

---

## 📋 Permission Types

### **Read (r)**
```
Files:       View contents (cat, less, more)
Directories: List contents (ls)

Symbol: r
Numeric: 4
```

### **Write (w)**
```
Files:       Modify/delete file
Directories: Create/delete files inside

Symbol: w
Numeric: 2
```

### **Execute (x)**
```
Files:       Run as program/script
Directories: Enter directory (cd into it)

Symbol: x
Numeric: 1
```

---

## 🔢 Permission Notation

### **Symbolic Notation (rwx)**

```bash
ls -l

-rw-r--r--  file.txt
 ││││││││
 ││││││└└─ Others: r-- (read only)
 │││└└└─── Group:  r-- (read only)
 └└└────── User:   rw- (read, write)

drwxr-xr-x  directory/
 ││││││││
 ││││││└└─ Others: r-x (read, execute = can list and enter)
 │││└└└─── Group:  r-x (read, execute)
 └└└────── User:   rwx (read, write, execute = full control)
```

### **Numeric Notation (Octal)**

Each permission has a number:
```
read (r)    = 4
write (w)   = 2
execute (x) = 1
none (-)    = 0

Add them up for each level:

rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0
```

**Complete permission number:**
```
User  Group  Others
 7      5      5    = 755
rwx    r-x    r-x

 6      4      4    = 644
rw-    r--    r--

 7      0      0    = 700
rwx    ---    ---
```

---

## 🛠️ chmod - Change Permissions

### **Using Numeric Method:**

```bash
# Common patterns:
chmod 644 file.txt     # rw-r--r-- (files - owner can edit, others read)
chmod 755 script.sh    # rwxr-xr-x (scripts - owner can edit/run, others run)
chmod 600 private.txt  # rw------- (private - only owner)
chmod 777 public.sh    # rwxrwxrwx (all access - AVOID!)
chmod 700 private-dir/ # rwx------ (private directory)

# Examples:
chmod 644 document.txt          # Standard file permissions
chmod 755 /usr/local/bin/script # Standard executable permissions
chmod 600 ~/.ssh/id_rsa         # Private SSH key
chmod 700 ~/.ssh                # SSH directory
```

### **Using Symbolic Method:**

**Syntax:** `chmod [who][operation][permission] file`

**Who:**
```
u = user (owner)
g = group
o = others
a = all (user + group + others)
```

**Operation:**
```
+ = add permission
- = remove permission
= = set exact permission
```

**Permission:**
```
r = read
w = write
x = execute
```

**Examples:**
```bash
# Add execute permission for owner
chmod u+x script.sh

# Remove write permission for group
chmod g-w file.txt

# Add read for everyone
chmod a+r file.txt

# Set exact permissions for owner
chmod u=rwx file.txt

# Multiple changes
chmod u+x,g-w,o-r file.txt

# Give group same permissions as owner
chmod g=u file.txt

# Remove all permissions for others
chmod o= file.txt

# Add execute for owner and group
chmod ug+x script.sh
```

### **Recursive chmod:**

```bash
# Change all files in directory
chmod -R 755 directory/

# Change only directories
find . -type d -exec chmod 755 {} \;

# Change only files
find . -type f -exec chmod 644 {} \;
```

---

## 👤 chown - Change Ownership

**Syntax:** `chown [owner][:group] file`

```bash
# Change owner only
chown john file.txt

# Change owner and group
chown john:developers file.txt

# Change group only
chown :developers file.txt
# Or use chgrp:
chgrp developers file.txt

# Recursive
chown -R john:developers directory/

# Change based on reference file
chown --reference=file1.txt file2.txt

# Preserve root directory (safety)
chown -R john:developers . --preserve-root
```

**Practical examples:**
```bash
# After creating file as root, give to user
sudo touch /etc/config.conf
sudo chown john:developers /etc/config.conf

# Change ownership of web files to web server
sudo chown -R www-data:www-data /var/www/html/

# Fix home directory ownership
sudo chown -R $USER:$USER ~

# Change ownership but preserve group
sudo chown john file.txt
```

---

## 🔍 Viewing Permissions

```bash
# Detailed listing
ls -l

# Show numeric permissions
stat file.txt

# Or use stat with format
stat -c "%a %n" file.txt
# Output: 644 file.txt

# View permissions for current directory
ls -ld .

# View directory and all contents
ls -la

# Show file type and permissions
file file.txt
```

---

## 🎯 Common Permission Patterns

### **Files:**

**Regular documents:**
```bash
644 (rw-r--r--)
Owner: Read + Write
Group: Read
Others: Read

chmod 644 document.txt
```

**Private files:**
```bash
600 (rw-------)
Owner: Read + Write
Group: None
Others: None

chmod 600 ~/.ssh/id_rsa
chmod 600 ~/private-notes.txt
```

**Executable scripts:**
```bash
755 (rwxr-xr-x)
Owner: Read + Write + Execute
Group: Read + Execute
Others: Read + Execute

chmod 755 script.sh
chmod 755 /usr/local/bin/my-command
```

### **Directories:**

**Standard directory:**
```bash
755 (rwxr-xr-x)
Owner: Full access
Group: Read + Enter
Others: Read + Enter

chmod 755 public-folder/
```

**Private directory:**
```bash
700 (rwx------)
Owner: Full access
Group: No access
Others: No access

chmod 700 ~/.ssh
chmod 700 ~/private-data/
```

**Shared directory:**
```bash
775 (rwxrwxr-x)
Owner: Full access
Group: Full access
Others: Read + Enter

chmod 775 /shared/team-folder/
```

---

## 🚨 Special Permissions

### **Setuid (Set User ID)**
```
File runs with owner's permissions (not user who runs it)

Symbol: s in user execute position (rws)
Numeric: 4000

chmod u+s program
chmod 4755 program

Example: passwd command
-rwsr-xr-x /usr/bin/passwd
```

### **Setgid (Set Group ID)**
```
Directory: Files created inside inherit directory's group
File: Runs with group's permissions

Symbol: s in group execute position
Numeric: 2000

chmod g+s directory/
chmod 2755 directory/

Example: Shared project directory
drwxr-sr-x /projects/team/
```

### **Sticky Bit**
```
Directory: Only owner can delete their files (even if others can write)

Symbol: t in others execute position
Numeric: 1000

chmod +t directory/
chmod 1777 directory/

Example: /tmp directory
drwxrwxrwt /tmp
Anyone can create files, but can only delete their own
```

**Combined special permissions:**
```bash
chmod 4755 file    # Setuid
chmod 2755 dir/    # Setgid
chmod 1777 dir/    # Sticky bit
chmod 6755 file    # Setuid + Setgid
```

---

## ⚠️ Security Best Practices

### **DO:**
```bash
✅ Use 644 for regular files
✅ Use 600 for sensitive files (SSH keys, passwords)
✅ Use 755 for executables and directories
✅ Use 700 for private directories
✅ Regularly audit permissions: find / -perm -002
✅ Use least privilege principle
```

### **DON'T:**
```bash
❌ chmod 777 anything (world-writable = dangerous!)
❌ chmod -R 777 / (NEVER!)
❌ Give execute permission to data files
❌ Make config files world-readable if they contain secrets
❌ Run services as root unnecessarily
```

### **Common Mistakes:**
```bash
# DANGEROUS: Makes everything readable/writable/executable by everyone
chmod -R 777 /var/www/
# Instead:
find /var/www -type d -exec chmod 755 {} \;
find /var/www -type f -exec chmod 644 {} \;

# DANGEROUS: Running as root when not needed
sudo chmod 777 file.txt
# Instead: Fix ownership and use proper permissions
chown $USER file.txt
chmod 644 file.txt
```

---

## 🔧 Practical Scenarios

### **Setting up SSH:**
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa         # Private key
chmod 644 ~/.ssh/id_rsa.pub     # Public key
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/known_hosts
chmod 600 ~/.ssh/config
```

### **Web server files:**
```bash
# Directories: 755 (rwxr-xr-x)
find /var/www/html -type d -exec chmod 755 {} \;

# Files: 644 (rw-r--r--)
find /var/www/html -type f -exec chmod 644 {} \;

# Set ownership to web server
chown -R www-data:www-data /var/www/html
```

### **Shared project directory:**
```bash
# Create shared directory
mkdir /projects/team-project

# Set group ownership
chown :developers /projects/team-project

# Set permissions: owner and group can write
chmod 775 /projects/team-project

# Set setgid: new files inherit group
chmod g+s /projects/team-project

# Now all developers can collaborate!
```

### **Backup scripts:**
```bash
# Make script executable
chmod +x backup.sh

# Only owner can modify and run
chmod 700 backup.sh

# If contains passwords, extra secure
chmod 600 backup.sh
# Then run with: bash backup.sh
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does 755 mean?**
   <details>
   <summary>Answer</summary>
   rwxr-xr-x - Owner: read+write+execute, Group: read+execute, Others: read+execute
   </details>

2. **How do you make a file readable by everyone?**
   <details>
   <summary>Answer</summary>
   chmod a+r file.txt or chmod 644 file.txt
   </details>

3. **Why is chmod 777 dangerous?**
   <details>
   <summary>Answer</summary>
   Gives everyone (including attackers) full access: read, write, execute. Major security risk.
   </details>

4. **What's the difference between chmod and chown?**
   <details>
   <summary>Answer</summary>
   chmod changes permissions (rwx). chown changes ownership (who owns the file).
   </details>

5. **What permissions should SSH private key have?**
   <details>
   <summary>Answer</summary>
   600 (rw-------) - Only owner can read/write, nobody else has access
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Create test directory
mkdir ~/permissions-practice
cd ~/permissions-practice

# 2. Create test file
echo "Test content" > test.txt

# 3. Check current permissions
ls -l test.txt

# 4. Remove all permissions for others
chmod o-rwx test.txt
ls -l test.txt

# 5. Add execute permission for owner
chmod u+x test.txt
ls -l test.txt

# 6. Set specific permissions
chmod 644 test.txt
ls -l test.txt

# 7. View numeric permissions
stat -c "%a %n" test.txt

# 8. Create directory
mkdir test-dir

# 9. Set directory permissions
chmod 755 test-dir
ls -ld test-dir

# 10. Create script
echo '#!/bin/bash' > script.sh
echo 'echo "Hello"' >> script.sh

# 11. Make executable
chmod +x script.sh

# 12. Run it
./script.sh

# 13. View all permissions
ls -la

# 14. Clean up
cd ..
rm -r permissions-practice/
```

---

## 📝 Key Takeaways

✅ Three permission types: **read (r/4), write (w/2), execute (x/1)**  
✅ Three permission levels: **user, group, others**  
✅ **chmod** changes permissions (numeric or symbolic)  
✅ **chown** changes ownership  
✅ **644** = standard file permissions  
✅ **755** = standard directory/executable permissions  
✅ **600** = private files (SSH keys)  
✅ **700** = private directories  
✅ **Never use 777!** Major security risk  
✅ Directories need **execute (x)** permission to enter them  

---

## 🚀 Next Steps

You understand Linux permissions and security!

**Next lesson:** [06-text-processing.md](06-text-processing.md) - Searching and manipulating text

---

## 💡 Pro Tips

**Quick reference card:**
```
644 = -rw-r--r--  Files
755 = -rwxr-xr-x  Directories/Executables
600 = -rw-------  Private files
700 = -rwx------  Private directories
```

**Remember:**
```
Directories NEED execute permission to cd into them!

chmod 644 directory/  # Can't enter! (no x)
chmod 755 directory/  # Can enter ✅
```

**Security checklist:**
```bash
# Find world-writable files (security risk)
find / -type f -perm -002 2>/dev/null

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null

# Find setuid files (potential risk)
find / -perm -4000 2>/dev/null
```

Master permissions = master security! 🔐
