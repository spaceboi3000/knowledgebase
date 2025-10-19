# 🧠 Linux System Administration & DevOps Cheat Sheet

A complete, categorized reference for Linux power users, sysadmins, and embedded/DevOps engineers. Commands grouped by topic and alphabetized within each category. Includes service control, networking, kernel modules, storage, video devices, build tools, and more.

---

## 🏗️ 1. System Management

### journalctl — System Logs

```bash
journalctl -xe                             # Show recent logs with context
journalctl -u myservice                    # Logs for a specific service
journalctl -b                              # Logs from current boot
journalctl -b -1                           # Logs from previous boot
journalctl -f                              # Follow logs live
journalctl --since "1 hour ago"            # Show logs from last hour
journalctl -p err..alert                   # Show only errors and above
```

### systemctl — Service Management

```bash
systemctl start myservice                  # Start a service
systemctl stop myservice                   # Stop a service
systemctl restart myservice                # Restart a service
systemctl reload myservice                 # Reload configuration
systemctl enable myservice                 # Start at boot
systemctl disable myservice                # Disable autostart
systemctl status myservice                 # Show status
systemctl list-units --type=service        # List active services
systemctl daemon-reload                    # Reload configs after editing
```

### Systemd Unit (Basic Template)

**Location:** `/etc/systemd/system/myservice.service`

```ini
[Unit]
Description=My Service
After=network.target

[Service]
Type=simple
User=myuser
Environment=ENV=prod
ExecStart=/usr/local/bin/myservice --option
ExecStartPre=/usr/local/bin/precheck.sh
ExecStartPost=/usr/local/bin/poststart.sh
Restart=on-failure
RestartSec=5
WorkingDirectory=/var/lib/myservice
TimeoutStartSec=30
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

**Notes:**
- Type values: `simple` | `forking` | `oneshot` | `dbus` | `notify` | `idle`
- Use `ExecStartPre`/`ExecStartPost` for prepare/cleanup steps
- Use `EnvironmentFile=/etc/default/myservice` to load env vars from a file
- Put custom unit files in `/etc/systemd/system/` and run `systemctl daemon-reload`
- `systemctl enable --now myservice` enables and starts the service

---

## ⚙️ 2. Kernel & Module Management

### modprobe / lsmod / rmmod

```bash
sudo modprobe v4l2loopback                 # Load module
sudo modprobe -r v4l2loopback              # Unload module
sudo modprobe v4l2loopback devices=2 exclusive_caps=1
lsmod                                      # List loaded modules
lsmod | grep video                         # Filter modules
rmmod modulename                           # Force remove module
```

### dmesg — Kernel Messages

```bash
dmesg | tail                               # Recent kernel messages
dmesg -T                                   # Show timestamps
dmesg | grep usb                           # Filter messages
```

---

## 🎥 3. Video Devices (V4L2 & Loopback)

### v4l2-ctl

```bash
v4l2-ctl --list-devices                    # List cameras
v4l2-ctl -d /dev/video0 --list-formats-ext # Show formats
v4l2-ctl -d /dev/video0 --get-fmt-video    # Get current format
v4l2-ctl --set-fmt-video=width=1280,height=720,pixelformat=YUYV -d /dev/video0
v4l2-ctl --stream-mmap --stream-count=100 -d /dev/video0  # Test capture
```

### v4l2loopback / v4l2loopback-ctl

```bash
sudo apt install v4l2loopback-dkms         # Install DKMS module (Debian/Ubuntu)
sudo modprobe v4l2loopback video_nr=99 card_label="VirtualCam" exclusive_caps=1
v4l2loopback-ctl list                      # List loopback devices
v4l2loopback-ctl set-caps "YUYV 640x480@30" /dev/video99
ls /dev/video*                              # Confirm devices
```

---

## 🐚 4. Bash Scripting Basics & Syntax Rules

### Shebang and Execution

```bash
#!/bin/bash
chmod +x script.sh
./script.sh
```

### Quoting Rules

- Use single quotes `'...'` to prevent expansion of variables and most backslashes
- Use double quotes `"..."` to allow variable expansion but preserve whitespace
- Avoid unquoted variables: use `"$var"` to preserve spacing and avoid word-splitting

### Variables and Exports

```bash
VAR=value
export VAR
readonly VAR
unset VAR
```

### Command Substitution

```bash
DATE=$(date)
UPTIME=`uptime`     # older style; prefer $(...)
```

### Arithmetic

```bash
n=$(( 1 + 2 ))
(( n++ ))
let "n += 5"
```

### Tests (test / [ ] / [[ ]])

```bash
[ -f /etc/passwd ]        # file exists (regular file)
[ -d /etc ]               # directory exists
[ -L /path/to/link ]      # symbolic link
[ -s file ]               # file has size >0
[ -z "$var" ]             # string is empty
[ -n "$var" ]             # string is non-empty
[ "$a" -eq "$b" ]        # numeric comparison (-eq, -ne, -lt, -le, -gt, -ge)
[[ $str =~ regex ]]       # bash regex match (use double brackets)
```

### Conditional Constructs

```bash
if [ -f file ]; then
  echo "exists"
