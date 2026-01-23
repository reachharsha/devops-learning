# 04 - File Operations

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Creating files and directories
- Copying and moving files
- Deleting files and directories safely
- Finding and locating files
- File and directory naming best practices

---

## 📝 Creating Files

### **touch - Create Empty File**

```bash
# Create single file
touch file.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Create with path
touch /tmp/test.txt
touch ~/Documents/notes.txt

# Update file timestamp (if exists)
touch existing-file.txt

# Create file with specific timestamp
touch -t 202401231430 file.txt
```

**When to use:**
- Create placeholder files
- Update file modification time
- Create files before editing

---

### **echo - Create File with Content**

```bash
# Create file with text
echo "Hello World" > file.txt

# Append to file
echo "New line" >> file.txt

# Create empty file (alternative to touch)
echo -n > file.txt

# Multiple lines (method 1)
echo "Line 1" > file.txt
echo "Line 2" >> file.txt
echo "Line 3" >> file.txt

# Multiple lines (method 2)
cat > file.txt << EOF
Line 1
Line 2
Line 3
EOF
```

**Difference:**
```
>   Create/overwrite file
>>  Append to file

echo "First" > file.txt     # Creates file
echo "Second" > file.txt    # OVERWRITES! File now has only "Second"
echo "Third" >> file.txt    # Appends: "Second\nThird"
```

---

### **cat - Create File Interactively**

```bash
# Create and type content
cat > file.txt
Type your content here
Press Ctrl+D when done

# Alternative: here document
cat > file.txt << EOF
First line
Second line
Third line
EOF
```

---

## 📁 Creating Directories

### **mkdir - Make Directory**

```bash
# Create single directory
mkdir my-folder

# Create multiple directories
mkdir dir1 dir2 dir3

# Create nested directories
mkdir -p parent/child/grandchild

# Create with specific permissions
mkdir -m 755 public-folder

# Create with path
mkdir ~/Projects/new-project
```

**Important flags:**
```bash
-p    Create parent directories as needed (no error if exists)
-m    Set permissions
-v    Verbose (show what's being created)

# Examples:
mkdir -p /tmp/a/b/c/d        # Creates all levels
mkdir -pv ~/test/sub         # Creates with messages
mkdir -m 700 ~/private       # Only owner can access
```

---

## 📋 Copying Files and Directories

### **cp - Copy**

**Basic usage:**
```bash
# Copy file
cp source.txt destination.txt

# Copy to directory
cp file.txt /tmp/

# Copy multiple files
cp file1.txt file2.txt dir3.txt /destination/

# Copy with new name
cp oldname.txt newname.txt
```

**Common options:**
```bash
# Recursive (copy directories)
cp -r source-dir/ destination-dir/

# Preserve attributes (timestamps, permissions)
cp -p file.txt backup.txt

# Interactive (ask before overwriting)
cp -i file.txt existing.txt

# Verbose (show what's being copied)
cp -v file.txt /tmp/

# Combination (recursive, preserve, verbose)
cp -rpv source/ destination/

# Create backup of destination if exists
cp -b file.txt existing.txt    # Creates existing.txt~

# Force overwrite
cp -f file.txt destination.txt
```

**Practical examples:**
```bash
# Backup file
cp important.txt important.txt.backup
cp important.txt important.txt.$(date +%Y%m%d)

# Copy entire directory
cp -r project/ project-backup/

# Copy only .txt files
cp *.txt /backup/

# Copy maintaining structure
cp -r --parents path/to/file /backup/
# Creates: /backup/path/to/file

# Copy only if newer
cp -u source.txt destination.txt
```

---

## 🚚 Moving and Renaming

### **mv - Move/Rename**

**Move files:**
```bash
# Move file to directory
mv file.txt /tmp/

# Move multiple files
mv file1.txt file2.txt file3.txt /destination/

# Move directory
mv old-directory/ new-location/
```

**Rename files:**
```bash
# Rename file
mv oldname.txt newname.txt

# Rename directory
mv old-folder/ new-folder/
```

**Common options:**
```bash
# Interactive (ask before overwriting)
mv -i file.txt existing.txt

# Never overwrite
mv -n file.txt existing.txt

# Force overwrite
mv -f file.txt existing.txt

# Backup if destination exists
mv -b file.txt existing.txt

# Verbose
mv -v file.txt /new/location/

# Update only if source is newer
mv -u old.txt new.txt
```

