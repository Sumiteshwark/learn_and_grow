# Linux CLI and Shell Scripting - Complete Guide

## Table of Contents
1. [Introduction to Linux](#introduction-to-linux)
2. [File System Navigation](#file-system-navigation)
3. [File Operations](#file-operations)
4. [Text Processing](#text-processing)
5. [Process Management](#process-management)
6. [User and Permission Management](#user-and-permission-management)
7. [Package Management](#package-management)
8. [Network Tools](#network-tools)
9. [Shell Scripting Fundamentals](#shell-scripting-fundamentals)
10. [Advanced Shell Scripting](#advanced-shell-scripting)
11. [System Administration](#system-administration)
12. [Troubleshooting](#troubleshooting)

---

## Introduction to Linux

### What is Linux?
Linux is a free and open-source operating system kernel that serves as the foundation for various Linux distributions (distros). It's widely used in servers, cloud computing, and containerized environments.

### Key Linux Distributions
- **Ubuntu**: User-friendly, great for beginners
- **CentOS/RHEL**: Enterprise-focused, stable
- **Debian**: Community-driven, very stable
- **Alpine**: Minimal, perfect for containers
- **Amazon Linux**: Optimized for AWS

### Linux Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    User Applications                        │
├─────────────────────────────────────────────────────────────┤
│                    Shell (Bash, Zsh, etc.)                  │
├─────────────────────────────────────────────────────────────┤
│                    System Libraries                         │
├─────────────────────────────────────────────────────────────┤
│                    Kernel                                   │
├─────────────────────────────────────────────────────────────┤
│                    Hardware                                 │
└─────────────────────────────────────────────────────────────┘
```

### Basic Commands
```bash
# Check system information
uname -a
cat /etc/os-release

# Check current user
whoami

# Check current directory
pwd

# List files and directories
ls
ls -la

# Change directory
cd /path/to/directory
cd ~  # Home directory
cd -  # Previous directory

# Clear screen
clear
```

---

## File System Navigation

### Linux File System Structure
```
/
├── bin/          # Essential binaries
├── boot/         # Boot loader files
├── dev/          # Device files
├── etc/          # Configuration files
├── home/         # User home directories
├── lib/          # Shared libraries
├── media/        # Mount points for removable media
├── mnt/          # Mount points
├── opt/          # Optional applications
├── proc/         # Process information
├── root/         # Root user home
├── run/          # Runtime data
├── sbin/         # System binaries
├── srv/          # Service data
├── sys/          # System information
├── tmp/          # Temporary files
├── usr/          # User programs and data
└── var/          # Variable data
```

### Navigation Commands
```bash
# List directory contents
ls
ls -l          # Long format
ls -a          # Show hidden files
ls -la         # Long format with hidden files
ls -lh         # Human-readable sizes
ls -lt         # Sort by modification time
ls -ltr        # Sort by modification time (reverse)

# Change directory
cd /absolute/path
cd relative/path
cd ~           # Home directory
cd ~username   # Specific user's home
cd -           # Previous directory
cd ..          # Parent directory

# Show current directory
pwd

# Create directories
mkdir directory_name
mkdir -p parent/child/grandchild  # Create parent directories

# Remove directories
rmdir directory_name              # Remove empty directory
rm -rf directory_name            # Remove directory and contents (dangerous!)
```

### File System Information
```bash
# Check disk usage
df -h

# Check directory size
du -sh /path/to/directory

# Check inode usage
df -i

# Mount points
mount
cat /proc/mounts
```

---

## File Operations

### File Creation and Editing
```bash
# Create empty file
touch filename

# Create file with content
echo "content" > filename
echo "content" >> filename  # Append

# Copy files
cp source destination
cp -r source_dir destination_dir  # Recursive copy
cp -v source destination          # Verbose

# Move/rename files
mv old_name new_name
mv file /path/to/destination/

# Remove files
rm filename
rm -f filename                    # Force (no confirmation)
rm -i filename                    # Interactive (confirmation)
rm -rf directory                  # Recursive and force (dangerous!)
```

### File Viewing and Searching
```bash
# View file content
cat filename
cat -n filename                   # With line numbers
cat -A filename                   # Show non-printing characters

# View file page by page
less filename
more filename

# View beginning of file
head filename
head -n 20 filename              # First 20 lines

# View end of file
tail filename
tail -n 20 filename              # Last 20 lines
tail -f filename                 # Follow (watch for changes)

# Search in files
grep "pattern" filename
grep -i "pattern" filename       # Case insensitive
grep -r "pattern" directory      # Recursive search
grep -v "pattern" filename       # Invert match
grep -n "pattern" filename       # Show line numbers
```

### File Permissions and Ownership
```bash
# Check file permissions
ls -l filename

# Change permissions
chmod 755 filename               # Numeric mode
chmod u+rwx,g+rx,o+rx filename  # Symbolic mode
chmod +x filename                # Make executable

# Change ownership
chown user:group filename
chown user filename
chgrp group filename

# Change permissions recursively
chmod -R 755 directory/
chown -R user:group directory/
```

### File Types and Links
```bash
# Check file type
file filename

# Create symbolic link
ln -s target link_name

# Create hard link
ln target link_name

# Find files
find /path -name "pattern"
find /path -type f -name "*.txt"
find /path -mtime -7            # Modified in last 7 days
find /path -size +100M          # Larger than 100MB
find /path -exec rm {} \;       # Execute command on found files
```

---

## Text Processing

### Text Editors
```bash
# Nano (beginner-friendly)
nano filename

# Vim (advanced)
vim filename
# Basic vim commands:
# i - insert mode
# Esc - command mode
# :w - save
# :q - quit
# :wq - save and quit
# :q! - quit without saving

# Sed (stream editor)
sed 's/old/new/g' filename      # Replace text
sed -i 's/old/new/g' filename   # Edit file in place
sed -n '5,10p' filename         # Print lines 5-10
```

### Text Processing Tools
```bash
# Cut (extract columns)
cut -d: -f1 /etc/passwd         # Extract first field
cut -c1-10 filename             # Extract characters 1-10

# Awk (pattern scanning and processing)
awk '{print $1}' filename       # Print first field
awk -F: '{print $1}' /etc/passwd # Use colon as delimiter
awk '$3 > 1000 {print $1}' /etc/passwd # Conditional printing

# Sort
sort filename
sort -n filename                # Numeric sort
sort -r filename                # Reverse sort
sort -u filename                # Unique lines

# Uniq
uniq filename                   # Remove consecutive duplicates
sort filename | uniq            # Remove all duplicates

# Wc (word count)
wc filename                      # Lines, words, characters
wc -l filename                  # Line count only
wc -w filename                  # Word count only
wc -c filename                  # Character count only
```

### Advanced Text Processing
```bash
# Combine commands with pipes
cat filename | grep "pattern" | sort | uniq

# Process multiple files
for file in *.txt; do
    echo "Processing $file"
    grep "pattern" "$file"
done

# Extract specific lines
sed -n '10,20p' filename        # Lines 10-20
head -20 filename | tail -10    # Lines 11-20

# Replace text across multiple files
find . -name "*.txt" -exec sed -i 's/old/new/g' {} \;
```

---

## Process Management

### Process Basics
```bash
# List processes
ps
ps aux                          # All processes with details
ps -ef                          # Alternative format
ps aux | grep process_name      # Find specific process

# Process tree
pstree
pstree -p                       # Show PIDs

# Real-time process monitoring
top
htop                           # Enhanced top (if installed)

# Process information
ps -p PID -o pid,ppid,cmd      # Specific process details
```

### Process Control
```bash
# Start process in background
command &

# Start process and detach
nohup command &

# Send signals to processes
kill PID                        # Send TERM signal
kill -9 PID                     # Send KILL signal
kill -HUP PID                   # Send HUP signal
killall process_name            # Kill all processes with name

# Suspend and resume
Ctrl+Z                          # Suspend process
fg                              # Resume in foreground
bg                              # Resume in background

# Job control
jobs                            # List background jobs
fg %1                           # Bring job 1 to foreground
bg %1                           # Send job 1 to background
```

### System Monitoring
```bash
# System load
uptime
w

# Memory usage
free -h
cat /proc/meminfo

# CPU information
cat /proc/cpuinfo
nproc                           # Number of processors

# Disk I/O
iostat
iotop                          # Real-time I/O monitoring

# Network connections
netstat -tuln
ss -tuln                       # Modern alternative to netstat
lsof -i                        # List open network connections
```

---

## User and Permission Management

### User Management
```bash
# List users
cat /etc/passwd
getent passwd

# Add user
useradd username
useradd -m -s /bin/bash username  # With home directory and shell

# Delete user
userdel username
userdel -r username              # Remove home directory too

# Change password
passwd username

# Switch user
su username
su - username                   # Switch with environment

# Sudo access
sudo command
sudo -i                         # Root shell
sudo -u username command        # Run as specific user
```

### Group Management
```bash
# List groups
cat /etc/group
getent group

# Add group
groupadd groupname

# Delete group
groupdel groupname

# Add user to group
usermod -a -G groupname username

# Check user groups
groups username
id username
```

### File Permissions
```bash
# Permission types
# r (read) = 4
# w (write) = 2
# x (execute) = 1

# Permission categories
# u (user/owner)
# g (group)
# o (others)
# a (all)

# Common permission examples
chmod 755 file                  # rwxr-xr-x
chmod 644 file                  # rw-r--r--
chmod 600 file                  # rw-------
chmod 777 file                  # rwxrwxrwx (dangerous!)

# Symbolic permissions
chmod u+x file                  # Add execute for user
chmod g-w file                  # Remove write for group
chmod o=r file                  # Set read-only for others
chmod a+x file                  # Add execute for all
```

### Special Permissions
```bash
# Setuid (run as owner)
chmod u+s file                  # 4000

# Setgid (run as group)
chmod g+s file                  # 2000

# Sticky bit (only owner can delete)
chmod +t directory              # 1000

# Examples
chmod 4755 executable           # Setuid executable
chmod 2755 directory            # Setgid directory
chmod 1755 /tmp                 # Sticky bit on /tmp
```

---

## Package Management

### Debian/Ubuntu (apt)
```bash
# Update package list
sudo apt update

# Upgrade packages
sudo apt upgrade
sudo apt dist-upgrade           # Smart upgrade

# Install package
sudo apt install package_name

# Remove package
sudo apt remove package_name
sudo apt purge package_name     # Remove configuration too

# Search packages
apt search package_name
apt list --installed

# Show package information
apt show package_name

# Clean up
sudo apt autoremove            # Remove unused packages
sudo apt autoclean             # Clean package cache
```

### Red Hat/CentOS (yum/dnf)
```bash
# Update system
sudo yum update
sudo dnf update                # Modern alternative

# Install package
sudo yum install package_name
sudo dnf install package_name

# Remove package
sudo yum remove package_name
sudo dnf remove package_name

# Search packages
yum search package_name
dnf search package_name

# List installed packages
yum list installed
dnf list installed

# Clean up
sudo yum clean all
sudo dnf clean all
```

### Alpine (apk)
```bash
# Update package list
apk update

# Install package
apk add package_name

# Remove package
apk del package_name

# Search packages
apk search package_name

# List installed packages
apk info

# Clean up
apk cache clean
```

### Snap Packages
```bash
# Install snap
sudo apt install snapd

# Install package
sudo snap install package_name

# Remove package
sudo snap remove package_name

# List installed snaps
snap list

# Update snaps
sudo snap refresh
```

---

## Network Tools

### Network Configuration
```bash
# Network interfaces
ip addr show
ifconfig                       # Legacy command

# Network routes
ip route show
route -n                       # Legacy command

# DNS configuration
cat /etc/resolv.conf
systemd-resolve --status

# Network connectivity
ping hostname
ping -c 4 hostname             # 4 packets only

# Traceroute
traceroute hostname
mtr hostname                   # Real-time traceroute
```

### Network Services
```bash
# Check listening ports
netstat -tuln
ss -tuln
lsof -i

# Check specific port
netstat -tuln | grep :80
ss -tuln | grep :80

# Test port connectivity
telnet hostname port
nc -zv hostname port           # Netcat
nmap hostname                  # Port scanning
```

### Network Troubleshooting
```bash
# Check DNS resolution
nslookup hostname
dig hostname
host hostname

# Check HTTP connectivity
curl -I http://hostname
wget --spider http://hostname

# Network speed test
speedtest-cli

# Check network interface statistics
cat /proc/net/dev
ip -s link show
```

---

## Shell Scripting Fundamentals

### Shell Types
```bash
# Common shells
/bin/bash                      # Bourne Again Shell
/bin/sh                        # Bourne Shell
/bin/zsh                       # Z Shell
/bin/fish                      # Friendly Interactive Shell

# Check current shell
echo $SHELL
ps -p $$

# Change shell
chsh -s /bin/bash username
```

### Basic Script Structure
```bash
#!/bin/bash
# This is a comment
# Script description

# Variables
NAME="World"

# Main script
echo "Hello, $NAME!"
```

### Variables
```bash
# Variable assignment
NAME="John"
AGE=25

# Variable usage
echo $NAME
echo ${NAME}

# Environment variables
echo $HOME
echo $PATH
echo $USER

# Command substitution
DATE=$(date)
DATE=`date`                    # Legacy syntax

# Export variables
export NAME="John"
export PATH=$PATH:/usr/local/bin
```

### Input and Output
```bash
# Read input
read name
read -p "Enter your name: " name
read -s password               # Silent input

# Output
echo "Hello, World!"
echo -e "Hello\nWorld!"        # Interpret escape sequences
printf "Hello, %s!\n" "World"

# Redirection
echo "Hello" > file.txt        # Overwrite
echo "World" >> file.txt       # Append
command < input.txt            # Input redirection
command 2> error.log           # Error redirection
command > output.log 2>&1      # Both output and error
```

### Control Structures

#### Conditional Statements
```bash
# If statement
if [ condition ]; then
    commands
elif [ condition ]; then
    commands
else
    commands
fi

# Test conditions
[ -f file ]                    # File exists
[ -d directory ]               # Directory exists
[ -r file ]                    # File is readable
[ -w file ]                    # File is writable
[ -x file ]                    # File is executable
[ -z string ]                  # String is empty
[ -n string ]                  # String is not empty
[ string1 = string2 ]          # Strings are equal
[ num1 -eq num2 ]              # Numbers are equal
[ num1 -lt num2 ]              # Number1 less than number2
```

#### Loops
```bash
# For loop
for i in 1 2 3 4 5; do
    echo $i
done

for file in *.txt; do
    echo "Processing $file"
done

for ((i=1; i<=5; i++)); do
    echo $i
done

# While loop
while [ condition ]; do
    commands
done

# Until loop
until [ condition ]; do
    commands
done
```

### Functions
```bash
# Function definition
function_name() {
    local var="local variable"  # Local variable
    echo "Function called with: $1"
    return 0
}

# Function call
function_name "argument"

# Function with return value
get_date() {
    date +%Y-%m-%d
}

today=$(get_date)
echo "Today is: $today"
```

---

## Advanced Shell Scripting

### Error Handling
```bash
#!/bin/bash

# Exit on error
set -e

# Exit on undefined variable
set -u

# Show commands being executed
set -x

# Function to handle errors
error_handler() {
    echo "Error occurred in line $1"
    exit 1
}

trap 'error_handler $LINENO' ERR
```

### Logging
```bash
#!/bin/bash

# Logging function
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a /var/log/script.log
}

log "Script started"
log "Processing file: $1"
log "Script completed"
```

### Configuration Files
```bash
#!/bin/bash

# Read configuration file
CONFIG_FILE="config.conf"

if [ -f "$CONFIG_FILE" ]; then
    source "$CONFIG_FILE"
else
    echo "Configuration file not found"
    exit 1
fi

# Use configuration variables
echo "Database host: $DB_HOST"
echo "Database port: $DB_PORT"
```

### Command Line Arguments
```bash
#!/bin/bash

# Parse command line arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            echo "Usage: $0 [OPTIONS]"
            echo "Options:"
            echo "  -h, --help     Show this help message"
            echo "  -v, --verbose  Verbose output"
            echo "  -f, --file     Input file"
            exit 0
            ;;
        -v|--verbose)
            VERBOSE=1
            shift
            ;;
        -f|--file)
            INPUT_FILE="$2"
            shift 2
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done
```

### Process Management in Scripts
```bash
#!/bin/bash

# Background process
start_service() {
    nohup service_command > /dev/null 2>&1 &
    echo $! > /var/run/service.pid
}

# Stop background process
stop_service() {
    if [ -f /var/run/service.pid ]; then
        kill $(cat /var/run/service.pid)
        rm /var/run/service.pid
    fi
}

# Check if process is running
is_running() {
    if [ -f /var/run/service.pid ]; then
        kill -0 $(cat /var/run/service.pid) 2>/dev/null
        return $?
    fi
    return 1
}
```

---

## System Administration

### System Information
```bash
# System information
uname -a
cat /etc/os-release
lsb_release -a

# Hardware information
lscpu
free -h
df -h
lspci
lsusb

# System uptime
uptime
who
w

# System load
cat /proc/loadavg
```

### Service Management
```bash
# Systemd services
systemctl status service_name
systemctl start service_name
systemctl stop service_name
systemctl restart service_name
systemctl enable service_name
systemctl disable service_name

# List all services
systemctl list-units --type=service
systemctl list-units --type=service --state=running

# Service logs
journalctl -u service_name
journalctl -u service_name -f
journalctl -u service_name --since "1 hour ago"
```

### System Monitoring
```bash
# Real-time monitoring
top
htop
iotop
nethogs

# System statistics
vmstat 1 10
iostat 1 10
sar -u 1 10

# Memory usage
cat /proc/meminfo
free -h
ps aux --sort=-%mem | head -10

# CPU usage
cat /proc/cpuinfo
ps aux --sort=-%cpu | head -10
```

### Backup and Recovery
```bash
# Create backup
tar -czf backup.tar.gz /path/to/backup

# Extract backup
tar -xzf backup.tar.gz

# Incremental backup with rsync
rsync -av --delete /source/ /backup/

# Database backup
mysqldump -u user -p database > backup.sql
pg_dump database > backup.sql

# System backup
dd if=/dev/sda of=/backup/system.img bs=4M
```

---

## Troubleshooting

### System Boot Issues
```bash
# Check boot logs
dmesg
journalctl -b
journalctl -b -1              # Previous boot

# Check systemd services
systemctl list-units --failed
systemctl --failed

# Check disk space
df -h
du -sh /*

# Check inodes
df -i
```

### Network Issues
```bash
# Check network interfaces
ip addr show
ip link show

# Check routing
ip route show
route -n

# Test connectivity
ping -c 4 8.8.8.8
ping -c 4 google.com

# Check DNS
nslookup google.com
dig google.com

# Check ports
netstat -tuln
ss -tuln
```

### Performance Issues
```bash
# Check system load
uptime
cat /proc/loadavg

# Check memory usage
free -h
cat /proc/meminfo

# Check disk I/O
iostat
iotop

# Check CPU usage
top
htop
mpstat

# Check network usage
iftop
nethogs
```

### Security Issues
```bash
# Check failed login attempts
grep "Failed password" /var/log/auth.log
journalctl -u ssh

# Check open ports
netstat -tuln
ss -tuln
nmap localhost

# Check file permissions
find / -perm -4000 -type f 2>/dev/null  # Setuid files
find / -perm -2000 -type f 2>/dev/null  # Setgid files

# Check for suspicious processes
ps aux | grep -E "(crypto|miner|backdoor)"
```

### Log Analysis
```bash
# System logs
tail -f /var/log/syslog
tail -f /var/log/messages

# Application logs
tail -f /var/log/nginx/access.log
tail -f /var/log/apache2/access.log

# Authentication logs
tail -f /var/log/auth.log

# Kernel logs
dmesg | tail -20

# Journal logs
journalctl -f
journalctl --since "1 hour ago"
```

---

## Best Practices

### Scripting Best Practices
1. **Always use shebang**: `#!/bin/bash`
2. **Add comments**: Explain what the script does
3. **Use meaningful variable names**: `USER_NAME` instead of `u`
4. **Quote variables**: `"$variable"` to handle spaces
5. **Check for errors**: Use `set -e` and error handling
6. **Use functions**: Organize code into reusable functions
7. **Add logging**: Track script execution
8. **Handle arguments**: Validate command line arguments
9. **Use configuration files**: Separate configuration from logic
10. **Test thoroughly**: Test with various inputs and conditions

### Security Best Practices
1. **Use strong passwords**: Complex passwords for all accounts
2. **Limit sudo access**: Only give necessary privileges
3. **Keep systems updated**: Regular security updates
4. **Use firewall**: Configure iptables or ufw
5. **Monitor logs**: Regular log analysis
6. **Backup regularly**: Automated backup systems
7. **Use SSH keys**: Instead of passwords for SSH
8. **Disable unnecessary services**: Reduce attack surface
9. **Use encrypted connections**: HTTPS, SSH, VPN
10. **Regular security audits**: Check for vulnerabilities

### Performance Best Practices
1. **Monitor system resources**: Regular monitoring
2. **Optimize disk usage**: Regular cleanup
3. **Use efficient commands**: Avoid unnecessary operations
4. **Cache frequently used data**: Reduce I/O operations
5. **Use appropriate file systems**: Choose based on use case
6. **Optimize network**: Use appropriate protocols
7. **Load balancing**: Distribute load across systems
8. **Resource limits**: Set appropriate limits
9. **Regular maintenance**: Clean up temporary files
10. **Capacity planning**: Plan for growth

---

## Next Steps

After mastering Linux CLI and shell scripting, you'll be ready to:

1. **Learn Nomad**: Use shell scripts for job definitions and automation
2. **Explore Consul**: Understand service discovery and health checks
3. **Integrate Vault**: Automate secret management with scripts
4. **Build Infrastructure**: Create automated deployment scripts

### Practice Projects

1. **System Monitoring Script**: Create a script that monitors system resources
2. **Backup Automation**: Automate backup procedures
3. **Log Analysis**: Create scripts to analyze log files
4. **Service Management**: Automate service deployment and management

### Resources

- [Bash Reference Manual](https://www.gnu.org/software/bash/manual/)
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)
- [Linux Documentation Project](https://tldp.org/)
- [Shell Scripting Tutorial](https://www.shellscript.sh/)
- [Bash Pitfalls](http://mywiki.wooledge.org/BashPitfalls) 