elif [ -d dir ]; then
  echo "dir"
else
  echo "nope"
fi
```

### Case Statement

```bash
case "$1" in
  start) echo "start" ;;
  stop)  echo "stop"  ;;
  *)     echo "usage" ;;
esac
```

### Loops

```bash
for i in {1..5}; do echo $i; done
for f in *.log; do echo "$f"; done
while read -r line; do echo "$line"; done < file
until command; do sleep 1; done
```

### Functions

```bash
myfunc() {
  local arg="$1"
  echo "Arg: $arg"
}
```

### Arrays

```bash
arr=(one two three)
echo "${arr[0]}"
echo "${#arr[@]}"   # number of elements
for i in "${arr[@]}"; do echo "$i"; done
```

### Here-documents & Here-strings

```bash
cat <<'EOF' > file.txt
literal text, no expansions if quoted 'EOF'
EOF

while read -r line; do echo "$line"; done <<< "one line here-string"
```

### Redirections & Pipelines

```bash
cmd > out.txt            # stdout to file (truncate)
cmd >> out.txt           # append
cmd 2> err.txt           # stderr to file
cmd &> all.txt           # both stdout+stderr (bash specific)
cmd 2>&1 | tee both.txt  # redirect stderr to stdout then pipe
cmd1 | cmd2 | cmd3       # pipelines
cmd &                    # background job
$(cmd)                   # substitution run in subshell
( cd /tmp && ./run )     # subshell; directory change won't persist
{ cmd1; cmd2; }          # grouped commands in current shell
```

### Signal Handling and Cleanup

```bash
trap 'echo "Cleaning up"; rm -f /tmp/mytmp; exit' INT TERM EXIT
```

### Strictness and Debugging

```bash
set -e      # exit on error
set -u      # error on unset variables
set -o pipefail # pipeline returns failure if any command fails
set -x      # debug: print commands
# Recommended combined:
set -euxo pipefail
```

### Exit Codes

- `0` = success; non-zero = failure
- Use `exit N` to return code
- Check last command: `$?`

### Process Substitution (bash)

```bash
diff <(cmd1) <(cmd2)
```

### Common Idioms

```bash
: "${VAR:=default}"     # set VAR to default if unset
>&2 echo "error"        # write to stderr
if command -v program >/dev/null 2>&1; then echo "installed"; fi
```

---

## 🔍 5. grep — Patterns, Flags & Practical Examples

### Flags (Common & Useful)

- `-i` — Ignore case
- `-v` — Invert match (show lines that DO NOT match)
- `-r` — Recursive through directories
- `-R` — Recursive, follow symlinks
- `-n` — Show line numbers
- `-c` — Count matching lines per file
- `-l` — List file names that match
- `-L` — List files that do NOT match
- `-o` — Show only matched (useful with -P/-E)
- `-E` — Use extended regex (egrep)
- `-F` — Fixed-strings (fgrep) — fast, no regex parsing
- `-P` — Use Perl-Compatible Regular Expressions (PCRE)
- `-w` — Match whole words only
- `-x` — Match whole line
- `-m NUM` — Stop after NUM matches
- `-H` — Always print filename
- `-s` — Suppress error messages
- `--color=auto` — Highlight matches

### Regex & Pattern Basics

- `.` matches any single character
- `*` previous token 0 or more times
- `+` previous token 1 or more times (use -E or -P)
- `?` previous token 0 or 1 time (use -E or -P)
- `^` anchor: start of line
- `$` anchor: end of line
- `[]` character classes: `[abc]`, `[a-z]`, `[0-9]`
- `[^...]` negated class
- `\b` word boundary (PCRE / -P)
- `\s` whitespace, `\S` non-whitespace (PCRE)
- `()` grouping and capturing (use -E/-P)
- `|` alternation (OR) (use -E/-P)
- `\` escape special characters
- `{n,m}` quantifiers (PCRE/Extended)

### Practical Examples

```bash
grep "error" /var/log/syslog
grep -i "error" /var/log/syslog
grep -n "TODO" -R src/
grep -E "colou?r" file.txt                     # colour or color
grep -P '\bfunction\b' file.py                 # match whole word 'function' (PCRE)
grep -oP '(?<=src=")[^"]+' index.html          # extract src attributes (PCRE lookbehind)
grep -E '^(INFO|ERROR|WARN)' logfile.log       # match lines starting with these tokens
grep -E 'foo.*bar' file                         # foo then later bar on same line
grep -E '^foo' file                              # lines that start with 'foo'
grep -E 'bar$' file                              # lines that end with 'bar'
grep -w 'port' file                              # match whole word 'port' not 'passport'
grep -F 'a+b' file                                # literal string "a+b" without regex
grep -m 1 'pattern' file                         # stop after first match
grep -c 'pattern' file                           # count matches
```

### Tips

- Quote patterns to prevent shell globbing: `grep "a*b" file` (not `grep a*b file`)
- Use `-r --include/--exclude` to filter files: `grep -R --include="*.py" "TODO" .`
- For binary files, use `-a` to treat as text: `grep -a "pattern" binaryfile`
- Combine with sed/awk for complex extraction pipelines
- Use ack/ag/rg (ripgrep) for faster searching in large trees

---

## 🔎 6. find — More Examples & Safe Usage

### Common Usage Examples

```bash
find / -name file.txt                       # Find by exact name
find . -type f -name "*.py"                 # Find python files
find /etc -mtime -1                         # Files modified within last day
find . -size +100M                          # Files larger than 100MB
find . -empty                                # Empty files/dirs
find / -perm -4000                          # Files with setuid bit
find . -iname "*.log"                       # Case-insensitive name search
find . -maxdepth 2 -type d                   # Limit depth
find . -exec chmod 644 {} \;                 # Run chmod on each result (careful)
find . -execdir rm -v {} \;                  # Safer deletion relative to each file's dir
find . -delete                                # Delete matches (use with care)
find . -print0 | xargs -0 rm -f              # Safe handling of whitespace/newlines
```

---

## 🧰 7. File & Disk Utilities

### dd

```bash
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress  # Clone disk
dd if=/dev/zero of=file.img bs=1M count=100       # Create 100MB zero file
dd if=backup.img of=/dev/sdb conv=fsync,status=progress
```

### df / du / mount / umount

```bash
df -h                                      # Show mounted volumes
du -sh *                                   # Folder sizes
sudo mount /dev/sdb1 /mnt                  # Mount device
sudo umount /mnt                           # Unmount
```

### tar / gzip

```bash
tar -czvf backup.tar.gz /folder            # Create archive
tar -xzvf backup.tar.gz                    # Extract archive
tar -tf file.tar.gz                        # List contents
```

### rsync

```bash
rsync -avh src/ dest/                      # Sync directories
rsync -avz --progress remote:/data /backup # Remote sync via SSH
rsync --delete src/ dest/                  # Mirror (delete on dest)
```

### chmod / chown

```bash
chmod +x script.sh
chmod 644 file
sudo chown user:group file
```

---

## 🌐 8. Networking & Monitoring

### tcpdump

```bash
sudo tcpdump -i eth0                        # Capture packets on interface
sudo tcpdump -n port 80                     # Filter by port (no DNS resolution)
sudo tcpdump -vv -i eth0 host 192.168.0.10  # Verbose capture
sudo tcpdump -w out.pcap                    # Save to file
sudo tcpdump -r out.pcap                    # Read capture file
```

### ip

```bash
ip a                                       # Show interfaces and IPs
ip link set eth0 up                        # Bring up interface
ip addr add 192.168.1.10/24 dev eth0       # Add IP to interface
ip route                                   # Show routing table
ip neigh                                   # ARP table
```

### netstat / ss

```bash
netstat -tuln                               # Listening TCP/UDP ports
netstat -anp                                # All sockets + process ids (older)
ss -tuln                                    # Modern alternative to netstat
```

### curl / wget / traceroute / ping

```bash
curl -I https://example.com                 # Get headers
curl -fsSL https://example.com -o file      # Fetch quietly, fail on error
wget https://example.com/file.zip           # Download file
traceroute 8.8.8.8                          # Trace network hops
ping -c 4 8.8.8.8                           # Ping host
```

### scp / ssh

```bash
scp file user@remote:/path                  # Copy file to remote host
scp -r folder user@remote:/path             # Recursive copy
ssh user@host                               # Connect to remote host
ssh -L 8080:localhost:80 user@remote        # Local port forwarding
```

---

## ⏱️ 9. Process & Resource Management

```bash
top                                         # Monitor processes (interactive)
htop                                        # Improved interactive monitor
ps aux | grep name                          # Find process by name
lsof -i :8080                               # Which process uses a port
kill PID                                    # Graceful terminate
kill -9 PID                                 # Force kill (use as last resort)
pkill -f name                               # Kill processes by matching name
nice -n 10 command                          # Run with lower priority
renice -n 5 -p PID                          # Change priority of running process
watch -n 2 "lsusb"                          # Run command repeatedly (every 2s)
strace -f -o trace.log command              # Trace system calls
ltrace command                              # Trace library calls
```

---

## 🧱 10. Building From Source — Tools & Quick Guide

### Common Helpers

- **pkg-config**: discover compiler/linker flags for libraries  
  `pkg-config --cflags --libs libname`
- **ccache**: cache compiled objects to speed rebuilds  
  `ccache gcc ...`
- **install-sh / checkinstall**: helpers for packaging installs

### Autotools (Classic GNU Flow)

```bash
./autogen.sh                               # or autoreconf -i
./configure --prefix=/usr/local            # Configure build options
make -j$(nproc)                            # Build with parallel jobs
sudo make install                          # Install
sudo make uninstall                        # Uninstall (if supported)
```

### CMake (Cross-platform)

```bash
mkdir build; cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
cmake --build . -- -j$(nproc)              # Build
sudo cmake --install .                     # Install (CMake 3.15+)
# Modern invocation:
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=/usr/local
```

### Meson + Ninja (Fast Modern Workflow)

```bash
meson setup builddir --prefix=/usr/local
meson configure builddir                   # show options
ninja -C builddir                          # build
sudo ninja -C builddir install             # install
meson test -C builddir                     # run tests
```

### Ninja (Low-level Fast Builder)

```bash
ninja -C builddir                          # Build using generated build.ninja
```

### Tips

- Use `-DCMAKE_INSTALL_PREFIX=/usr/local` or Meson's `--prefix` to keep locally built software separate
- Use `checkinstall` to produce a distro package for easy removal
- Clean builds: `rm -rf build` and recreate build directory to avoid stale artifacts
- Use `DESTDIR` during packaging: `make install DESTDIR=/tmp/package-root`

---

## 👀 11. File Watchers (inotify)

```bash
inotifywait -m /dir                           # Monitor changes continuously
inotifywait -e create,delete,modify -r /path  # Watch events recursively
inotifywatch -v /tmp                          # Summarize activity
```

---

## 📁 12. Filesystem Hierarchy (FHS) Overview

### Key Directories

- `/etc` — System configuration (shipable config)
- `/usr/bin` — Distro-managed executables (read-only for packages)
- `/usr/local/bin` — Local admin-installed executables (your builds)
- `/usr/lib` — Shared libraries for system packages
- `/opt` — Third-party or optional applications
- `/tmp` — Temporary runtime data (may be cleared on reboot)
- `/var/log` — Log files and variable runtime data
- `/home` — User home directories
- `/srv` — Data served by services (http, ftp, etc.)

### Guidelines

- Install your compiled tools under `/usr/local` or `/opt` to avoid clobbering package-managed files
- Use `/etc` for system configuration; do not put local executables there
- Use `/tmp` for ephemeral files; use `/var/tmp` for longer lived temporary data

---

## 🕓 13. Scheduled Tasks (cron) & systemd Timers

### cron

```bash
crontab -e                                  # Edit user's crontab
crontab -l                                  # List crontab entries
# System cron jobs: /etc/cron.hourly /etc/cron.daily
```

**Example cron entry:**
```
0 2 * * * /usr/local/bin/backup.sh        # Run daily at 2:00 AM
```

### systemd Timers (Better for Complex Tasks)

Create `mytimer.timer` and `mytimer.service` in `/etc/systemd/system/`

**Example timer unit:**
```ini
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

