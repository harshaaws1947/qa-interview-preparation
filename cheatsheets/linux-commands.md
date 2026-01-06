# Linux Commands Cheatsheet

## File & Directory Operations

### Navigation
```bash
pwd                     # Print working directory
cd /path/to/dir         # Change directory
cd ~                    # Go to home directory
cd -                    # Go to previous directory
cd ..                   # Go up one level
```

### Listing Files
```bash
ls                      # List files
ls -l                   # Long format with details
ls -a                   # Include hidden files
ls -la                  # Long format + hidden
ls -lh                  # Human readable sizes
ls -lt                  # Sort by time (newest first)
ls -lS                  # Sort by size
ls -R                   # Recursive listing
```

### File Operations
```bash
# Create
touch file.txt          # Create empty file
mkdir directory         # Create directory
mkdir -p dir1/dir2      # Create nested directories

# Copy
cp file1 file2          # Copy file
cp -r dir1 dir2         # Copy directory recursively
cp -i file1 file2       # Interactive (confirm overwrite)

# Move/Rename
mv file1 file2          # Rename file
mv file1 /path/         # Move file
mv -i file1 file2       # Interactive

# Delete
rm file                 # Remove file
rm -r directory         # Remove directory recursively
rm -rf directory        # Force remove (dangerous!)
rm -i file              # Interactive deletion
rmdir directory         # Remove empty directory

# View file content
cat file                # Display entire file
less file               # Paginated view
more file               # Paginated view
head file               # First 10 lines
head -n 20 file         # First 20 lines
tail file               # Last 10 lines
tail -n 20 file         # Last 20 lines
tail -f file            # Follow file (live updates)
```

### File Information
```bash
file filename           # Determine file type
stat filename           # Detailed file info
wc file                 # Word, line, character count
wc -l file              # Line count only
du -h file              # File size
du -sh directory        # Directory size
```

## Text Processing

### Searching
```bash
# grep
grep "pattern" file           # Search in file
grep -i "pattern" file        # Case insensitive
grep -r "pattern" directory   # Recursive search
grep -n "pattern" file        # Show line numbers
grep -v "pattern" file        # Invert match (exclude)
grep -c "pattern" file        # Count matches
grep -l "pattern" *.txt       # List files with matches
grep -E "regex" file          # Extended regex

# find
find /path -name "*.txt"      # Find by name
find /path -type f            # Find files only
find /path -type d            # Find directories only
find /path -mtime -7          # Modified in last 7 days
find /path -size +100M        # Larger than 100MB
find /path -name "*.log" -delete   # Find and delete
```

### Text Manipulation
```bash
# sed (stream editor)
sed 's/old/new/' file         # Replace first occurrence
sed 's/old/new/g' file        # Replace all occurrences
sed -i 's/old/new/g' file     # In-place replacement
sed -n '5,10p' file           # Print lines 5-10
sed '/pattern/d' file         # Delete lines matching pattern

# awk
awk '{print $1}' file         # Print first column
awk -F: '{print $1}' file     # Custom delimiter
awk '/pattern/ {print}' file  # Print matching lines
awk '{sum+=$1} END {print sum}' file   # Sum first column

# cut
cut -d: -f1 file              # Cut first field (delimiter :)
cut -c1-10 file               # Cut characters 1-10

# sort
sort file                     # Sort alphabetically
sort -n file                  # Sort numerically
sort -r file                  # Reverse sort
sort -u file                  # Sort and remove duplicates
sort -k2 file                 # Sort by second column

# uniq
uniq file                     # Remove adjacent duplicates
uniq -c file                  # Count duplicates
sort file | uniq              # Remove all duplicates
```

## Process Management

```bash
# View processes
ps                      # Current user processes
ps aux                  # All processes
ps aux | grep java      # Find Java processes
top                     # Real-time process viewer
htop                    # Interactive process viewer

# Process control
kill PID                # Terminate process
kill -9 PID             # Force kill
killall processname     # Kill by name
pkill processname       # Kill by pattern

# Background processes
command &               # Run in background
jobs                    # List background jobs
fg %1                   # Bring job 1 to foreground
bg %1                   # Resume job 1 in background
nohup command &         # Run immune to hangups

# Resource usage
free -h                 # Memory usage
df -h                   # Disk usage
du -sh *                # Directory sizes
```

## Permissions

### Understanding Permissions
```
-rwxrwxrwx = file type + owner + group + others
r = read (4)
w = write (2)
x = execute (1)
```

