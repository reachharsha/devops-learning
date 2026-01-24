---
render_with_liquid: false
---
# 12 - Archives & Compression

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Creating and extracting tar archives
- Compressing files with gzip, bzip2, xz
- Working with zip files
- Choosing the right compression method
- Archiving for backups

---

## 📦 Archives vs Compression

**Real-World Analogy:**
```
Archive = Moving box
Put many items into one container
Easier to move, organize, share

Compression = Vacuum seal
Makes the box smaller
Saves space, faster to send

tar        = Archive (combine files)
gzip       = Compression (make smaller)
tar.gz     = Both! (combine + compress)
```

**Why do we need this?**
```
DevOps use cases:
✅ Backup databases
✅ Deploy applications
✅ Share log files
✅ Distribute software
✅ Save disk space
✅ Faster transfers
```

---

## 📦 TAR - Tape Archive

**tar = Archive multiple files/folders into one file**

### **Creating Archives**

```bash
# Create tar archive
tar -cvf archive.tar files/

# Options:
-c    Create archive
-v    Verbose (show files being added)
-f    File name

# Example:
tar -cvf backup.tar /home/john/documents/

# Create from multiple sources
tar -cvf archive.tar file1.txt file2.txt folder/

# Create without verbose
tar -cf archive.tar files/
```

### **Extracting Archives**

```bash
# Extract tar archive
tar -xvf archive.tar

# Options:
-x    Extract
-v    Verbose
-f    File name

# Extract to specific directory
tar -xvf archive.tar -C /destination/path/

# List contents without extracting
tar -tvf archive.tar

# Extract specific file
tar -xvf archive.tar path/to/specific/file

# Extract specific directory
tar -xvf archive.tar folder/
```

---

## 🗜️ TAR with Compression

**Combine archiving + compression**

### **GZIP (.tar.gz or .tgz)**

**Most common, good balance of speed and compression**

```bash
# Create compressed archive
tar -czvf archive.tar.gz files/
tar -czvf archive.tgz files/        # Same thing

# Options:
-z    Use gzip compression

# Extract compressed archive
tar -xzvf archive.tar.gz

# Extract to specific directory
tar -xzvf archive.tar.gz -C /destination/

# List contents
tar -tzvf archive.tar.gz

# Example: Backup website
tar -czvf website-backup-$(date +%Y%m%d).tar.gz /var/www/html/

# Output: website-backup-20240115.tar.gz
```

### **BZIP2 (.tar.bz2)**

**Better compression, slower speed**

```bash
# Create bzip2 compressed archive
tar -cjvf archive.tar.bz2 files/

# Options:
-j    Use bzip2 compression

# Extract bzip2 archive
tar -xjvf archive.tar.bz2

# List contents
tar -tjvf archive.tar.bz2

# Example:
tar -cjvf logs-backup.tar.bz2 /var/log/
```

### **XZ (.tar.xz)**

**Best compression, slowest speed**

```bash
# Create xz compressed archive
tar -cJvf archive.tar.xz files/

# Options:
-J    Use xz compression (capital J!)

# Extract xz archive
tar -xJvf archive.tar.xz

# Example:
tar -cJvf database-backup.tar.xz /var/lib/mysql/
```

---

## 📊 Compression Comparison

```bash
Original size:    100 MB

gzip (.tar.gz):   25 MB   ⚡ Fast      ✅ Common
bzip2 (.tar.bz2): 20 MB   ⚡⚡ Medium   ✅ Better compression
xz (.tar.xz):     15 MB   ⚡⚡⚡ Slow    ✅ Best compression

Choose based on:
Speed needed:     gzip
Max compression:  xz
Best balance:     gzip (most used)
```

---

## 🗜️ Individual File Compression

### **GZIP**

```bash
# Compress file (replaces original)
gzip file.txt
# Creates: file.txt.gz (original is gone)

# Compress and keep original
gzip -k file.txt
gzip --keep file.txt

# Decompress
gunzip file.txt.gz
gzip -d file.txt.gz

# Compress multiple files
gzip file1.txt file2.txt file3.txt

# Compress with specific level (1=fast/less, 9=slow/best)
gzip -9 file.txt        # Maximum compression
gzip -1 file.txt        # Fastest compression

# View compressed file without decompressing
zcat file.txt.gz
zless file.txt.gz
zmore file.txt.gz
zgrep "pattern" file.txt.gz
```

### **BZIP2**

