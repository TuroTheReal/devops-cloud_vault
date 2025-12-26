# Linux Basics Commands - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, linux, cli, daily-use]
created: 2025-12-26
updated: 2025-12-26
```

**Related**: [[linux-security]], [[bash-scripting]]

---

## 📂 Files & Directories

### Navigation
```bash
# List files
ls                    # Basic list
ls -l                # Long format (permissions, owner, size, date)
ls -lh               # Human-readable sizes
ls -la               # Include hidden files
ls -lt               # Sort by modification time (newest first)
ls -lS               # Sort by size (largest first)

# Change directory
cd /path/to/dir      # Absolute path
cd ~/Documents       # Home directory
cd ..                # Parent directory
cd -                 # Previous directory
pwd                  # Print current directory

# Tree view
tree                 # Show directory tree
tree -L 2            # Limit depth to 2 levels
tree -a              # Include hidden files
```

### File Operations
```bash
# Copy
cp file.txt copy.txt              # Copy file
cp -r dir/ backup/                # Copy directory recursively
cp -i file.txt dest/              # Interactive (ask before overwrite)
cp -p file.txt dest/              # Preserve permissions/timestamps

# Move/Rename
mv old.txt new.txt                # Rename file
mv file.txt /path/to/dest/        # Move file
mv -i file.txt dest/              # Interactive

# Remove
rm file.txt                       # Remove file
rm -r dir/                        # Remove directory recursively
rm -rf dir/                       # IRREVERSIBLE : Force remove (no confirmation)
rm -i *.txt                       # Interactive remove

# Create
touch file.txt                    # Create empty file or update timestamp
mkdir newdir                      # Create directory
mkdir -p path/to/nested/dir       # Create nested directories
```

### File Info
```bash
# File size and disk usage
du -h file.txt                    # File size human-readable
du -sh dir/                       # Directory total size
du -h --max-depth=1               # Size of subdirectories (1 level)
df -h                             # Disk space usage (all partitions)
df -h /home                       # Disk space for /home

# File type
file document.pdf                 # Identify file type
stat file.txt                     # Detailed file info (size, perms, dates)

# Count
wc file.txt                       # Lines, words, bytes
wc -l file.txt                    # Count lines only
wc -w file.txt                    # Count words only
wc -c file.txt                    # Count bytes
```

---

## 🔍 Search & Find

### Find Files
```bash
# Find by name
find . -name "*.txt"              # Find .txt files in current dir
find /path -name "file.txt"       # Find specific file
find . -iname "*.TXT"             # Case-insensitive search

# Find by type
find . -type f                    # Find files only
find . -type d                    # Find directories only
find . -type l                    # Find symbolic links

# Find by size
find . -size +100M                # Files larger than 100MB
find . -size -1M                  # Files smaller than 1MB
find . -size 50M                  # Files exactly 50MB

# Find by time
find . -mtime -7                  # Modified in last 7 days
find . -mtime +30                 # Modified more than 30 days ago
find . -atime -1                  # Accessed in last 24h

# Find and execute
find . -name "*.log" -delete      # Find and delete
find . -name "*.txt" -exec cat {} \;  # Find and display content
```

### Locate (Fast Search)
```bash
# Update database first
sudo updatedb

