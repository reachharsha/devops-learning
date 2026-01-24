---
render_with_liquid: false
---
# 14 - Text Editors

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- When to use Nano vs Vim
- Nano basics (beginner-friendly)
- Vim basics (powerful but complex)
- Essential editing workflows
- Choosing the right editor for DevOps

---

## 📝 Text Editors in Linux

**Why you need a terminal text editor:**
```
DevOps Reality:
✅ SSH into remote servers (no GUI)
✅ Edit configuration files
✅ Fix issues quickly
✅ Modify scripts
✅ View logs

You MUST know at least one terminal editor!
```

**The Two Main Options:**

```
Nano:
✅ Easy to learn
✅ Shows shortcuts on screen
✅ Works like normal text editor
✅ Great for beginners
❌ Less powerful

Vim:
✅ Extremely powerful
✅ Very fast once learned
✅ Available everywhere
✅ Preferred by experts
❌ Steep learning curve
```

---

## 🟢 Nano - The Beginner's Choice

**Best for: Quick edits, beginners, simple tasks**

### **Opening Files**

```bash
# Open/create file
nano filename.txt

# Open at specific line
nano +25 filename.txt

# Open multiple files
nano file1.txt file2.txt

# Open with sudo (for system files)
sudo nano /etc/hosts
```

---

### **Basic Nano Usage**

**The screen:**
```
  GNU nano 6.2                filename.txt

This is the content of your file.
You can type here like a normal text editor.




^G Help      ^O Write Out  ^W Where Is   ^K Cut
^X Exit      ^R Read File  ^\ Replace    ^U Paste
^T Execute   ^J Justify    ^C Location   ^V Page Down
```

**Key symbols:**
```
^   = Ctrl key
M-  = Alt key (or Esc)

^G  = Ctrl+G
M-U = Alt+U (or Esc then U)
```

---

### **Essential Nano Commands**

**Navigation:**
```bash
Arrow keys      Move cursor
Ctrl+A          Start of line
Ctrl+E          End of line
Ctrl+Y          Page up
Ctrl+V          Page down

Alt+\           Start of file
Alt+/           End of file

Ctrl+_          Go to line number
```

**Editing:**
```bash
Ctrl+K          Cut line
Ctrl+U          Paste (uncut)
Ctrl+J          Justify paragraph
Ctrl+6          Mark start of selection
                (Move cursor to select)
Alt+6           Copy selection
Ctrl+K          Cut selection

Ctrl+W          Search
Alt+W           Search next
Ctrl+\          Search and replace
```

**File Operations:**
```bash
Ctrl+O          Save (Write Out)
                Press Enter to confirm

Ctrl+X          Exit
                If unsaved: prompts to save

Ctrl+R          Read file (insert file contents)
```

**Undo/Redo:**
```bash
Alt+U           Undo
Alt+E           Redo
```

---

### **Practical Nano Workflow**

```bash
# 1. Open file
nano config.txt

# 2. Edit content
(Type your changes)

# 3. Search for text
Ctrl+W
(Type search term)
Enter

# 4. Replace text
Ctrl+\
(Type search term)
Enter
(Type replacement)
Enter
A (Replace all) or Y (Replace) or N (Skip)

# 5. Go to specific line
Ctrl+_
(Type line number)
Enter

# 6. Save
Ctrl+O
Enter

# 7. Exit
Ctrl+X
```

---

### **Nano Configuration**

**Enable helpful features:**

```bash
# Edit nano config
nano ~/.nanorc

# Add these lines:
set linenumbers        # Show line numbers
set mouse              # Enable mouse support
set tabsize 4          # Tab = 4 spaces
set tabstospaces       # Convert tabs to spaces
set autoindent         # Auto indent new lines
set softwrap           # Wrap long lines
set backup             # Backup files

# Syntax highlighting (usually auto-enabled)
include "/usr/share/nano/*.nanorc"
```

---

## 🔵 Vim - The Power User's Choice

**Best for: Power users, complex editing, programming**

### **The Vim Philosophy**