**Practical examples:**
```bash
# Rename with timestamp
mv log.txt log-$(date +%Y%m%d).txt

# Move all .log files
mv *.log /var/log/archive/

# Rename extension
mv file.txt file.md

# Move and rename
mv /tmp/old.txt ~/Documents/new.txt

# Batch rename (advanced)
for file in *.txt; do
  mv "$file" "${file%.txt}.md"
done
```

---

## 🗑️ Deleting Files and Directories

### **rm - Remove Files**

**⚠️ WARNING: rm is PERMANENT! No recycle bin!**

```bash
# Delete file
rm file.txt

# Delete multiple files
rm file1.txt file2.txt file3.txt

# Interactive (ask confirmation)
rm -i file.txt

# Force (no confirmation)
rm -f file.txt

# Verbose (show what's deleted)
rm -v file.txt

# Delete all .txt files
rm *.txt

# Delete all except pattern
rm !(*.txt)    # Requires: shopt -s extglob
```

**Deleting directories:**
```bash
# Delete empty directory
rmdir empty-folder/

# Delete directory and contents (DANGEROUS!)
rm -r directory/

# Force delete directory (VERY DANGEROUS!)
rm -rf directory/

# Interactive deletion
rm -ri directory/

# Verbose recursive delete
rm -rv directory/
```

**Safety tips:**
```bash
# ALWAYS check before rm -rf
ls directory/        # Verify contents first
rm -ri directory/    # Use -i for important stuff

# Common mistake - DON'T DO THIS:
rm -rf /            # Deletes EVERYTHING!
rm -rf / tmp        # Space typo = disaster

# Safer alternatives:
# 1. Move to trash instead
mkdir -p ~/.trash
mv file.txt ~/.trash/

# 2. Use trash-cli
sudo apt install trash-cli
trash file.txt      # Can restore later
trash-list          # See what's in trash
restore-trash       # Restore files

# 3. Add confirmation alias
alias rm='rm -i'    # Always ask
```

---

## 🔍 Finding Files

### **find - Powerful File Search**

**Basic searches:**
```bash
# Find by name
find . -name "file.txt"

# Find by pattern
find . -name "*.txt"

# Find case-insensitive
find . -iname "FILE.txt"

# Find in specific directory
find /home -name "*.log"

# Find directories only
find . -type d -name "project*"

# Find files only
find . -type f -name "*.sh"
```

**Advanced searches:**
```bash
# Find by size
find . -size +100M          # Larger than 100MB
find . -size -1K            # Smaller than 1KB
find . -size 50M            # Exactly 50MB

# Find by time
find . -mtime -7            # Modified in last 7 days
find . -mtime +30           # Modified more than 30 days ago
find . -mmin -60            # Modified in last 60 minutes

# Find by permissions
find . -perm 644            # Exactly 644
find . -perm -644           # At least 644

# Find by owner
find . -user john
find . -group developers

# Find empty files/directories
find . -empty

# Combine conditions
find . -name "*.log" -size +10M -mtime -7
# .log files larger than 10MB modified in last week
```

**Execute commands on found files:**
```bash
# Delete found files
find . -name "*.tmp" -delete

# Execute command on each file
find . -name "*.txt" -exec cat {} \;

# Execute with confirmation
find . -name "*.old" -ok rm {} \;

# Execute command once with all files
find . -name "*.log" -exec tar czf logs.tar.gz {} +

# Examples:
# Find and delete old logs
find /var/log -name "*.log" -mtime +30 -delete

# Find large files
find . -type f -size +100M -exec ls -lh {} \;

# Find and change permissions
find . -type f -name "*.sh" -exec chmod +x {} \;
```

---

### **locate - Fast File Search**

**Much faster than find (uses database)**

```bash
# Update database first
sudo updatedb

# Find file
locate file.txt

# Case-insensitive search
locate -i FiLe.TxT

# Limit results
locate -n 10 file.txt

# Count matches only
locate -c "*.log"

# Follow symbolic links
locate -L file.txt

# Show existing files only
locate -e file.txt
```

**find vs locate:**
```
find:
✅ Real-time search
✅ Can search by size, time, permissions
✅ Can execute commands on results
❌ Slower

locate:
✅ Very fast
✅ Searches entire system
❌ Uses database (may be outdated)
❌ Limited search criteria

Use find for: Complex searches, recent files
Use locate for: Quick name-based searches
```

---

### **which - Find Command Location**

```bash
# Find command location
which ls
# Output: /usr/bin/ls

which python3
# Output: /usr/bin/python3

which -a python
# Shows all locations in PATH
```