### Changing Permissions
```bash
chmod 755 file          # rwxr-xr-x
chmod 644 file          # rw-r--r--
chmod +x file           # Add execute
chmod -w file           # Remove write
chmod u+x file          # Add execute for owner
chmod g-w file          # Remove write for group
chmod o=r file          # Set others to read only

# Recursive
chmod -R 755 directory

# Ownership
chown user file         # Change owner
chown user:group file   # Change owner and group
chown -R user directory # Recursive
```

## Networking

```bash
# Network info
ifconfig                # Network interfaces
ip addr                 # Network interfaces (newer)
hostname                # Display hostname
hostname -I             # Display IP addresses

# Connectivity
ping hostname           # Test connectivity
ping -c 5 hostname      # Ping 5 times
traceroute hostname     # Trace route

# Ports
netstat -tulpn          # List listening ports
ss -tulpn               # List listening ports (newer)
lsof -i :8080           # What's using port 8080

# DNS
nslookup domain         # DNS lookup
dig domain              # DNS lookup (detailed)
host domain             # DNS lookup

# Downloads
curl url                # Fetch URL content
curl -O url             # Download file
curl -I url             # Headers only
wget url                # Download file
wget -r url             # Recursive download
```

## SSH & Remote

```bash
# Connect
ssh user@hostname       # SSH connect
ssh -p 2222 user@host   # Custom port
ssh -i key.pem user@host # Using key file

# File transfer
scp file user@host:/path/     # Copy to remote
scp user@host:/path/file .    # Copy from remote
scp -r dir user@host:/path/   # Copy directory

# SSH keys
ssh-keygen -t rsa -b 4096    # Generate key pair
ssh-copy-id user@host        # Copy public key to server
```

## Compression

```bash
# tar
tar -cvf archive.tar files    # Create tar
tar -xvf archive.tar          # Extract tar
tar -tvf archive.tar          # List contents
tar -czvf archive.tar.gz dir  # Create gzipped tar
tar -xzvf archive.tar.gz      # Extract gzipped tar

# zip
zip archive.zip files         # Create zip
zip -r archive.zip directory  # Recursive zip
unzip archive.zip             # Extract zip
unzip -l archive.zip          # List contents

# gzip
gzip file                     # Compress (replaces original)
gunzip file.gz                # Decompress
gzip -k file                  # Keep original
```

## Environment & Variables

```bash
# Variables
echo $PATH              # Print PATH
echo $HOME              # Print home directory
env                     # All environment variables
export VAR=value        # Set variable
unset VAR               # Remove variable

# PATH
export PATH=$PATH:/new/path   # Add to PATH

# Shell config
source ~/.bashrc        # Reload config
. ~/.bashrc             # Same as source
```

## System Information

```bash
uname -a                # System info
cat /etc/os-release     # OS info
uptime                  # System uptime
whoami                  # Current user
id                      # User and group IDs
last                    # Login history
history                 # Command history
```

## Package Management

### Debian/Ubuntu (apt)
```bash
apt update              # Update package list
apt upgrade             # Upgrade packages
apt install package     # Install package
apt remove package      # Remove package
apt search keyword      # Search packages
apt list --installed    # List installed
```

### RedHat/CentOS (yum/dnf)
```bash
yum update              # Update packages
yum install package     # Install package
yum remove package      # Remove package
yum search keyword      # Search packages
```

## Useful One-Liners

```bash
# Find largest files
find / -type f -size +100M 2>/dev/null | head -20

# Count files in directory
find . -type f | wc -l

# Find and replace in multiple files
find . -name "*.txt" -exec sed -i 's/old/new/g' {} \;

# List directories by size
du -h --max-depth=1 | sort -hr

# Find process using port
lsof -i :8080 | grep LISTEN

# Watch log file
tail -f /var/log/syslog

# Create backup with date
tar -czvf backup_$(date +%Y%m%d).tar.gz directory

# Kill process by name
pkill -f "process_name"

# Check disk usage percentage
df -h | awk '$5 > 80 {print $0}'
```

## Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + C` | Kill current command |
| `Ctrl + Z` | Suspend current command |
| `Ctrl + D` | Exit shell |
| `Ctrl + L` | Clear screen |
| `Ctrl + A` | Go to start of line |
| `Ctrl + E` | Go to end of line |
| `Ctrl + R` | Search command history |
| `Tab` | Auto-complete |
| `!!` | Repeat last command |
| `!$` | Last argument of previous command |