```
Vim = "Vi IMproved"
Modal editor:
- Normal mode: Navigate and command
- Insert mode: Type text
- Visual mode: Select text
- Command mode: Execute commands

Different from normal editors!
Learning curve is real, but worth it.
```

---

### **Opening Files**

```bash
# Open file
vim filename.txt

# Open at specific line
vim +25 filename.txt

# Open multiple files
vim file1.txt file2.txt file3.txt

# Open with sudo
sudo vim /etc/hosts

# Read-only mode
vim -R filename.txt
view filename.txt
```

---

### **Vim Modes**

```
Normal Mode (default):
- Navigate
- Delete/copy/paste
- Enter commands
- Press Esc to return here

Insert Mode:
- Type text like normal editor
- Press i, a, o to enter
- Press Esc to exit

Visual Mode:
- Select text
- Press v to enter
- Press Esc to exit

Command Mode:
- Execute commands
- Type : from Normal mode
- Like :w (save), :q (quit)
```

---

### **Essential Vim Commands**

**Starting to Type:**
```
i       Insert at cursor
a       Append after cursor
I       Insert at start of line
A       Append at end of line
o       New line below
O       New line above

(Press Esc to return to Normal mode)
```

**Navigation (Normal Mode):**
```
h       Left
j       Down
k       Up
l       Right

0       Start of line
$       End of line
gg      Start of file
G       End of file

w       Next word
b       Previous word

Ctrl+F  Page down
Ctrl+B  Page up

:123    Go to line 123
```

**Editing (Normal Mode):**
```
x       Delete character
dd      Delete line
D       Delete to end of line
dw      Delete word

yy      Copy (yank) line
yw      Copy word
p       Paste after cursor
P       Paste before cursor

u       Undo
Ctrl+R  Redo

.       Repeat last command
```

**Searching:**
```
/pattern    Search forward
?pattern    Search backward
n           Next match
N           Previous match

*           Search for word under cursor
```

**Saving & Quitting:**
```
:w          Save
:q          Quit
:wq         Save and quit
:q!         Quit without saving

ZZ          Save and quit (shortcut)
ZQ          Quit without saving (shortcut)

:w newname  Save as newname
:e file     Open another file
```

---

### **Practical Vim Workflow**

```bash
# 1. Open file
vim config.txt

# 2. Enter insert mode
i

# 3. Type your changes
(Type normally)

# 4. Return to normal mode
Esc

# 5. Search
/searchterm
Enter
n (next match)

# 6. Delete a line
dd

# 7. Copy and paste
yy (copy line)
p (paste)

# 8. Save and quit
:wq
Enter
```

---

### **Common Vim Tasks**

**Replace text:**
```
:%s/old/new/g          Replace all occurrences
:%s/old/new/gc         Replace with confirmation
:s/old/new/            Replace in current line
:5,10s/old/new/g       Replace in lines 5-10
```

**Delete operations:**
```
dd              Delete current line
5dd             Delete 5 lines
d$              Delete to end of line
dgg             Delete to start of file
dG              Delete to end of file
:5,10d          Delete lines 5-10
```

**Copy/paste:**
```
yy              Copy current line
5yy             Copy 5 lines
p               Paste after cursor
P               Paste before cursor
```

**Visual mode selection:**
```
v               Start visual selection
V               Select whole lines
Ctrl+V          Block selection
y               Copy selection
d               Delete selection
```

**Multiple files:**
```
:e file2.txt    Open another file
:bnext          Next buffer
:bprev          Previous buffer
:ls             List buffers
:b2             Go to buffer 2
:bd             Close buffer
```

---

### **Vim Configuration**

```bash
# Create/edit vimrc
vim ~/.vimrc

# Add these helpful settings:
set number              " Show line numbers
set relativenumber      " Relative line numbers
set tabstop=4           " Tab = 4 spaces
set shiftwidth=4        " Indent = 4 spaces
set expandtab           " Tabs to spaces
set autoindent          " Auto indent
set smartindent         " Smart indent
set mouse=a             " Enable mouse
set hlsearch            " Highlight search
set incsearch           " Incremental search
set ignorecase          " Case-insensitive search
set smartcase           " Smart case search
syntax on               " Syntax highlighting
set background=dark     " Dark background
colorscheme desert      " Color scheme

# Save and reload:
:source ~/.vimrc
```