```bash
# Compress file
bzip2 file.txt
# Creates: file.txt.bz2

# Keep original
bzip2 -k file.txt

# Decompress
bunzip2 file.txt.bz2
bzip2 -d file.txt.bz2

# View compressed file
bzcat file.txt.bz2
bzless file.txt.bz2
bzgrep "pattern" file.txt.bz2
```

### **XZ**

```bash
# Compress file
xz file.txt
# Creates: file.txt.xz

# Keep original
xz -k file.txt

# Decompress
unxz file.txt.xz
xz -d file.txt.xz

# View compressed file
xzcat file.txt.xz
xzless file.txt.xz
```

---

## 🤐 ZIP Files

**Cross-platform (Windows, Mac, Linux)**

```bash
# Install if needed
sudo apt install zip unzip

# Create zip archive
zip archive.zip file1.txt file2.txt

# Zip entire directory
zip -r archive.zip folder/

# Options:
-r    Recursive (include subdirectories)
-q    Quiet (no output)
-9    Maximum compression

# Add files to existing zip
zip archive.zip newfile.txt

# Extract zip
unzip archive.zip

# Extract to specific directory
unzip archive.zip -d /destination/

# List contents without extracting
unzip -l archive.zip

# Test zip integrity
unzip -t archive.zip

# Extract specific file
unzip archive.zip file.txt

# Exclude files when creating
zip -r archive.zip folder/ -x "*.log" "*.tmp"

# Password protect
zip -e -r secure.zip folder/
# Prompts for password

# Extract password-protected
unzip -P password secure.zip
```

---

## 🎯 Practical Examples

### **Website Backup**

```bash
# Backup website with date
tar -czvf website-backup-$(date +%Y%m%d-%H%M%S).tar.gz /var/www/html/

# Output: website-backup-20240115-143022.tar.gz

# Exclude cache and logs
tar -czvf website.tar.gz \
    --exclude='*.log' \
    --exclude='cache/*' \
    /var/www/html/
```

### **Database Backup**

```bash
# MySQL dump and compress
mysqldump -u root -p database_name | gzip > db-backup-$(date +%Y%m%d).sql.gz

# Restore
gunzip < db-backup-20240115.sql.gz | mysql -u root -p database_name

# PostgreSQL
pg_dump database_name | gzip > db-backup.sql.gz
```

### **Log Files**

```bash
# Compress old logs
gzip /var/log/app.log.1
gzip /var/log/app.log.2

# Archive and compress logs
tar -czvf logs-$(date +%Y%m).tar.gz /var/log/app/*.log

# Compress logs older than 7 days
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;
```

### **Application Deployment**

```bash
# Create deployment package
tar -czvf app-v1.2.0.tar.gz \
    --exclude='node_modules' \
    --exclude='.git' \
    --exclude='*.log' \
    /opt/myapp/

# Extract on server
tar -xzvf app-v1.2.0.tar.gz -C /opt/
```

### **Home Directory Backup**

```bash
# Backup home directory
tar -czvf home-backup-$(date +%Y%m%d).tar.gz \
    --exclude='Downloads' \
    --exclude='*.cache' \
    --exclude='node_modules' \
    $HOME

# Restore
tar -xzvf home-backup-20240115.tar.gz -C /
```

---

## 🔧 Advanced tar Options

```bash
# Exclude patterns
tar -czvf backup.tar.gz \
    --exclude='*.log' \
    --exclude='*.tmp' \
    --exclude='node_modules' \
    /path/to/backup/

# Exclude file (list of patterns)
tar -czvf backup.tar.gz --exclude-from=exclude.txt /path/

# File: exclude.txt
*.log
*.tmp
.git
node_modules
__pycache__

# Show progress with size
tar -czvf backup.tar.gz --checkpoint=1000 --checkpoint-action=dot /path/

# Create with specific permissions
tar -czvf backup.tar.gz --preserve-permissions /path/

# Append to existing archive (uncompressed only)
tar -rvf archive.tar newfile.txt

# Compare archive with filesystem
tar -dvf archive.tar

# Extract with different owner
sudo tar -xzvf backup.tar.gz --same-owner

# Extract preserving permissions
tar -xzvf backup.tar.gz --preserve-permissions
```

---

## 📊 Archive Information

```bash
# Check archive size
ls -lh archive.tar.gz

# Detailed listing
tar -tzvf archive.tar.gz

# Count files in archive
tar -tzf archive.tar.gz | wc -l

# Find file in archive
tar -tzf archive.tar.gz | grep filename

# Show only directories
tar -tzf archive.tar.gz | grep '/$'

# Compression ratio
du -h original/
ls -lh archive.tar.gz
```