# Search
locate file.txt                   # Fast search in database
locate -i FILE.TXT                # Case-insensitive
locate */bin/*python*             # Search with pattern
```

---

## 📝 Text Processing

### View Files
```bash
# Display content
cat file.txt                      # Display entire file
cat -n file.txt                   # With line numbers
tac file.txt                      # Reverse order (last line first)

# Paginate
less file.txt                     # View with navigation (q to quit)
more file.txt                     # Simple pager
head file.txt                     # First 10 lines
head -n 20 file.txt               # First 20 lines
tail file.txt                     # Last 10 lines
tail -n 50 file.txt               # Last 50 lines
tail -f logs.txt                  # Follow file (real-time)
```

### Search in Files (grep)
```bash
# Basic search
grep "pattern" file.txt           # Search pattern
grep -i "pattern" file.txt        # Case-insensitive
grep -r "pattern" dir/            # Recursive search in directory
grep -n "pattern" file.txt        # Show line numbers
grep -c "pattern" file.txt        # Count matches

# Advanced
grep -v "pattern" file.txt        # Invert match (exclude)
grep -w "word" file.txt           # Match whole word only
grep -A 3 "pattern" file.txt      # Show 3 lines after match
grep -B 3 "pattern" file.txt      # Show 3 lines before match
grep -C 3 "pattern" file.txt      # Show 3 lines around match

# Multiple patterns
grep "error\|warning" file.txt    # OR search
grep -E "error|warning" file.txt  # Extended regex
```

### Text Manipulation
```bash
# Sort
sort file.txt                     # Alphabetical sort
sort -r file.txt                  # Reverse sort
sort -n file.txt                  # Numeric sort
sort -u file.txt                  # Unique (remove duplicates)

# Unique
uniq file.txt                     # Remove adjacent duplicates
sort file.txt | uniq              # Remove all duplicates
uniq -c file.txt                  # Count occurrences

# Cut columns
cut -d':' -f1 /etc/passwd         # Extract 1st field (delimiter :)
cut -c1-10 file.txt               # Extract characters 1-10

# Replace (sed)
sed 's/old/new/' file.txt         # Replace first occurrence per line
sed 's/old/new/g' file.txt        # Replace all occurrences
sed -i 's/old/new/g' file.txt     # Edit file in-place

# Process (awk)
awk '{print $1}' file.txt         # Print 1st column
awk -F':' '{print $1}' /etc/passwd  # Print 1st field (delimiter :)
```

---

## 🔗 Pipes & Redirection

### Pipes (|)
```bash
# Chain commands
ls -l | grep ".txt"               # List only .txt files
cat file.txt | wc -l              # Count lines
ps aux | grep python              # Find python processes
history | tail -20                # Last 20 commands
```

### Redirection
```bash
# Output redirection
command > file.txt                # Overwrite file
command >> file.txt               # Append to file
command 2> errors.txt             # Redirect errors only
command &> output.txt             # Redirect stdout + stderr
command > /dev/null               # Discard output

# Input redirection
command < input.txt               # Read from file

# Here document
cat << EOF > file.txt
Line 1
Line 2
EOF
```

---

## ⚙️ Process Management

### View Processes
```bash
# List processes
ps                                # Current terminal processes
ps aux                            # All processes (detailed)
ps aux | grep nginx               # Find specific process
top                               # Interactive process viewer
htop                              # Better interactive viewer (install first)

# Process tree
pstree                            # Tree view of processes
pstree -p                         # Include PIDs
```

### Control Processes
```bash
# Kill processes
kill PID                          # Terminate process
kill -9 PID                       # Force kill
killall process_name              # Kill by name
pkill -f pattern                  # Kill by pattern

# Background/Foreground
command &                         # Run in background
Ctrl+Z                            # Suspend current process
bg                                # Resume in background
fg                                # Bring to foreground
jobs                              # List background jobs
```

---

## 🌐 Network

### Basic Network
```bash
# IP address
ip addr show                      # Show all interfaces
ip addr show eth0                 # Show specific interface
hostname -I                       # Show IP addresses

# Connectivity
ping google.com                   # Test connectivity
ping -c 4 8.8.8.8                 # Ping 4 times
curl ifconfig.me                  # Get public IP
curl -I https://example.com       # HTTP headers

# Download
wget https://example.com/file.zip # Download file
curl -O https://example.com/file  # Download file (curl)
curl -L url                       # Follow redirects
```

### Network Connections
```bash
# Active connections
ss -tuln                          # Listening ports (TCP/UDP)
ss -tulnp                         # With process names
netstat -tuln                     # Legacy (if ss not available)

# Test port
nc -zv host 80                    # Test if port 80 is open
telnet host 22                    # Test SSH port
```

---

## 🔐 Permissions

### View Permissions
```bash
ls -l file.txt
# Output: -rw-r--r-- 1 user group 1024 Dec 26 10:00 file.txt
#         │││││││││
#         ││└┬┘└┬┘└─ Others permissions (read)
#         │└─┴───┴─── Group permissions (read)
#         └────────── Owner permissions (read + write)
```

### Change Permissions
```bash
# Symbolic mode
chmod u+x script.sh               # Owner: add execute
chmod g+w file.txt                # Group: add write
chmod o-r file.txt                # Others: remove read
chmod a+r file.txt                # All: add read

# Numeric mode
chmod 644 file.txt                # rw-r--r--
chmod 755 script.sh               # rwxr-xr-x
chmod 600 secret.txt              # rw-------
chmod 777 file.txt                # rwxrwxrwx (avoid!)

# Recursive
chmod -R 755 dir/                 # Apply to directory and contents
```

### Change Owner
```bash
chown user file.txt               # Change owner
chown user:group file.txt         # Change owner and group
chown -R user:group dir/          # Recursive
```

---

## 📦 Archives & Compression

### Tar Archives
```bash
# Create
tar -czf archive.tar.gz dir/      # Create compressed archive
tar -czvf archive.tar.gz dir/     # Verbose output
tar -cjf archive.tar.bz2 dir/     # Bzip2 compression (better)

# Extract
tar -xzf archive.tar.gz           # Extract compressed
tar -xzvf archive.tar.gz          # Verbose
tar -xzf archive.tar.gz -C /dest  # Extract to directory

# List contents
tar -tzf archive.tar.gz           # List files in archive
```

### Zip
```bash
# Create
zip archive.zip file.txt          # Zip single file
zip -r archive.zip dir/           # Zip directory

# Extract
unzip archive.zip                 # Extract
unzip archive.zip -d /dest        # Extract to directory
unzip -l archive.zip              # List contents
```

---

## 💾 Disk & Storage

### Disk Usage
```bash
# Overall disk space
df -h                             # Human-readable
df -h /home                       # Specific mount point

# Directory sizes
du -h --max-depth=1               # Immediate subdirectories
du -sh *                          # Size of each item
du -sh dir/                       # Total size of directory

# Find large files
du -ah / | sort -rh | head -20    # 20 largest files/dirs
find / -type f -size +100M        # Files > 100MB
```

### Mount
```bash
# List mounts
mount                             # All mounted filesystems
df -h                             # Mounted partitions with usage

# Mount/Unmount
sudo mount /dev/sdb1 /mnt         # Mount partition
sudo umount /mnt                  # Unmount
```

---

## 🔧 System Info

### System Information
```bash
# OS info
uname -a                          # Kernel info
lsb_release -a                    # Distribution info (Ubuntu/Debian)
cat /etc/os-release               # OS details

# Hardware
lscpu                             # CPU info
free -h                           # RAM usage
lsblk                             # Block devices (disks)
```

### Date & Time
```bash
date                              # Current date/time
date "+%Y-%m-%d %H:%M:%S"         # Custom format
cal                               # Calendar
uptime                            # System uptime
```

---

## 📚 Quick Reference

### File Permissions Numeric
```
0 = --- (no permission)
1 = --x (execute)
2 = -w- (write)
3 = -wx (write + execute)
4 = r-- (read)
5 = r-x (read + execute)
6 = rw- (read + write)
7 = rwx (read + write + execute)

Example: 755 = rwxr-xr-x
  7 (owner)  = rwx
  5 (group)  = r-x
  5 (others) = r-x
```

### Common Keyboard Shortcuts
```
Ctrl+C    : Kill current process
Ctrl+Z    : Suspend current process
Ctrl+D    : EOF / Logout
Ctrl+L    : Clear screen
Ctrl+A    : Beginning of line
Ctrl+E    : End of line
Ctrl+R    : Reverse search history
Ctrl+U    : Clear line before cursor
Tab       : Auto-complete
!!        : Previous command
!$        : Last argument of previous command
```

---

## 🎯 Daily Workflow Examples

### Quick File Search
```bash
# Find all Python files modified today
find . -name "*.py" -mtime -1

# Search for text in all files
grep -r "TODO" .

# Find large log files
find /var/log -name "*.log" -size +100M
```

### System Cleanup
```bash
# Disk usage overview
df -h && echo "---" && du -sh /*

# Find and remove old logs
find /var/log -name "*.log" -mtime +30 -delete

# Clear package cache (Ubuntu/Debian)
sudo apt clean
```

### Quick Backup
```bash
# Backup directory
tar -czf backup-$(date +%Y%m%d).tar.gz ~/important/

# Sync directories
rsync -av source/ dest/
```

---

**Last updated**: 2025-12-26
**Difficulty**: ⭐ (1/5) - Beginner friendly