---

## 🎯 Choosing Your Editor

### **Use Nano when:**
```
✅ Quick one-time edit
✅ Simple configuration change
✅ You're a beginner
✅ Need to teach someone quickly
✅ Editing as non-tech user

Examples:
sudo nano /etc/hosts
nano README.md
nano .bashrc
```

### **Use Vim when:**
```
✅ Complex editing tasks
✅ Programming/scripting
✅ Multiple file editing
✅ Need powerful find/replace
✅ Speed matters (once learned)
✅ On servers without nano

Examples:
vim script.py
vim /etc/nginx/nginx.conf
vim multiple files at once
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you save and exit Nano?**
   <details>
   <summary>Answer</summary>
   Ctrl+O (save), Enter, Ctrl+X (exit)
   </details>

2. **How do you save and exit Vim?**
   <details>
   <summary>Answer</summary>
   Esc (normal mode), :wq, Enter
   </details>

3. **How do you enter insert mode in Vim?**
   <details>
   <summary>Answer</summary>
   Press i (or a, o, I, A, O)
   </details>

4. **How do you search in Nano?**
   <details>
   <summary>Answer</summary>
   Ctrl+W, type search term, Enter
   </details>

5. **How do you undo in Vim?**
   <details>
   <summary>Answer</summary>
   Press u in normal mode
   </details>

---

## 🏋️ Hands-On Exercise

### **Nano Exercise:**

```bash
# 1. Create file
nano practice.txt

# 2. Type some text
Line 1
Line 2
Line 3

# 3. Save (Ctrl+O, Enter)

# 4. Search for "Line 2" (Ctrl+W)

# 5. Replace "Line" with "Row" (Ctrl+\)

# 6. Save and exit (Ctrl+X)

# 7. View result
cat practice.txt
```

### **Vim Exercise:**

```bash
# 1. Create file
vim practice-vim.txt

# 2. Enter insert mode (i)

# 3. Type some text
This is line 1
This is line 2
This is line 3

# 4. Exit insert mode (Esc)

# 5. Go to top (gg)

# 6. Delete first line (dd)

# 7. Copy second line (yy)

# 8. Paste (p)

# 9. Save and quit (:wq)

# 10. View result
cat practice-vim.txt
```

---

## 📝 Key Takeaways

**Nano:**
✅ **Ctrl+O** - save  
✅ **Ctrl+X** - exit  
✅ **Ctrl+W** - search  
✅ **Ctrl+K** - cut line  
✅ **Ctrl+U** - paste  
✅ **Ctrl+_** - go to line  

**Vim:**
✅ **i** - insert mode  
✅ **Esc** - normal mode  
✅ **:w** - save  
✅ **:q** - quit  
✅ **:wq** - save and quit  
✅ **dd** - delete line  
✅ **yy** - copy line  
✅ **p** - paste  
✅ **/text** - search  

---

## 🚀 Next Steps

You can now edit files on any Linux system!

**Next lesson:** [15-shell-scripting.md](15-shell-scripting.md) - Automating tasks with bash scripts

---

## 💡 Pro Tips

**Nano shortcuts:**
```bash
# Quick config edit
alias editconf='nano ~/.bashrc'

# Always show line numbers
alias nano='nano -l'

# Backup before editing
cp file.txt file.txt.bak && nano file.txt
```

**Vim survival:**
```bash
# If stuck in Vim:
Esc Esc Esc :q! Enter

# Set Nano as default editor
export EDITOR=nano
export VISUAL=nano

# Or set Vim
export EDITOR=vim
export VISUAL=vim
```

**Practice Vim:**
```bash
# Built-in tutorial
vimtutor

# Practice game
vim-adventures.com
```

**Remember:**
```
Nano: Easy to start
Vim: Hard to start, powerful to master

Learn Nano first
Learn Vim eventually
Both are valuable!
```

Master one editor = Essential Linux skill! 🚀