---

## 🚨 Troubleshooting

### **File not found in archive**

```bash
# List full paths
tar -tzf archive.tar.gz

# Extract with path
tar -xzf archive.tar.gz path/to/file

# Strip leading directories
tar -xzf archive.tar.gz --strip-components=1
```

### **Permission denied**

```bash
# Extract as root
sudo tar -xzvf archive.tar.gz

# Change ownership after extraction
sudo tar -xzvf backup.tar.gz
sudo chown -R user:group extracted-dir/
```

### **Corrupted archive**

```bash
# Test integrity
tar -tzf archive.tar.gz > /dev/null
gzip -t file.gz
bzip2 -t file.bz2
zip -T archive.zip

# Try to recover
tar -xzvf archive.tar.gz --ignore-failed-read
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you create a compressed tar archive?**
   <details>
   <summary>Answer</summary>
   tar -czvf archive.tar.gz directory/
   </details>

2. **How do you extract a .tar.gz file?**
   <details>
   <summary>Answer</summary>
   tar -xzvf archive.tar.gz
   </details>

3. **How do you list contents of an archive without extracting?**
   <details>
   <summary>Answer</summary>
   tar -tzvf archive.tar.gz
   </details>

4. **What's the difference between tar and tar.gz?**
   <details>
   <summary>Answer</summary>
   tar = archive only; tar.gz = archive + gzip compression
   </details>

5. **How do you create a zip file?**
   <details>
   <summary>Answer</summary>
   zip -r archive.zip directory/
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Create test directory with files
mkdir test-archive
cd test-archive
echo "File 1" > file1.txt
echo "File 2" > file2.txt
mkdir subdir
echo "File 3" > subdir/file3.txt

# 2. Create tar archive
tar -cvf test.tar *

# 3. List contents
tar -tvf test.tar

# 4. Create compressed version
tar -czvf test.tar.gz *

# 5. Compare sizes
ls -lh test.tar test.tar.gz

# 6. Extract to new location
mkdir extracted
tar -xzvf test.tar.gz -C extracted/

# 7. Verify
ls -R extracted/

# 8. Create zip file
zip -r test.zip *

# 9. List zip contents
unzip -l test.zip

# 10. Compress individual file
gzip file1.txt

# 11. View without extracting
zcat file1.txt.gz

# 12. Clean up
cd ..
rm -rf test-archive
```

---

## 📝 Key Takeaways

✅ **tar -czvf archive.tar.gz dir/** - create compressed archive  
✅ **tar -xzvf archive.tar.gz** - extract archive  
✅ **tar -tzvf archive.tar.gz** - list contents  
✅ **zip -r archive.zip dir/** - create zip  
✅ **unzip archive.zip** - extract zip  
✅ **gzip file** - compress individual file  
✅ **gunzip file.gz** - decompress file  
✅ **.tar.gz** - most common (gzip)  
✅ **.tar.bz2** - better compression (bzip2)  
✅ **.tar.xz** - best compression (xz)  

---

## 🚀 Next Steps

You can now create, compress, and extract archives!

**Next lesson:** [13-io-redirection.md](13-io-redirection.md) - Input/Output redirection and pipes

---

## 💡 Pro Tips

**Quick compression cheat sheet:**
```bash
# Archive + compress (most common)
tar -czvf file.tar.gz dir/          # Create
tar -xzvf file.tar.gz               # Extract

# Just archive (no compression)
tar -cvf file.tar dir/              # Create
tar -xvf file.tar                   # Extract

# Better compression
tar -cjvf file.tar.bz2 dir/         # Create (bzip2)
tar -xjvf file.tar.bz2              # Extract

# Best compression
tar -cJvf file.tar.xz dir/          # Create (xz)
tar -xJvf file.tar.xz               # Extract

# Cross-platform
zip -r file.zip dir/                # Create
unzip file.zip                      # Extract
```

**Backup script:**
```bash
#!/bin/bash
# daily-backup.sh

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d)

# Backup website
tar -czvf $BACKUP_DIR/website-$DATE.tar.gz /var/www/html/

# Backup database
mysqldump -u root -p$(cat /root/.mysql_password) database | gzip > $BACKUP_DIR/db-$DATE.sql.gz

# Delete backups older than 30 days
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: $DATE"
```

**Remember:**
```
c = create
x = extract
z = gzip
j = bzip2
J = xz (capital!)
v = verbose
f = file
t = list
```

Master archives = Efficient backups and deployments! 🚀