Enable and start: `systemctl enable --now mytimer.timer`

---

## 🛠️ 14. udev — Rules Guide & Examples

### Where to Put Rules

- `/etc/udev/rules.d/` — Administrator rules (overrides vendor rules)
- `/lib/udev/rules.d/` — Package-supplied rules (do not edit here)

### Rule Syntax Basics

```bash
SUBSYSTEM=="usb"               # Match kernel subsystem
KERNEL=="sd*"                  # Match kernel device name
ATTR{vendor}=="0x1234"         # Match device attribute
ATTR{name}=="..."              # Match device name attribute
ENV{ID_MODEL}=="..."           # Match udev-provided env variables
MODE="0660"                    # Set device node permissions
OWNER="root"                   # Set owner
GROUP="video"                  # Set group
SYMLINK+="mycam"               # Create a symlink /dev/mycam -> actual device node
NAME="customname"              # Set device node name (rare; be careful)
RUN+="/usr/local/bin/hook.sh"  # Run a program (asynchronous; keep fast)
TAG+="systemd"                 # Tag device for systemd integration
OPTIONS+="last_rule"           # Stop processing following rules
IMPORT{program}="/bin/sh -c '...' "  # Import environment from external program
```

### Matching Examples

**Video camera example:**
```
SUBSYSTEM=="video4linux", ATTR{name}=="HD USB Camera", SYMLINK+="video_front_rgb", MODE="0660", GROUP="video"
```