---

### **whereis - Find Binary, Source, Manual**

```bash
# Find binary, source, and manual
whereis ls
# Output: ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz

whereis -b ls     # Binary only
whereis -m ls     # Manual only
whereis -s ls     # Source only
```

---

## 📏 File and Directory Naming Best Practices

### **Good naming:**
```bash
✅ my-project.txt          Lowercase, hyphens
✅ backup_2024-01-23.tar   Underscores, dates
✅ install.sh              Descriptive, extension
✅ user-data.csv           Clear purpose
✅ v1.2.3                  Versioning
```

### **Avoid:**
```bash
❌ My File.txt             Spaces (use quotes or escape)
❌ file(1).txt             Parentheses (can cause issues)
❌ file?.txt               Special characters
❌ $file.txt               Variables symbols
❌ file&more.txt           Shell special characters
❌ -file.txt               Starts with dash (looks like option)
```

### **Handling spaces:**
```bash
# If file has spaces, use quotes
cat "My File.txt"
cp "File With Spaces.txt" "New Location/"

# Or escape spaces
cat My\ File.txt
cp File\ With\ Spaces.txt destination/

# Rename to remove spaces
mv "My File.txt" my-file.txt
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you create an empty file?**
   <details>
   <summary>Answer</summary>
   touch filename.txt
   </details>

2. **What's the difference between cp and mv?**
   <details>
   <summary>Answer</summary>
   cp copies (original remains), mv moves/renames (original removed)
   </details>

3. **How do you delete a directory and all its contents?**
   <details>
   <summary>Answer</summary>
   rm -r directory/ (or rm -rf for force, but be careful!)
   </details>

4. **How do you find all .log files larger than 10MB?**
   <details>
   <summary>Answer</summary>
   find . -name "*.log" -size +10M
   </details>

5. **What's safer: rm -rf or rm -ri?**
   <details>
   <summary>Answer</summary>
   rm -ri (interactive, asks for confirmation before deleting each file)
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Create test directory
mkdir ~/file-operations-practice
cd ~/file-operations-practice

# 2. Create some files
touch file1.txt file2.txt file3.txt
echo "Hello World" > hello.txt

# 3. List to verify
ls -lh

# 4. Create subdirectory
mkdir backup

# 5. Copy file to backup
cp hello.txt backup/

# 6. Verify copy
ls backup/

# 7. Rename file
mv file1.txt renamed-file.txt

# 8. List to see change
ls

# 9. Delete file2.txt
rm file2.txt

# 10. Verify deletion
ls

# 11. Create nested directories
mkdir -p projects/web/frontend

# 12. Find all .txt files
find . -name "*.txt"

# 13. Copy all .txt files to backup
cp *.txt backup/

# 14. Verify
ls -R

# 15. Clean up (be careful!)
cd ..
rm -r file-operations-practice/
```

---

## 📝 Key Takeaways

✅ **touch** - create empty files  
✅ **mkdir** - create directories (-p for nested)  
✅ **cp** - copy files (-r for directories)  
✅ **mv** - move/rename files  
✅ **rm** - delete files (-r for directories, BE CAREFUL!)  
✅ **find** - search files (powerful, many options)  
✅ **locate** - fast file search (uses database)  
✅ Always use **-i** flag for safety with rm, cp, mv  
✅ No recycle bin in Linux - deletion is permanent!  
✅ Use **-v** flag to see what commands are doing  

---

## 🚀 Next Steps

You can now create, copy, move, and delete files like a pro!

**Next lesson:** [05-file-permissions.md](05-file-permissions.md) - Understanding and managing file permissions

---

## 💡 Pro Tips

**Safety first:**
```bash
# Always verify before deleting
ls directory/
rm -ri directory/    # Not rm -rf!

# Create backups before major changes
cp -r important/ important.backup/
# Then make changes

# Use trash-cli instead of rm
trash file.txt       # Can restore later
```

**Efficiency tips:**
```bash
# Create and enter directory
mkdir project && cd project

# Copy with progress (for large files)
rsync -avh --progress source/ dest/

# Find recently modified files
find . -mtime -1 -type f

# Create dated backups
cp file.txt file-$(date +%Y%m%d).txt
```

**Remember:**
```
Tab completion works on filenames!
Use it to avoid typos and save time.

Example:
cp my-very-long-filename[Tab]
```

Master these file operations - you'll use them every day! 🚀