**Block device example (persistent symlink for USB disk by serial):**
```
SUBSYSTEM=="block", KERNEL=="sd?", ENV{ID_SERIAL_SHORT}=="123456789", SYMLINK+="disk_by_serial/usbdisk1", MODE="0660", OWNER="backup"
```

**Network interface naming:**
```
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="aa:bb:cc:dd:ee:ff", NAME="eth-backup"
```

### Notes & Debugging

```bash
# After editing rules:
udevadm control --reload-rules
udevadm trigger

# Inspect device attributes:
udevadm info -a -p $(udevadm info -q path -n /dev/video0)

# Monitor events:
udevadm monitor --udev
udevadm monitor --udev --kernel
```

**Example file:** `/etc/udev/rules.d/99-mycam.rules`
```
SUBSYSTEM=="video4linux", ATTR{name}=="HD USB Camera", SYMLINK+="video_front_rgb", MODE="0660", GROUP="video", TAG+="systemd"
```

**Important:** Avoid long-running programs in `RUN+=`; prefer systemd tmpfiles or systemd services triggered by udev tag+systemd.

---

## 💡 15. Best Practices & Conventions

- Put admin-installed binaries in `/usr/local/bin` and libraries in `/usr/local/lib`
- Keep package-managed files in `/usr` and `/lib`; don't overwrite them
- Use `/opt` for large third-party bundles when appropriate
- Write services with clear logging (to stdout/stderr so journal collects it)
- Use systemd timers instead of cron for better integration with systemd service units
- Use safe find patterns (`print0` + `xargs -0`) to handle spaces/newlines in filenames
- Test systemd units with `systemd-analyze verify` and `systemctl start` in non-production first
- For udev rules, prefer `SYMLINK` over `NAME` unless you fully understand kernel naming implications
- Keep udev RUN scripts very short — long-running tasks should be done by a systemd service

---

## 📚 16. References & Further Reading

- systemd.service(5) and systemd.unit(5)
- udev man pages: `man udev`, `man udevadm`
- v4l2loopback GitHub: https://github.com/umlaeute/v4l2loopback
- Filesystem Hierarchy Standard (FHS)
- GNU Autotools, CMake, Meson documentation
- Bash Reference Manual

---

**Author:** Generated by ChatGPT (GPT-5 Thinking mini)  
**License:** MIT  
**Last Updated:** October 19, 2025
