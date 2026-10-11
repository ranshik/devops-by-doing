---
title: DevOps Mastery â€” Learn by Doing
description: A production-grade, hands-on DevOps learning path covering Linux, networking (on-prem + cloud), every major cloud (AWS, Azure, GCP), containers, orchestration, IaC, CI/CD, observability, security, and platform engineering.
---

# ðŸ› ï¸ DevOps Mastery â€” Learn by Doing

> Welcome. This is a **single, sequential learning path** for mastering DevOps end-to-end. Every tool is hands-on, production-grade, and ends in a real project. Work through them in order â€” each builds on the last.

> **How to use this file:** Open `index.html` in a browser. Each `## ` section becomes a tool card. Click any card to open the full step-by-step guide, code samples, and a checkable practice project.

> **Suggested cadence:** 1â€“2 tools per week. Each tool is sized to fit a focused weekend of practice.

---

## ðŸ§ Linux Fundamentals

**Difficulty:** Beginner | **Time:** 20-25 hours

### Overview
Linux is the operating system that runs the internet â€” every cloud VM, every container host, every network appliance, and almost every CI/CD runner runs a Linux kernel. Before you can operate Docker, Kubernetes, Terraform, or any cloud, you need to be fluent in the Linux shell, file system, processes, users, packages, networking stack, and service management. This tool is the foundation of the entire DevOps path.

### Why Learn This
- ~96% of the top 1 million web servers run Linux â€” it's the lingua franca of DevOps
- Every troubleshooting session you will ever do in production starts at a Linux shell
- Cloud VMs (EC2, Azure VM, GCE), containers, and Kubernetes nodes all expose Linux internals
- Shell scripting and Linux automation are the prerequisites for Ansible, Docker, and CI/CD
- You cannot secure, monitor, or debug what you don't understand at the OS level

### Learn by Doing â€” Step by Step

#### Step 1: Set Up Your Linux Lab
1. **Choose a distro** â€” for production parity, use **Ubuntu 22.04 LTS** or **Rocky Linux 9**. Mac/Windows users can use the WSL2 (Windows Subsystem for Linux) or multipass.
   ```bash
   # On Windows (PowerShell admin)
   wsl --install -d Ubuntu-22.04

   # On macOS
   brew install --cask multipass
   multipass launch --name devops-lab --cpus 2 --memory 4G --disk 20G lts

   # On Linux
   sudo apt update && sudo apt upgrade -y
   ```
2. **Verify your environment:**
   ```bash
   uname -a
   cat /etc/os-release
   lsb_release -a
   whoami
   hostname
   ip a
   ```
3. **Install the essential toolkit:**
   ```bash
   sudo apt install -y curl wget git vim nano htop net-tools \
     iproute2 dnsutils traceroute tcpdump nmap bash-completion \
     ca-certificates gnupg lsb-release apt-transport-https
   ```
4. **Troubleshooting tip:** If `apt` is locked (`Could not get lock /var/lib/dpkg/lock`), wait a moment or check `ps aux | grep apt`.

#### Step 2: The File System & Navigation
1. **Understand the hierarchy** â€” every Linux system follows the FHS (Filesystem Hierarchy Standard):
   ```bash
   # The 5 minute tour
   ls -la /
   ```
   | Path | Purpose |
   |------|---------|
   | `/` | Root of the entire filesystem |
   | `/bin`, `/sbin` | Essential user + system binaries |
   | `/etc` | System-wide configuration (the most important dir) |
   | `/home` | User home directories |
   | `/var` | Variable data â€” logs, databases, spool |
   | `/var/log` | Almost all log files live here |
   | `/tmp` | Temporary files (cleared on reboot) |
   | `/opt` | Optional / third-party software |
   | `/usr` | User programs (read-only on most distros) |
   | `/proc`, `/sys` | Virtual filesystems â€” kernel and process info |
   | `/dev` | Device files (disks, terminals, etc.) |
   | `/boot` | Bootloader, kernel, initramfs |
2. **Master navigation:**
   ```bash
   pwd                       # where am I
   cd /etc
   ls -lah                   # human-readable sizes, hidden files, long format
   ls -lSh                   # sort by size
   ls -lt                    # sort by mtime
   tree -L 2 /etc            # recursive tree (install with apt if missing)
   ```
3. **Find anything, fast:**
   ```bash
   # Find a file by name
   find /etc -name "*.conf" 2>/dev/null | head
   # Find a file by content (grep recursively)
   grep -r "DocumentRoot" /etc/apache2/ 2>/dev/null
   # Locate using a prebuilt index
   sudo updatedb && locate nginx.conf
   # Find files modified in the last day
   find /var/log -mtime -1 -type f
   # Find files by size (>100M)
   find / -size +100M -type f 2>/dev/null
   ```
4. **Hands-on exercise:** Without using a file manager, find every `.log` file under `/var` that was modified in the last 24 hours and `wc -l` the 5 largest ones.

#### Step 3: Users, Groups & Permissions
1. **The permission model â€” read this twice:**
   ```bash
   ls -l /etc/passwd
   # -rw-r--r-- 1 root root 2932 Oct 11 10:00 /etc/passwd
   #  ^  ^       ^  ^     ^    ^     ^
   #  |  |       |  |     |    |     filename
   #  |  |       |  |     |    mtime
   #  |  |       |  |     group
   #  |  |       |  owner
   #  |  |       other permissions (r--)
   #  |  group permissions (r--)
   #  |  owner permissions (rw-)
   #  type (- = file, d = dir, l = symlink)
   ```
2. **Numeric vs symbolic modes:**
   ```bash
   # numeric: r=4, w=2, x=1  â†’  755 = rwxr-xr-x
   chmod 755 script.sh
   # symbolic
   chmod u+x,g-w,o-r file
   # recursive
   chmod -R 750 /opt/myapp
   ```
3. **Manage users and groups:**
   ```bash
   sudo useradd -m -s /bin/bash devops
   sudo passwd devops
   sudo usermod -aG sudo,docker devops   # add to groups
   groups devops
   id devops
   sudo userdel -r devops                # remove user + home
   ```
4. **Special bits â€” setuid, setgid, sticky:**
   ```bash
   # Find all setuid binaries (potential privesc vectors)
   find / -perm -4000 -type f 2>/dev/null
   # Set sticky bit on /tmp (only owner can delete their files)
   chmod +t /tmp
   ls -ld /tmp   # drwxrwxrwt
   ```
5. **Hands-on exercise:** Create a `devops` user, give it sudo + docker group membership, then create a shared directory `/opt/team` where only members of the `team` group can read/write.

#### Step 4: Processes, Services & systemd
1. **Inspect what's running:**
   ```bash
   ps aux                 # all processes
   ps -ef --forest        # tree view
   top                    # interactive
   htop                   # better interactive (installed earlier)
   pgrep -a nginx         # find PIDs by name
   ```
2. **Control processes:**
   ```bash
   # Run a long task in the background
   sleep 1000 &
   jobs -l
   # Bring to foreground
   fg %1
   # Send signals
   kill -TERM <pid>       # graceful (SIGTERM = 15)
   kill -HUP <pid>        # reload config (nginx, sshd)
   kill -KILL <pid>       # force (SIGKILL = 9) â€” last resort
   # Run immune to hangups
   nohup ./long-job.sh &
   ```
3. **systemd â€” the modern service manager:**
   ```bash
   systemctl status ssh
   systemctl list-units --type=service --state=running
   sudo systemctl enable --now nginx       # enable + start
   sudo systemctl restart nginx
   sudo systemctl stop nginx
   journalctl -u nginx -f                  # follow logs
   journalctl -u nginx --since "1 hour ago"
   journalctl -p err -b                    # errors since boot
   ```
4. **Write your own systemd service:**
   ```bash
   sudo tee /etc/systemd/system/myapp.service > /dev/null <<'EOF'
   [Unit]
   Description=My Sample App
   After=network.target

   [Service]
   Type=simple
   User=devops
   WorkingDirectory=/opt/myapp
   ExecStart=/usr/bin/python3 /opt/myapp/server.py
   Restart=on-failure
   RestartSec=5
   Environment=PORT=8080

   [Install]
   WantedBy=multi-user.target
   EOF

   sudo systemctl daemon-reload
   sudo systemctl enable --now myapp
   sudo systemctl status myapp
   ```
5. **Troubleshooting tip:** If a service won't start, always read `journalctl -u <name> -n 50 --no-pager` first. 80% of the time the answer is in the last 10 lines.

#### Step 5: The Networking Stack
1. **Inspect interfaces and addresses:**
   ```bash
   ip a                  # all interfaces
   ip -br a              # brief
   ip r                  # routing table
   ip link               # link layer
   ss -tulnp             # listening sockets + owning process
   netstat -tulnp        # older equivalent
   ```
2. **DNS, the right way:**
   ```bash
   cat /etc/resolv.conf
   systemd-resolve --status
   dig example.com
   dig +short example.com
   dig @8.8.8.8 example.com
   dig -x 1.1.1.1                 # reverse lookup
   host -t MX google.com
   ```
3. **Reachability & path:**
   ```bash
   ping -c 4 1.1.1.1
   traceroute example.com
   mtr -rwc 10 example.com        # combines ping + traceroute, gold standard
   curl -vI https://example.com   # headers + TLS handshake
   ```
4. **Packet capture â€” when you must:**
   ```bash
   sudo tcpdump -i any port 80 -nn
   sudo tcpdump -i eth0 -w /tmp/cap.pcap host 1.1.1.1
   sudo tcpdump -r /tmp/cap.pcap | less
   ```
5. **Hands-on exercise:** Start a `python3 -m http.server 8000`, then in another terminal capture only the traffic to that port with `tcpdump`, then browse to it from a third terminal with `curl`. Decode what you see.

#### Step 6: Package Management & Software
1. **Debian/Ubuntu (apt):**
   ```bash
   sudo apt update
   apt search nginx
   apt show nginx
   sudo apt install nginx
   sudo apt remove nginx
   sudo apt autoremove
   ```
2. **RHEL/Rocky (dnf/yum):**
   ```bash
   sudo dnf install epel-release
   sudo dnf install htop
   sudo dnf remove htop
   ```
3. **Tarballs & binaries â€” the universal fallback:**
   ```bash
   wget https://example.com/tool-1.0.tar.gz
   tar -xzf tool-1.0.tar.gz
   cd tool-1.0 && ./configure && make && sudo make install
   # or for static binaries
   sudo install -m 0755 kubectl /usr/local/bin/kubectl
   ```
4. **Source vs binary â€” when to use which:** distros lag behind upstream. For tools like `kubectl`, `terraform`, `helm`, always install the upstream binary. For system services like `nginx`, `postgresql`, prefer the distro package so updates flow through your security patching.

#### Step 7: Shell Scripting for DevOps
1. **Bash fundamentals:**
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail          # fail fast: errors, unset vars, pipe failures

   LOG=/var/log/myapp.log
   mkdir -p "$(dirname "$LOG")"

   if [[ -f "$LOG" ]]; then
     echo "Log exists, size: $(wc -c < "$LOG")"
   else
     echo "Log missing, creating" | tee -a "$LOG"
   fi

   for svc in nginx ssh cron; do
     if systemctl is-active --quiet "$svc"; then
       echo "OK: $svc"
     else
       echo "DOWN: $svc" >&2
     fi
   done
   ```
2. **Functions, args, and traps:**
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail

   log() { printf '%s [%s] %s\n' "$(date -Is)" "$1" "${*:2}"; }

   cleanup() { log INFO "Cleaning upâ€¦"; rm -f /tmp/mylock; }
   trap cleanup EXIT

   main() {
     local env="${1:-dev}"
     log INFO "Deploying to $env"
     # ... deploy steps ...
   }

   main "$@"
   ```
3. **SSH â€” the most-used tool in DevOps:**
   ```bash
   # Generate a key (always ed25519 in 2026)
   ssh-keygen -t ed25519 -C "you@host"
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ssh-copy-id user@server
   # Config shortcuts
   cat >> ~/.ssh/config <<'EOF'
   Host prod-bastion
     HostName 10.0.0.5
     User ubuntu
     IdentityFile ~/.ssh/id_ed25519
   EOF
   ssh prod-bastion
   # Tunnel
   ssh -L 8080:internal-service:80 bastion
   # Jump host
   ssh -J bastion user@internal
   ```
4. **Hands-on exercise:** Write a `backup.sh` that tars `/etc` and `/var/log` to `/backups`, rotates files older than 7 days, and exits non-zero on failure. Run it from cron.

#### Step 8: Logs, Performance & Observability at the OS Level
1. **The log locations that matter:**
   ```bash
   /var/log/syslog       # Debian/Ubuntu general
   /var/log/messages     # RHEL general
   /var/log/auth.log     # SSH, sudo, PAM
   /var/log/kern.log     # kernel
   /var/log/nginx/       # nginx
   /var/log/journal/     # systemd journal
   ```
2. **System load & resource pressure:**
   ```bash
   uptime
   free -h
   nproc
   cat /proc/loadavg
   vmstat 1 5
   iostat -xz 1 5            # install sysstat
   iotop                     # install iotop
   ```
3. **The USE method (Brendan Gregg) â€” for every resource: Utilization, Saturation, Errors:**
   - CPU: `top`, `mpstat -P ALL 1`
   - Memory: `free -h`, `vmstat`
   - Disk: `iostat`, `df -h`, `du -sh /*`
   - Network: `sar -n DEV 1`, `iftop`, `nethogs`
4. **Hands-on exercise:** Run `stress-ng --cpu 4 --vm 2 --vm-bytes 1G --timeout 60s` and watch what happens in `top`, `free -h`, and `iostat` simultaneously.

#### Step 9: Security Hardening Basics
1. **Patching:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo unattended-upgrade --dry-run
   # RHEL
   sudo dnf check-update
   sudo dnf update -y
   ```
2. **SSH hardening (minimum):**
   ```bash
   sudo tee /etc/ssh/sshd_config.d/hardening.conf > /dev/null <<'EOF'
   Port 2222
   PermitRootLogin no
   PasswordAuthentication no
   PubkeyAuthentication yes
   MaxAuthTries 3
   ClientAliveInterval 300
   AllowUsers devops
   EOF
   sudo sshd -t && sudo systemctl restart sshd
   ```
3. **The firewall â€” `ufw` for Ubuntu, `firewalld` for RHEL:**
   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow 2222/tcp        # SSH on new port
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw enable
   sudo ufw status verbose
   ```
4. **Audit & intrusion detection primitives:**
   ```bash
   sudo apt install -y auditd fail2ban
   sudo systemctl enable --now auditd fail2ban
   sudo fail2ban-client status sshd
   sudo ausearch -m USER_LOGIN -ts today
   ```
5. **Hands-on exercise:** Harden an Ubuntu VM end-to-end: change SSH port, disable root login, enable ufw, install fail2ban, then run `nmap` against it from another machine and confirm only port 2222 is open.

#### Step 10: Storage, LVM & Filesystems
1. **Disks and partitions:**
   ```bash
   lsblk
   sudo fdisk -l
   sudo parted /dev/sdb mklabel gpt
   sudo parted /dev/sdb mkpart primary ext4 0% 100%
   sudo mkfs.ext4 /dev/sdb1
   sudo mkdir /data && sudo mount /dev/sdb1 /data
   # Persistent mount
   echo "UUID=$(blkid -s UUID -o value /dev/sdb1) /data ext4 defaults,nofail 0 2" \
     | sudo tee -a /etc/fstab
   sudo mount -a
   ```
2. **Logical Volume Manager (LVM) â€” for resizable storage:**
   ```bash
   sudo pvcreate /dev/sdc
   sudo vgcreate vgdata /dev/sdc
   sudo lvcreate -L 10G -n lvapps vgdata
   sudo mkfs.xfs /dev/vgdata/lvapps
   # Resize online
   sudo lvextend -L +5G /dev/vgdata/lvapps
   sudo xfs_growfs /data
   ```
3. **Hands-on exercise:** Add a second virtual disk to your VM, partition it, mount it persistently, then convert it to LVM and resize it live while writing data to a file.

#### Step 11: Production-Grade Linux Habits
- **Never** edit files on a server without version control â€” use `etckeeper` (`sudo apt install etckeeper && sudo etckeeper init`) or Ansible
- **Immutable infrastructure** mindset: treat servers as cattle, not pets
- **Document the golden image** in Packer or cloud-init, not on a wiki
- **Centralize logs** with journald forwarding, rsyslog, or a sidecar like Vector
- **Patch on a schedule**, not when you feel like it
- **Use configuration management** (Ansible, later in the path) â€” manual `vi` is a smell

### Practice Project â€” Build a Hardened Linux Server

> **Goal:** From zero, build a production-shaped Ubuntu 22.04 server with users, services, networking, storage, observability, and a runbook.
> **Time estimate:** 8-10 hours
> **Deliverables:** A working VM, a Git repo of all configs, and a `RUNBOOK.md`.

#### Phase 1 â€” Foundation (2h)
- [ ] **(B)** Provision an Ubuntu 22.04 VM (multipass/VMware/Proxmox). Document its specs in `INVENTORY.md`
- [ ] **(B)** Update + upgrade the system, set timezone to UTC, set hostname to `lab01`
- [ ] **(B)** Install the essential toolkit from Step 1
- [ ] **(B)** Create 3 users: `devops` (sudo), `deploy` (no sudo), `app` (service account) â€” Step 3
- [ ] **(B)** Add a `team` group, set up `/opt/team` shared directory with `setgid` â€” Step 3

#### Phase 2 â€” Network & SSH (1.5h)
- [ ] **(I)** Move SSH to port 2222, disable root + password auth â€” Step 9
- [ ] **(I)** Generate an ed25519 key, push to the new `devops` user â€” Step 7
- [ ] **(I)** Enable ufw, allow only 2222 + 8080 â€” Step 9
- [ ] **(I)** Install + enable fail2ban, prove it bans a brute-force attempt â€” Step 9
- [ ] **(I)** Confirm from a second machine: only ports 2222/8080 reachable via `nmap`

#### Phase 3 â€” Service & Automation (2h)
- [ ] **(I)** Write the `myapp` systemd service from Step 4 with auto-restart + env vars
- [ ] **(I)** Verify it survives `kill -9` and is reachable on port 8080
- [ ] **(I)** Write `backup.sh` from Step 7 + add a daily cron at 02:17 (off-round minute)
- [ ] **(I)** Verify the backup ran, file size > 1MB, and old backups are rotated

#### Phase 4 â€” Storage (1h)
- [ ] **(I)** Attach a second virtual disk, partition with GPT, format as ext4 â€” Step 10
- [ ] **(I)** Mount persistently via fstab with UUID, `nofail` option â€” Step 10
- [ ] **(I)** Convert the partition to an LVM volume and resize it live while writing data â€” Step 10

#### Phase 5 â€” Observability & Security (1.5h)
- [ ] **(I)** Install auditd, generate a one-line summary of today's auth failures â€” Step 9
- [ ] **(I)** Run `stress-ng` and capture `vmstat`, `iostat`, `mpstat` during the load â€” Step 8
- [ ] **(I)** Capture a tcpdump pcap of a port scan and annotate every line â€” Step 5

#### Phase 6 â€” Documentation & DR (1h)
- [ ] **(A)** Write `/root/RUNBOOK.md` documenting every command needed to rebuild the server
- [ ] **(A)** Initialize a Git repo in `/etc` with `etckeeper`, commit every config change
- [ ] **(A)** Practice a full rebuild from the runbook on a fresh VM in < 60 minutes

#### Bonus Challenges
- [ ] **(A)** Add WireGuard so the server is only reachable through the tunnel
- [ ] **(A)** Enable BBR + the Step 6 sysctl tuning and benchmark before/after with `iperf3`
- [ ] **(A)** Set up `sysstat`/`sar` to collect metrics every 5 minutes for a week, then graph

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 22 tasks across 6 phases + 3 bonus challenges**

### Key Takeaways
- The Linux shell is not optional â€” it is your IDE, your debugger, and your audit tool
- `systemd` + `journalctl` is the canonical place to look for any service problem
- SSH keys + ufw + fail2ban + auditd is the minimum bar for any internet-facing host
- LVM + fstab + UUIDs is how you build storage that survives reboots
- Every later tool in this path â€” Docker, Kubernetes, Terraform, Ansible â€” is just Linux with opinionated defaults

---

## ðŸŒ Networking Fundamentals

**Difficulty:** Beginner | **Time:** 25-30 hours

### Overview
Networking is the substrate of every distributed system you will ever build. From the moment a packet leaves a container, through a vSwitch, across an underlay, into a load balancer, over a VPN, through a cloud edge router, and into another VPC â€” every DevOps problem eventually becomes a networking problem. This tool takes you from OSI model to BGP, from subnetting to eBPF, and gives you the mental model you need to debug anything in production.

### Why Learn This
- 80% of "the service is down" tickets are networking tickets in disguise
- Kubernetes CNI, service mesh, cloud VPCs, and zero-trust are all built on these primitives
- Performance, latency, and reliability budgets are network-shaped
- Security boundaries (security groups, NACLs, netpol) all map to the same packet model
- Mastering tools like Wireshark, iperf, and tcpdump will save your career at least once

### Learn by Doing â€” Step by Step

#### Step 1: The OSI & TCP/IP Models â€” The Map of the Territory
1. **The 7-layer OSI model â€” and what each layer actually does:**
   | Layer | Name | PDU | Example protocols | Devices |
   |------:|------|-----|-------------------|---------|
   | 7 | Application | Data | HTTP, DNS, SSH, TLS | proxies, gateways |
   | 6 | Presentation | Data | TLS, MIME, ASCII | â€” |
   | 5 | Session | Data | NetBIOS, RPC, SIP | â€” |
   | 4 | Transport | Segment / Datagram | TCP, UDP | load balancers, firewalls |
   | 3 | Network | Packet | IP, ICMP, OSPF, BGP | routers, L3 switches |
   | 2 | Data Link | Frame | Ethernet, ARP, VLAN | switches, bridges |
   | 1 | Physical | Bits | 1000BASE-T, fiber, radio | cables, NICs, radios |
2. **The 4-layer TCP/IP model (the one you actually use):**
   ```
   Application  â†’ HTTP, gRPC, DNS, SSH
   Transport    â†’ TCP, UDP, QUIC
   Internet     â†’ IP, ICMP
   Link         â†’ Ethernet, Wi-Fi
   ```
3. **Encapsulation â€” how data wraps as it descends:**
   ```
   App data â†’ [TCP header | payload] â†’ [IP header | TCP | payload]
           â†’ [Ethernet header | IP | TCP | payload | FCS]
   ```
4. **Hands-on exercise:** Use `curl -v https://example.com` and identify, line by line, where each layer shows up in the output (DNS, TCP handshake, TLS handshake, HTTP request, response).

#### Step 2: IPv4, Subnetting & CIDR
1. **The anatomy of an IPv4 address:**
   ```
   192.168.10.42/24
   â”‚      â”‚    â”‚  â”‚   â””â”€ prefix length (24 bits = 255.255.255.0)
   â”‚      â”‚    â”‚  â””â”€â”€â”€â”€â”€ host part
   â”‚      â”‚    â””â”€â”€â”€â”€â”€â”€â”€â”€ octet
   â”‚      â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ network part (192.168.10.0)
   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ 32-bit address, dotted quad
   ```
2. **CIDR cheat sheet â€” memorize the /8 â†’ /32 boundaries:**
   | CIDR | Mask | Hosts (usable) | Typical use |
   |-----:|------|---------------:|-------------|
   | /8 | 255.0.0.0 | 16,777,214 | legacy class A |
   | /16 | 255.255.0.0 | 65,534 | small VPC |
   | /20 | 255.255.240.0 | 4,094 | medium subnet |
   | /22 | 255.255.252.0 | 1,022 | K8s node pool |
   | /24 | 255.255.255.0 | 254 | classic /24 |
   | /27 | 255.255.255.224 | 30 | DMZ |
   | /28 | 255.255.255.240 | 14 | bastion pair |
   | /30 | 255.255.255.252 | 2 | point-to-point |
   | /32 | 255.255.255.255 | 1 | single host route |
3. **The 5 private ranges (RFC 1918):**
   - `10.0.0.0/8`
   - `172.16.0.0/12`
   - `192.168.0.0/16`
4. **Special-use ranges you'll see:**
   - `127.0.0.0/8` â€” loopback
   - `169.254.0.0/16` â€” link-local (AWS metadata lives at 169.254.169.254)
   - `100.64.0.0/10` â€” CGNAT / shared address space (often used by cloud egress)
   - `0.0.0.0/0` â€” default route / "any"
5. **Subnet calculation, the right way â€” in your head:**
   ```
   10.20.0.0/22
   Mask:        255.255.252.0
   Block size:  4 (256-252)
   Network:     10.20.0.0
   First host:  10.20.0.1
   Last host:   10.20.3.254
   Broadcast:   10.20.3.255
   Next net:    10.20.4.0
   ```
6. **Hands-on exercise:** Given `172.16.50.0/23`, compute by hand: network address, broadcast, first host, last host, total usable hosts. Then verify with `ipcalc 172.16.50.0/23` (install with `sudo apt install ipcalc`).

#### Step 3: IPv6 â€” You Cannot Skip It
1. **Why IPv6 matters now:** cloud-native (EKS, AKS, GKE) is dual-stack by default; AWS `::/0` is the new default route.
2. **Address format:**
   ```
   2001:0db8:85a3:0000:0000:8a2e:0370:7334
   2001:db8:85a3::8a2e:370:7334          â† compressed
   fe80::1                              â† link-local
   ::1                                  â† loopback
   ::                                   â† unspecified
   ```
3. **Common allocations:**
   - `2000::/3` â€” global unicast (the public internet)
   - `fc00::/7` â€” unique local (ULA, RFC 4193)
   - `fe80::/10` â€” link-local
   - `ff00::/8` â€” multicast
4. **Inspect it on a modern host:**
   ```bash
   ip -6 a
   ping6 2606:4700:4700::1111
   dig +short AAAA google.com
   curl -6 https://ifconfig.co
   ```
5. **Hands-on exercise:** Enable IPv6 on your Linux lab and ping `2606:4700:4700::1111` (Cloudflare DNS) successfully.

#### Step 4: ARP, MAC, VLANs & the L2 World
1. **ARP â€” how IPv4 finds a MAC:**
   ```bash
   ip neigh
   arp -a
   tcpdump -i eth0 arp -nn
   ```
2. **VLANs â€” L2 broadcast domains:**
   ```
   Switch port 1  â”€â”
   Switch port 2  â”€â”¤â”€â”€ VLAN 10 (10.0.10.0/24)
   Switch port 3  â”€â”˜
   Switch port 4  â”€â”€â”€ VLAN 20 (10.0.20.0/24)
   ```
   Tagged frames use 802.1Q (4-byte tag, 12-bit VLAN ID = 4094 VLANs max).
3. **Linux as a switch / router:**
   ```bash
   # Create two VLAN interfaces
   sudo ip link add link eth0 name eth0.10 type vlan id 10
   sudo ip link add link eth0 name eth0.20 type vlan id 20
   sudo ip addr add 10.0.10.1/24 dev eth0.10
   sudo ip addr add 10.0.20.1/24 dev eth0.20
   sudo ip link set eth0.10 up
   sudo ip link set eth0.20 up
   ```
4. **Bridging â€” Linux as a software switch:**
   ```bash
   sudo ip link add br0 type bridge
   sudo ip link set eth1 master br0
   sudo ip link set eth2 master br0
   sudo ip link set br0 up
   # This is exactly what `docker0` and K8s `cbr0` do.
   ```
5. **Hands-on exercise:** Using two netns (network namespaces), connect them with a veth pair, assign /30 addresses, and `ping` across.

#### Step 5: L3 â€” Routing, the Kernel Forwarding Path & the Linux Network Stack
1. **The routing table â€” read it fluently:**
   ```bash
   ip route
   # default via 10.0.0.1 dev eth0
   # 10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.42
   # 172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
   ```
2. **Enable IP forwarding and become a router:**
   ```bash
   echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-forward.conf
   sudo sysctl -p /etc/sysctl.d/99-forward.conf
   cat /proc/sys/net/ipv4/ip_forward  # must print 1
   ```
3. **Network namespaces â€” the building block of containers:**
   ```bash
   sudo ip netns add red
   sudo ip netns add blue
   sudo ip link add veth-red type veth peer name veth-blue
   sudo ip link set veth-red netns red
   sudo ip link set veth-blue netns blue
   sudo ip netns exec red ip addr add 10.99.0.1/24 dev veth-red
   sudo ip netns exec blue ip addr add 10.99.0.2/24 dev veth-blue
   sudo ip netns exec red ip link set veth-red up
   sudo ip netns exec blue ip link set veth-blue up
   sudo ip netns exec red ping 10.99.0.2
   ```
4. **The packet path through the Linux kernel:**
   ```
   NIC â†’ driver â†’ netfilter (PREROUTING) â†’ routing decision
       â†’ INPUT (local) | FORWARD (transit) â†’ POSTROUTING â†’ NIC
   iptables -t nat -L -n -v
   conntrack -L
   ```
5. **Hands-on exercise:** Build a "two-container network" with netns, NAT the red namespace through a third "router" namespace to the internet, and prove that ping to 1.1.1.1 works from "inside" the container.

#### Step 6: TCP Deep Dive â€” The Protocol That Carries the Web
1. **The 3-way handshake:**
   ```
   client â†’ SYN seq=x        â†’
   client â† SYN+ACK seq=y, ack=x+1 â† server
   client â†’ ACK seq=x+1, ack=y+1 â†’
   ```
2. **Watch it happen:**
   ```bash
   sudo tcpdump -i any -nn -S host 1.1.1.1 and tcp
   curl https://1.1.1.1 > /dev/null
   ```
3. **The state machine â€” `ss` shows it:**
   ```bash
   ss -tan state established
   ss -tan state time-wait | wc -l
   # States: LISTEN, SYN-SENT, SYN-RECV, ESTABLISHED, FIN-WAIT-1/2,
   #         CLOSE-WAIT, LAST-ACK, TIME-WAIT, CLOSE, CLOSING
   ```
4. **Tuning the kernel â€” the production defaults that matter:**
   ```bash
   sudo tee /etc/sysctl.d/99-net.conf > /dev/null <<'EOF'
   net.core.somaxconn = 65535
   net.ipv4.tcp_max_syn_backlog = 65535
   net.ipv4.tcp_tw_reuse = 1
   net.ipv4.tcp_fin_timeout = 15
   net.ipv4.ip_local_port_range = 1024 65535
   net.core.default_qdisc = fq
   net.ipv4.tcp_congestion_control = bbr
   net.ipv4.tcp_fastopen = 3
   EOF
   sudo sysctl -p /etc/sysctl.d/99-net.conf
   ```
5. **BBR â€” Bottleneck Bandwidth and Round-trip propagation time â€” Google's congestion control:**
   ```bash
   cat /proc/sys/net/ipv4/tcp_congestion_control   # should print bbr
   ss -tin | grep -i bbr
   ```
6. **Hands-on exercise:** Saturate a link with `iperf3 -c server` between two VMs and compare CUBIC vs BBR throughput and retransmits.

#### Step 7: UDP, QUIC, ICMP â€” And When to Use Each
1. **UDP â€” fire and forget:**
   ```bash
   # DNS is UDP/53
   dig @8.8.8.8 example.com +stats
   # Streaming, gaming, VoIP
   ```
2. **QUIC â€” UDP + TLS + multiplexing, the HTTP/3 carrier:**
   ```bash
   curl --http3 -v https://cloudflare.com 2>&1 | grep -i quic
   ```
3. **ICMP â€” the network's own diagnostic protocol:**
   ```bash
   ping -c 4 -M do -s 1472 1.1.1.1     # don't fragment probe (DF bit)
   # Path MTU discovery
   ip route get 1.1.1.1
   tracepath 1.1.1.1
   ```
4. **Hands-on exercise:** Use `mtr -rwc 100 1.1.1.1` to measure packet loss and jitter over 100 probes, then explain which hop is the bottleneck.

#### Step 8: DNS â€” The Most Important Distributed System You Use Every Day
1. **The hierarchy:**
   ```
   . (root)
   â”œâ”€â”€ com     â”€â”¬â”€ example.com  â”€â”€â”€â”€ www.example.com (A 1.2.3.4)
   â”‚            â””â”€ google.com  â”€â”€â”€â”€ ...
   â”œâ”€â”€ net
   â””â”€â”€ io
   ```
2. **Record types you must know cold:**
   | Type | Purpose | Example |
   |------|---------|---------|
   | A | IPv4 address | `www.example.com. 300 IN A 93.184.216.34` |
   | AAAA | IPv6 address | `www.example.com. 300 IN AAAA 2606:2800:220:1::1` |
   | CNAME | Alias | `blog.example.com. IN CNAME ghs.googlehosted.com.` |
   | MX | Mail exchanger | `example.com. IN MX 10 mail.example.com.` |
   | TXT | SPF, DKIM, verification | `example.com. IN TXT "v=spf1 -all"` |
   | NS | Authoritative servers | `example.com. IN NS ns1.example.com.` |
   | SOA | Zone of authority | start-of-authority record |
   | SRV | Service locator | `_sip._tcp.example.com. IN SRV 0 5 5060 sip.example.com.` |
   | PTR | Reverse | `34.216.184.93.in-addr.arpa. IN PTR www.example.com.` |
   | CAA | Which CAs can issue certs | `example.com. IN CAA 0 issue "letsencrypt.org"` |
3. **Recursive vs authoritative:**
   - **Resolver (recursive):** your ISP's `8.8.8.8`, `1.1.1.1` â€” does the legwork
   - **Authoritative:** the server that *knows* the answer for `example.com`
4. **TTL, caching, and glue records â€” why DNS is half of every outage:**
   ```bash
   dig +trace example.com        # see every server queried
   dig +short example.com
   dig example.com @8.8.8.8      # query a specific resolver
   ```
5. **DNSSEC â€” the chain of trust:**
   ```bash
   dig +dnssec example.com | grep -E "RRSIG|DNSKEY|DS|ad"
   drill -S example.com          # if you have ldnsutils
   ```
6. **Split-horizon DNS â€” on-prem vs external views of the same zone:**
   - Internal clients get RFC 1918 addresses for `db.corp`
   - External clients get a public IP
7. **Hands-on exercise:** Run a local recursive resolver (`unbound` or `bind9`), point your host at it, and prove that every recursive query is logged.

#### Step 9: Firewalls, NAT, and the Linux netfilter Stack
1. **iptables â€” the legacy, still everywhere:**
   ```bash
   sudo iptables -L -n -v
   sudo iptables -t nat -L -n -v
   # Default policy: drop all inbound except established/related
   sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   sudo iptables -A INPUT -i lo -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT
   sudo iptables -A INPUT -j DROP
   ```
2. **nftables â€” the modern replacement:**
   ```bash
   sudo nft list ruleset
   sudo nft add table inet filter
   sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
   sudo nft add rule inet filter input iif lo accept
   sudo nft add rule inet filter input tcp dport 2222 accept
   ```
3. **NAT â€” SNAT vs DNAT vs MASQUERADE:**
   ```bash
   # Outbound (many internal hosts share one public IP)
   sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/16 -o eth0 -j MASQUERADE

   # Inbound port forward
   sudo iptables -t nat -A PREROUTING -d 203.0.113.10 -p tcp --dport 80 \
     -j DNAT --to-destination 10.0.0.5:80
   sudo iptables -A FORWARD -p tcp -d 10.0.0.5 --dport 80 -j ACCEPT
   ```
4. **conntrack â€” the state table:**
   ```bash
   sudo conntrack -L
   sudo conntrack -L | grep ESTABLISHED | wc -l
   # Tunes for high-throughput
   sudo sysctl -w net.netfilter.nf_conntrack_max=524288
   ```
5. **Hands-on exercise:** Set up a Linux box as a NAT gateway for a private subnet and prove a host behind it can reach the internet using the gateway's public IP as source.

#### Step 10: TLS â€” The Cryptographic Backbone
1. **The handshake, simplified:**
   ```
   Client â†’ ClientHello (ciphers, SNI)
   Server â†’ ServerHello, Certificate, ServerHelloDone
   Client â†’ KeyExchange, ChangeCipherSpec, Finished
   Server â†’ ChangeCipherSpec, Finished
   ```
2. **Inspect it yourself:**
   ```bash
   openssl s_client -connect example.com:443 -servername example.com </dev/null
   echo | openssl s_client -connect example.com:443 2>/dev/null \
     | openssl x509 -noout -dates -subject -issuer
   ```
3. **Run your own â€” mkcert + a local CA:**
   ```bash
   sudo apt install mkcert
   mkcert -install
   mkcert myapp.local "*.myapp.local"
   # then point nginx/caddy at myapp.local+1.pem and +1-key.pem
   ```
4. **ACME / Let's Encrypt â€” automation:**
   ```bash
   sudo apt install certbot
   sudo certbot --nginx -d myapp.example.com
   sudo certbot renew --dry-run
   ```
5. **Hands-on exercise:** Generate a wildcard cert with mkcert, serve it from nginx, and verify with `openssl s_client` that the SNI matches and the chain validates.

#### Step 11: Load Balancing, Proxies & the L4/L7 Split
1. **L4 vs L7 â€” know which one you need:**
   | Layer | Decision basis | Examples | Pros | Cons |
   |------:|---------------|----------|------|------|
   | L4 | TCP/UDP src+dst | NLB, IPVS, HAProxy L4, kube-proxy | Fast, low-latency, protocol-agnostic | No header rewriting, no routing by path |
   | L7 | HTTP headers, path, cookie, JWT | nginx, Envoy, ALB, HAProxy L7, Traefik | Path routing, header injection, auth, mTLS termination | More CPU, more knobs |
2. **HAProxy in 60 seconds:**
   ```bash
   sudo apt install haproxy
   sudo tee /etc/haproxy/haproxy.cfg > /dev/null <<'EOF'
   frontend fe
     bind *:80
     default_backend be
   backend be
     balance roundrobin
     server web1 10.0.0.11:8080 check
     server web2 10.0.0.12:8080 check
   EOF
   sudo systemctl enable --now haproxy
   ```
3. **nginx as a reverse proxy + TLS terminator:**
   ```nginx
   server {
     listen 443 ssl http2;
     server_name myapp.example.com;
     ssl_certificate     /etc/letsencrypt/live/myapp.example.com/fullchain.pem;
     ssl_certificate_key /etc/letsencrypt/live/myapp.example.com/privkey.pem;

     location / {
       proxy_pass http://backend_upstream;
       proxy_set_header X-Forwarded-For $remote_addr;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_http_version 1.1;
       proxy_set_header Connection "";
     }
   }
   upstream backend_upstream {
     least_conn;
     server 10.0.0.11:8080;
     server 10.0.0.12:8080;
   }
   ```
4. **Hands-on exercise:** Stand up two `python3 -m http.server 8080` backends, put HAProxy in front, and `wrk -t4 -c100 -d30s http://localhost/` against it. Confirm round-robin distribution in HAProxy stats.

#### Step 12: VPNs, WireGuard & Zero-Trust Networking
1. **WireGuard â€” the modern, fast, small VPN:**
   ```ini
   # /etc/wireguard/wg0.conf on the server
   [Interface]
   Address = 10.99.0.1/24
   ListenPort = 51820
   PrivateKey = <server_priv>
   PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

   [Peer]
   PublicKey = <client_pub>
   AllowedIPs = 10.99.0.2/32
   ```
   ```bash
   sudo systemctl enable --now wg-quick@wg0
   wg show
   ```
2. **IPsec â€” what enterprises still run:**
   - IKEv2 with PSK or certs
   - Site-to-site (VPC peering on AWS, route-based VPN on-prem)
3. **Zero-trust â€” the modern model:**
   - mTLS between every service
   - Identity-based policy (Spiffe/SPIRE, Istio)
   - No implicit network trust
4. **Hands-on exercise:** Set up a WireGuard tunnel between your laptop and your Linux lab, route `10.0.0.0/24` through it, and prove with `mtr` that traffic really transits the tunnel.

#### Step 13: BGP, OSPF & the Routing Protocols You'll Meet in Real Life
1. **OSPF â€” link-state, used inside most enterprises and data centers:**
   - Areas, LSA types, cost = `100 Mbps / link bandwidth`
   - Converges in seconds
2. **BGP â€” path-vector, the protocol of the internet and every cloud VPC peering:**
   - eBGP (between autonomous systems) vs iBGP (within one)
   - Communities, local preference, AS path prepending, MED
3. **A real BGP session with FRRouting (FRR):**
   ```bash
   sudo apt install frr
   sudo sed -i 's/^bgpd=no/bgpd=yes/' /etc/frr/daemons
   sudo systemctl restart frr
   sudo vtysh
   conf t
   router bgp 65000
     neighbor 10.0.0.2 remote-as 65001
     network 10.10.0.0/24
   end
   write mem
   ```
4. **eBGP between two VMs â€” see it work:**
   - VM1 AS 65000, VM2 AS 65001
   - Each advertises a /24
   - Show `show ip bgp summary` and `show ip route bgp`
5. **Hands-on exercise:** Stand up a 3-node FRR topology (full mesh or route reflector) and verify that a route advertised on node1 appears on node3 with the correct AS path.

#### Step 14: The Cloud Networking Model â€” VPC, Subnets, Route Tables, Gateways
1. **Every cloud models the same five primitives:**
   | Concept | AWS | Azure | GCP |
   |---------|-----|-------|-----|
   | Isolated network | VPC | Virtual Network (VNet) | VPC |
   | Subnet | Subnet | Subnet | Subnet |
   | Router | Route Table + IGW | Route Table | Routes (implicit) |
   | NAT egress | NAT Gateway | NAT Gateway | Cloud NAT |
   | Cross-VPC | VPC Peering / TGW | VNet Peering | VPC Peering |
   | Private link | PrivateLink | Private Endpoint | Private Service Connect |
   | DNS | Route 53 + Resolver | Azure DNS | Cloud DNS |
   | Firewall | Security Group + NACL | NSG + ASG | VPC Firewall Rules |
2. **The default VPC anti-pattern â€” and the right way:**
   - Default VPC = open subnets in every AZ = unmitigated blast radius
   - Production design: one VPC per environment, private + public subnets per AZ, NAT in public, workloads in private
3. **VPC design, end to end:**
   ```
   VPC 10.0.0.0/16
   â”œâ”€â”€ public-a  10.0.1.0/24   (AZ a)  â†’ IGW route 0.0.0.0/0
   â”œâ”€â”€ public-b  10.0.2.0/24   (AZ b)  â†’ IGW route 0.0.0.0/0
   â”œâ”€â”€ private-a 10.0.10.0/24  (AZ a)  â†’ NAT route 0.0.0.0/0
   â”œâ”€â”€ private-b 10.0.20.0/24  (AZ b)  â†’ NAT route 0.0.0.0/0
   â”œâ”€â”€ data-a    10.0.100.0/24 (AZ a)  â†’ no internet route
   â”œâ”€â”€ data-b    10.0.200.0/24 (AZ b)  â†’ no internet route
   ```
4. **Hands-on exercise:** Draw the above layout for AWS, Azure, and GCP. Note the differences (AWS route tables are per-subnet, Azure uses route tables attached to subnets, GCP uses VPC-wide routes with subnet overrides).

#### Step 15: The Data Center / On-Prem Network
1. **The classic three-tier:**
   ```
   Internet â†’ Firewall â†’ Core (L3) â†’ Aggregation (L3) â†’ Access (L2) â†’ Servers
                          â”‚                                â”‚
                          â””â”€â”€â”€ Spanning Tree / MLAG â”€â”€â”€â”€â”€â”€â”€â”˜
   ```
2. **The modern leaf-spine (every hyperscaler and most enterprises):**
   ```
   Server â†” Leaf (ToR) â†” Spine â†” Leaf (ToR) â†” Server
   ```
   - Any server is exactly 2 hops from any other
   - BGP or OSPF on the underlay
   - VXLAN-EVPN on the overlay
3. **Overlay networking â€” what every CNI and SD-WAN does:**
   - **VXLAN:** L2 over L3, 24-bit VNI = 16M segments
   - **EVPN:** BGP-based control plane for VXLAN
   - **Geneve:** like VXLAN but extensible (used by Cilium)
4. **Hands-on exercise:** Using two VMs, build a VXLAN between them and ping across the L2-over-L3 tunnel.

#### Step 16: eBPF, Cilium & the New Network Stack
1. **eBPF â€” run sandboxed programs in the kernel without a module:**
   ```bash
   sudo apt install -y linux-headers-$(uname -r) \
     bpfcc-tools linux-headers-generic
   sudo tcpconnect-bpfcc   # live new TCP connections
   sudo tcpaccept-bpfcc
   sudo biolatency-bpfcc   # block I/O latency histogram
   ```
2. **Cilium â€” Kubernetes CNI powered by eBPF:**
   - Replaces kube-proxy, iptables, IPVS
   - L3-L7 policy without sidecars
   - Hubble for observability
3. **Hands-on exercise:** Use `tcpconnect-bpfcc` while running `curl` and `ssh`, and identify which processes made which connections.

#### Step 17: Performance, Observability & Troubleshooting
1. **The DevOps engineer's networking toolkit â€” pinned to muscle memory:**
   ```bash
   ping -c4 host            # reachability
   mtr -rwc 50 host         # path quality
   traceroute -T -p 80 host # path with TCP probes (bypasses ICMP filters)
   dig +trace host          # DNS delegation path
   curl -w "@curl-format" -o /dev/null -s https://host   # timing breakdown
   ss -s                    # socket summary
   ss -tin                  # TCP info (cwnd, rtt, retransmits)
   nstat -az                # kernel netstat counters
   cat /proc/net/dev        # per-NIC byte/packet counters
   ```
2. **A reusable `curl-format.txt`:**
   ```
   time_namelookup:  %{time_namelookup}\n
   time_connect:     %{time_connect}\n
   time_appconnect:  %{time_appconnect}\n
   time_pretransfer: %{time_pretransfer}\n
   time_redirect:    %{time_redirect}\n
   time_starttransfer:%{time_starttransfer}\n
   time_total:       %{time_total}\n
   http_code:         %{http_code}\n
   ```
3. **Wireshark â€” the gold standard:**
   - Display filter: `tcp.port == 443 && ip.addr == 10.0.0.5`
   - Follow stream: right-click â†’ Follow â†’ TCP Stream
   - Decode as: right-click â†’ Decode As â†’ select protocol
4. **The 6 network pathologies you'll see forever:**
   | Symptom | First check |
   |---------|-------------|
   | High latency | `mtr`, `ss -tin`, BBR on? |
   | Packet loss | `mtr -T` (TCP), `ping -f`, NIC ring buffers |
   | DNS slow | `dig +stats`, resolver anycast? |
   | TCP resets | Wireshark, conntrack, firewall state |
   | One-way traffic | asymmetric routing, `ip rule`, RPF check |
   | SSL handshake slow | cert chain, OCSP stapling, MTU |
5. **Hands-on exercise:** Use `tc qdisc` to inject 200ms latency and 1% loss on a loopback interface, then measure the impact on `curl` and `iperf3`.

#### Step 18: Production-Grade Networking Habits
- **Diagram every change.** Draw before deploying. NetBox is the canonical source of truth.
- **Reserve address space top-down.** Plan the /16, then carve /20s, never the other way.
- **Label every interface, every VLAN, every cable.** Future-you will thank present-you.
- **Monitor with synthetic probes.** Blackbox exporter + Prometheus is the minimum.
- **Document the blast radius of every firewall rule.**
- **Test failover.** DR drills are the only way to know your routing actually works.
- **Encrypt by default.** Even inside the VPC. mTLS or WireGuard between every tier.
- **Version your network config.** Terraform or Ansible â€” never a Confluence page.

### Practice Project â€” Build a Production-Grade Mini Network

> **Goal:** From a bare Linux box, build a routed, NATed, segmented network with VPN, load balancing, DNS, BGP, and a full runbook.
> **Time estimate:** 10-12 hours
> **Deliverables:** A working topology, a NetBox IPAM export, and `NETWORKING_RUNBOOK.md`.

#### Phase 1 â€” Linux Router (2h)
- [ ] **(B)** Set up a VM with 3 NICs: WAN, DMZ (10.10.0.0/24), LAN (10.20.0.0/24)
- [ ] **(B)** Enable IP forwarding, configure addresses, and verify reachability within each segment
- [ ] **(I)** Set up SNAT so LAN reaches the internet through the WAN interface
- [ ] **(I)** Set up DNAT to forward WAN:80 to a web server in the DMZ
- [ ] **(I)** Verify with `tcpdump` on every interface that traffic is flowing as expected

#### Phase 2 â€” Firewall & Policy (1.5h)
- [ ] **(I)** Replace the default rules with an explicit default-deny policy
- [ ] **(I)** Allow only SSH (WAN side, port 2222) and the DMZ web (port 80) â€” Step 9
- [ ] **(I)** Add conntrack tracking and prove that established flows are accepted
- [ ] **(I)** Document the rule order in a comment block and explain why ordering matters

#### Phase 3 â€” DNS & Load Balancing (2h)
- [ ] **(I)** Run `unbound` as a local recursive resolver, log every query
- [ ] **(I)** Set up split-horizon: `db.corp` â†’ private IP from LAN, public IP from WAN
- [ ] **(I)** Stand up 2 backend web servers, put HAProxy in front with round-robin
- [ ] **(I)** Benchmark with `wrk -t4 -c100 -d30s`; confirm even distribution in HAProxy stats

#### Phase 4 â€” VPN & BGP (2.5h)
- [ ] **(I)** Set up WireGuard between your router and a second VM, route the DMZ through it
- [ ] **(I)** Prove with `mtr` that traffic really transits the tunnel
- [ ] **(A)** Build a 2-node FRR BGP topology; advertise a /24 from each node
- [ ] **(A)** Verify route propagation, AS path, and remove the advertisement cleanly
- [ ] **(A)** Add a third BGP node, configure it as a route reflector

#### Phase 5 â€” Performance & Hardening (1.5h)
- [ ] **(I)** Apply the BBR + sysctl tuning from Step 6
- [ ] **(I)** Benchmark iperf3 before/after; document the delta
- [ ] **(I)** Use `tc qdisc` to inject 50ms latency + 1% loss; confirm it shows in `mtr` and `iperf`
- [ ] **(I)** Remove the impairment and re-measure

#### Phase 6 â€” Documentation & DR (1.5h)
- [ ] **(I)** Export IPAM as a NetBox-compatible CSV (CIDR, VRF, description, gateway, owner)
- [ ] **(A)** Draw the topology in `topology.drawio` with every interface, IP, and route
- [ ] **(A)** Write `NETWORKING_RUNBOOK.md` with troubleshooting flows for: WAN down, DNS broken, BGP session down, HAProxy full failure
- [ ] **(A)** Practice the "WAN down" runbook end-to-end in < 15 minutes

#### Bonus Challenges
- [ ] **(A)** Add a VXLAN between two VMs and prove it's L2 over L3
- [ ] **(A)** Install Cilium in a Kind cluster and use Hubble to map cluster traffic
- [ ] **(A)** Use `tcpconnect-bpfcc` to profile a 10-minute live capture and produce a service-to-service traffic map

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 22 tasks across 6 phases + 3 bonus challenges**

### Key Takeaways
- Networking is one mental model repeated at every layer: addresses, routes, stateful policy, MTU
- IPv4 and IPv6 are both required; treat IPv6 as the default, not an afterthought
- TCP is the workhorse â€” understand the handshake, the state machine, and the kernel knobs
- BGP/OSPF/VXLAN are not optional vocabulary; they show up in every cloud and every data center
- eBPF is the future of kernel-level observability and policy (Cilium, Falco, Pixie)
- Every production network should be diagrammed, versioned, monitored, and rehearsed

---

## ðŸ”€ Git & GitHub

**Difficulty:** Beginner | **Time:** 20-25 hours

### Overview
Git is the version control system that runs the world. Every line of code, every Kubernetes manifest, every Terraform module, every Ansible playbook, every CI/CD pipeline definition is stored, reviewed, and deployed through Git. GitHub (and its siblings GitLab, Bitbucket, Gitea, Codeberg) added the social layer â€” pull requests, code review, Actions, security scanning, package registries â€” that turned Git from a tool into the platform. Mastering Git is mastering the plumbing of modern software delivery.

### Why Learn This
- Git is the prerequisite for CI/CD, GitOps, IaC, and every modern deploy workflow
- Pull requests are the unit of code review, audit, and compliance in every regulated industry
- "My code works on my machine" disappears when everything â€” including infra â€” is in Git
- Branching strategies directly shape team velocity and incident response
- Knowing Git internals (the object model, refs, packfiles) turns a 10-minute "where did this bug come from" into 30 seconds

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model â€” The Three Trees
1. **The three trees, and why they exist:**
   ```
   Working Directory  â”€â”€ git add â”€â”€â–¶  Staging Area (Index)  â”€â”€ git commit â”€â”€â–¶  Repository (.git/)
          â–²                                                                 â”‚
          â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ git checkout / reset â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
   ```
2. **The object model â€” every commit is a snapshot, not a diff:**
   - **Blob:** file contents (named by SHA of contents)
   - **Tree:** directory listing (blobs + subtrees)
   - **Commit:** pointer to a tree + parent commit + author + message
   - **Tag:** named pointer to a commit
3. **Inspect the model directly:**
   ```bash
   mkdir git-inside && cd git-inside
   git init
   echo "hello" > a.txt
   git add a.txt && git commit -m "init"
   git cat-file -p HEAD          # commit object
   git cat-file -p HEAD^{tree}   # tree
   git cat-file -p <blob-sha>    # contents
   git ls-tree -r HEAD           # all blobs in the tree
   ```
4. **Branches are just pointers:**
   ```bash
   git update-ref refs/heads/main HEAD     # create a branch at current commit
   # Branches live as 41-byte files in .git/refs/heads/
   ```
5. **Hands-on exercise:** Recreate the entire object model by hand: write a blob, a tree, a commit, all with `git hash-object` and `git mktree`.

#### Step 2: The Daily 20 Commands
1. **Stage, commit, inspect:**
   ```bash
   git status
   git status -sb               # short branch + tracking
   git diff                     # unstaged
   git diff --staged            # staged
   git add -p                   # interactive hunks â€” the most underrated command
   git restore file.txt         # discard working tree changes
   git restore --staged file    # unstage
   git commit -m "msg"
   git commit --amend           # edit last commit (only if not pushed)
   git log --oneline --graph --all
   git log -p -- path/          # log for a path
   git show HEAD                # last commit
   ```
2. **Branches, remotes, sync:**
   ```bash
   git switch -c feature/x      # create + switch
   git switch main
   git fetch --all --prune
   git pull --rebase            # default: never merge remote into dirty history
   git push
   git push --force-with-lease  # safe force push (refuses if upstream moved)
   git branch -d feature/x      # delete (safe: refuses unmerged)
   git branch -D feature/x      # force delete
   ```
3. **The .gitignore that actually works:**
   ```gitignore
   # Comments start with #
   *.log
   build/
   !build/.gitkeep
   /dist                      # anchored to repo root
   **/__pycache__
   .env
   .env.*
   !.env.example
   secrets/
   ```
4. **Hands-on exercise:** Initialize a repo, make 5 commits across 2 branches, merge with and without `--no-ff`, and use `git log --graph` to see the difference.

#### Step 3: The Power Commands
1. **Interactive rebase â€” the editing tool of champions:**
   ```bash
   git rebase -i HEAD~5
   # pick   abc1234 first
   # squash def5678 second
   # reword ghi9012 third
   # fixup  jkl3456 typo
   # drop   mno7890 never
   ```
2. **Recover "lost" work:**
   ```bash
   git reflog                          # every move of HEAD for 90 days
   git reset --hard HEAD@{5}           # jump to where you were 5 moves ago
   git fsck --lost-found               # find dangling commits
   # Revert a published commit safely
   git revert <sha>
   ```
3. **Cherry-pick and bisect:**
   ```bash
   git cherry-pick <sha>               # apply one commit elsewhere
   git cherry-pick <sha1>..<sha2>      # apply a range
   git bisect start
   git bisect bad
   git bisect good <known-good-sha>
   # test, then: git bisect good | bad   â†’ git bisect reset
   ```
4. **Stash with intent:**
   ```bash
   git stash push -m "wip on auth" -u   # include untracked
   git stash list
   git stash show -p stash@{0}
   git stash pop
   git stash branch feature/x stash@{0}  # branch off a stash
   ```
5. **Submodules vs subtrees â€” when to use which:**
   - **Submodule:** pointer to a commit in another repo. Use for vendored, separately-versioned deps.
   - **Subtree:** merge another repo into a subdirectory. Use for in-tree vendoring.
6. **Worktrees â€” multiple branches checked out at once:**
   ```bash
   git worktree add ../hotfix main
   git worktree add ../experiment experimental
   git worktree list
   git worktree remove ../hotfix
   ```
7. **Hands-on exercise:** Use `git bisect` to find the commit that introduced a bug you plant in a file.

#### Step 4: Branching Strategies That Don't Hurt
1. **GitHub Flow â€” the simplest that works:**
   ```
   main â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—  (always deployable)
              \           /
   feature â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â”€â”€â—
   ```
   - Branch off `main`, PR back to `main`, deploy from `main`
   - Best for: continuous deployment, trunk-based teams
2. **GitFlow â€” heavier, release-driven:**
   ```
   main â”€â”€â—â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â—â”€â”€â”€â”€â”€â”€â”€â”€â— (releases)
   develop â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—â”€â”€â—
   feature â”€â”€â—â”€â”€â—  â”€â”€â—â”€â”€â—
   release  â”€â”€â”€â”€â”€â”€â—â”€â”€â—â”€â”€â—
   hotfix   â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â—â”€â”€â—
   ```
   - Best for: products with versioning, scheduled releases, multiple supported versions
3. **Trunk-based development â€” Google's model:**
   - Everyone commits to `main` (or `trunk`)
   - Feature flags hide in-progress work
   - Short-lived branches (< 1 day) for review
   - Best for: monorepos, microservices, high-velocity teams
4. **Choose by team size, release cadence, and regulation. Most modern teams converge on trunk-based or GitHub Flow.**
5. **Hands-on exercise:** Run a 3-person simulation: one feature, one hotfix, one release branch. Show the merges, the rebase, the cherry-pick.

#### Step 5: Pull Requests, Code Review & the Social Layer
1. **The anatomy of a good PR:**
   - Title: imperative mood, scannable â€” `feat(api): add idempotency keys to POST /payments`
   - Description: **why** + **what** + **how to test** + screenshots/recordings
   - Small (< 400 lines of diff is the industry guideline)
   - Linked issue / ticket
   - CI green before review
2. **Draft PRs â€” work in public, review later:**
   ```markdown
   ## What
   ## Why
   ## How to test
   - [ ] step 1
   ## Risks
   ## Rollback
   ```
3. **Review etiquette â€” the senior engineer checklist:**
   - Are tests included and meaningful?
   - Are errors handled at the boundary, not deep in the stack?
   - Is the diff scoped to one concern?
   - Are secrets, PII, and config out of the repo?
   - Does the change match the architecture docs?
   - Is there a migration or rollback plan?
4. **Conventional Commits â€” the schema for automated changelogs:**
   ```
   feat: add OAuth login
   fix: correct off-by-one in pagination
   feat(api)!: drop /v1 endpoints
   chore: bump dependencies
   docs: clarify README
   refactor: extract payment validator
   test: add property tests for retry
   ```
   - Pairs with `semantic-release` and `standard-version` for auto-versioning
5. **The CODEOWNERS file â€” make review automatic:**
   ```
   # .github/CODEOWNERS
   /infra/terraform/    @platform-team
   /k8s/                @sre-team
   /services/payments/  @payments-team @security-team
   *.md                 @docs-team
   ```
6. **Hands-on exercise:** Open a PR on a sample repo with a Conventional Commit, a CODEOWNERS file, and a description template â€” and review it as if you were a senior engineer.

#### Step 6: The GitHub Platform â€” Beyond Just Git
1. **Issues & Projects â€” the lightweight PM layer:**
   - Issues: bug reports, feature requests, task tracking
   - Labels, milestones, assignees
   - GitHub Projects (v2): Kanban + sprints, automations
2. **GitHub Actions â€” CI/CD native:**
   ```yaml
   # .github/workflows/ci.yml
   name: CI
   on: [push, pull_request]
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-node@v4
           with:
             node-version: 20
             cache: 'npm'
         - run: npm ci
         - run: npm test
         - run: npm run lint
   ```
3. **GitHub Packages & Container Registry:**
   ```bash
   # Push an image
   echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
   docker build -t ghcr.io/myorg/myapp:1.2.3 .
   docker push ghcr.io/myorg/myapp:1.2.3
   ```
4. **Dependabot & Code Scanning â€” security by default:**
   ```yaml
   # .github/dependabot.yml
   version: 2
   updates:
     - package-ecosystem: "npm"
       directory: "/"
       schedule: { interval: "weekly" }
       open-pull-requests-limit: 10
     - package-ecosystem: "docker"
       directory: "/"
       schedule: { interval: "weekly" }
     - package-ecosystem: "github-actions"
       directory: "/"
       schedule: { interval: "weekly" }
   ```
5. **GitHub Advanced Security â€” code scanning, secret scanning, dependency review:**
   - Push Protection: blocks commits containing secrets
   - CodeQL: semantic code analysis for vulnerabilities
   - Secret scanning: alerts on leaked tokens (GH will auto-revoke many)
6. **Hands-on exercise:** Create a repo, enable Dependabot, push a Dockerfile with an outdated base image, and let Dependabot open a PR.

#### Step 7: Git Internals â€” Packfiles, the Reflog, and Performance
1. **Loose objects vs packfiles:**
   ```bash
   git count-objects -v          # loose + packed
   git gc --aggressive --prune=now
   git repack -ad
   ```
2. **The reflog is your safety net â€” every ref movement, 90 days:**
   ```bash
   git reflog
   git reflog show feature/x
   git reset --hard @{2}
   ```
3. **Filters â€” clean data on commit/ checkout:**
   ```bash
   git config filter.scrubSecret.clean "sed 's/AWS_ACCESS_KEY=.*/AWS_ACCESS_KEY=REDACTED/'"
   git config filter.scrubSecret.smudge "cat"
   echo "*.env filter=scrubSecret" > .gitattributes
   ```
4. **Large file storage (LFS) â€” for binaries:**
   ```bash
   git lfs install
   git lfs track "*.psd" "*.zip" "*.bin"
   git add .gitattributes
   git commit -m "track LFS"
   git lfs ls-files
   ```
5. **Performance tuning for huge repos:**
   ```bash
   git config --global core.preloadindex true
   git config --global core.fsmonitor true
   git config --global feature.manyFiles true
   git config --global index.threads 4
   ```
6. **Shallow clones and partial clones for CI:**
   ```bash
   git clone --depth 1 --branch main https://github.com/myorg/repo
   git clone --filter=blob:none --no-checkout https://github.com/myorg/monorepo
   ```
7. **Hands-on exercise:** Make a 10k-commit repo by `for i in {1..10000}; do echo $i > f && git add f && git commit -m $i; done`, then benchmark `git log` with shallow vs full clones.

#### Step 8: Monorepos, Submodules, and the Repo Strategy Question
1. **Monorepo â€” one repo, many projects (Google, Meta, Microsoft):**
   - Pros: atomic cross-project changes, single source of truth, unified CI
   - Cons: tooling complexity, scale of `.git`, requires partial clones
2. **Polyrepo â€” one repo per service (Netflix, Amazon historically):**
   - Pros: team autonomy, clear ownership, simpler per-repo security
   - Cons: cross-service changes need multiple PRs, dependency hell
3. **Tools for monorepos:**
   - **Bazel** (Google): hermetic, remote build cache, polyglot
   - **Nx** (Nrwl): great for JS/TS, graph-aware CI
   - **Turborepo:** simple, fast, Vercel ecosystem
   - **Pants:** Python + JVM + Go
   - **Buck:** Meta's tool
4. **A monorepo CI trick â€” only build what changed:**
   ```yaml
   # Nx affected
   - run: npx nx affected -t lint test build --base=origin/main --head=HEAD
   ```
5. **When to use submodules:**
   - Vendoring a shared library that has its own release cadence
   - The risk: every developer needs to remember `git submodule update --init --recursive`
6. **Hands-on exercise:** Create a 3-service monorepo (API, worker, web), set up Nx, and prove that changing one service doesn't trigger the others' CI.

#### Step 9: Hooks â€” Enforce the Rules Automatically
1. **Client-side hooks (`.git/hooks/` or `core.hooksPath`):**
   ```bash
   # .githooks/pre-commit
   #!/usr/bin/env bash
   set -e
   # Block secrets with gitleaks
   gitleaks protect --staged --redact --no-banner
   # Lint and format
   npm run lint
   npx prettier --check .
   ```
   ```bash
   git config core.hooksPath .githooks
   ```
2. **Server-side hooks (in bare repos) â€” for centralized control:**
   - `pre-receive`: enforce branch naming, block large files
   - `update`: enforce signed commits
3. **Managed alternatives:**
   - **pre-commit framework** (`pip install pre-commit`): declarative, cross-language
     ```yaml
     # .pre-commit-config.yaml
     repos:
       - repo: https://github.com/gitleaks/gitleaks
         rev: v8.18.0
         hooks:
           - id: gitleaks
       - repo: https://github.com/pre-commit/pre-commit-hooks
         rev: v4.5.0
         hooks:
           - id: check-yaml
           - id: end-of-file-fixer
           - id: trailing-whitespace
     ```
4. **Hands-on exercise:** Set up `pre-commit` with gitleaks, prettier, and a YAML linter, then try to commit a fake AWS key and a malformed YAML â€” confirm both are blocked.

#### Step 10: Signed Commits, Sigstore & the Supply Chain
1. **GPG-sign your commits:**
   ```bash
   gpg --full-generate-key
   git config --global user.signingkey <KEY_ID>
   git config --global commit.gpgsign true
   git config --global tag.gpgsign true
   git commit -S -m "signed commit"
   git verify-commit HEAD
   ```
2. **SSH-sign your commits (newer, simpler):**
   ```bash
   git config --global gpg.format ssh
   git config --global user.signingkey ~/.ssh/id_ed25519.pub
   git config --global commit.gpgsign true
   ```
3. **Sigstore â€” the future of artifact signing (cosign, gitsign):**
   ```bash
   # Sign a container image
   cosign sign --key cosign.key ghcr.io/myorg/myapp:1.2.3
   cosign verify --key cosign.pub ghcr.io/myorg/myapp:1.2.3
   ```
4. **SLSA â€” Supply-chain Levels for Software Artifacts:** a framework for attesting build provenance. Level 3 = hermetic, isolated builds with signed provenance.
5. **Hands-on exercise:** Set up SSH signing, push a signed commit, and verify it on GitHub (the `Verified` badge appears).

#### Step 11: Releases, Tags & Versioning
1. **Annotated tags vs lightweight:**
   ```bash
   git tag v1.0.0                       # lightweight (just a pointer)
   git tag -a v1.0.0 -m "Release 1.0"   # annotated (full object)
   git tag --list -l "v1.*"
   git push origin v1.0.0
   git push origin --tags
   ```
2. **Semantic Versioning â€” the contract:**
   ```
   MAJOR.MINOR.PATCH
   1.0.0 â†’ first stable
   1.1.0 â†’ backward-compatible features
   1.1.1 â†’ backward-compatible bug fix
   2.0.0 â†’ breaking change
   ```
3. **Auto-versioning with semantic-release:**
   ```bash
   npx semantic-release
   # reads Conventional Commits, bumps version, generates CHANGELOG, creates GH release
   ```
4. **GitHub Releases â€” the human face:**
   ```bash
   gh release create v1.0.0 --generate-notes ./dist/*
   gh release list
   ```
5. **Hands-on exercise:** Use `gh release create` to ship a binary, attach a `SHA256SUMS` file, and write a signing key fingerprint into the release notes.

#### Step 12: GitHub CLI â€” The Missing Native Tool
1. **Auth, repos, issues, PRs:**
   ```bash
   gh auth login
   gh repo clone myorg/myapp
   gh repo create myorg/newapp --public --source=. --remote=origin --push
   gh issue create --title "bug" --body "steps to repro" --label bug
   gh issue list --assignee @me
   gh pr create --fill --base main
   gh pr list --search "is:open review:required"
   gh pr checkout 42
   gh pr review 42 --approve
   gh pr merge 42 --squash --delete-branch
   gh run watch
   gh workflow run ci.yml
   ```
2. **Aliases â€” your own super-commands:**
   ```bash
   gh alias set prs 'pr list --search "is:open author:@me"'
   gh alias set mine 'pr list --assignee @me'
   ```
3. **Hands-on exercise:** Use `gh` to clone, open a PR, add a review comment, merge with squash, and delete the branch â€” all from the terminal.

#### Step 13: Disaster Recovery & Forensics
1. **Undo any mistake:**
   ```bash
   # undo last commit, keep changes staged
   git reset --soft HEAD~1
   # undo last commit, keep changes unstaged
   git reset HEAD~1
   # undo last commit, throw away changes
   git reset --hard HEAD~1
   # revert a pushed commit
   git revert <sha>
   # restore a deleted file from history
   git log --all --full-history -- path/to/file
   git checkout <sha>^ -- path/to/file
   ```
2. **Find the commit that deleted a line:**
   ```bash
   git log -S "the_missing_line" --oneline
   git log -p -S "the_missing_line"   # show the diff
   ```
3. **Find who knows what â€” `git blame`:**
   ```bash
   git blame -L 10,30 file.go
   git blame -C file.go               # detect moved code
   ```
4. **Clean a history of secrets (when prevention failed):**
   - `git filter-repo` is the modern, safe tool (not the old `filter-branch`)
   - Or use BFG Repo-Cleaner: `bfg --delete-files id_rsa`
   - After: rotate the secret, force-push, notify all integrators
5. **Hands-on exercise:** Use `git log -S` to find when a specific string was added and removed across a repo's history.

#### Step 14: GitOps â€” The Philosophy
1. **The GitOps principles (Weaveworks / OpenGitOps):**
   1. **Declarative** â€” the entire system is described declaratively (Terraform, Helm, Kustomize)
   2. **Versioned and immutable** â€” stored in Git, never edited live
   3. **Pulled automatically** â€” agents pull desired state from Git (ArgoCD, Flux)
   4. **Continuously reconciled** â€” drift between Git and live is detected and corrected
2. **Push vs pull deployment:**
   - **Push (CI):** CI server runs `kubectl apply` â€” works, but CI has prod credentials
   - **Pull (GitOps):** in-cluster agent reconciles â€” safer, audit trail in Git
3. **The repo structure of a GitOps platform:**
   ```
   infra-gitops/
   â”œâ”€â”€ apps/
   â”‚   â”œâ”€â”€ myapp/
   â”‚   â”‚   â”œâ”€â”€ base/             # Kustomize base
   â”‚   â”‚   â””â”€â”€ overlays/
   â”‚   â”‚       â”œâ”€â”€ dev/
   â”‚   â”‚       â”œâ”€â”€ staging/
   â”‚   â”‚       â””â”€â”€ prod/
   â”œâ”€â”€ infra/
   â”‚   â”œâ”€â”€ terraform-aws/        # Terraform for the cloud
   â”‚   â””â”€â”€ terraform-k8s/        # Crossplane / providers
   â””â”€â”€ clusters/
       â”œâ”€â”€ prod-east/
       â””â”€â”€ prod-west/
   ```
4. **Hands-on exercise:** Sketch the GitOps repo for a 3-service platform and identify the 4 repos vs monorepo tradeoff.

#### Step 15: Production-Grade Git Habits
- **Never commit secrets** â€” pre-commit hook with gitleaks, push protection on the platform
- **Never rebase published history** â€” `git push --force-with-lease` is your only safe force
- **Small PRs, fast review** â€” < 400 lines, < 24h SLA
- **CI must be green before review** â€” protect reviewers from formatting fights
- **Document the branching model in `CONTRIBUTING.md`** â€” every new hire reads it
- **Tag every release, sign every release, generate a changelog every release**
- **Mirror the repo to a second platform** for DR (GH is not the only place your code should live)
- **Practice a restore from backup at least once a year** â€” refs, packfiles, LFS objects, and the bare mirror

### Practice Project â€” Production-Grade Git Workflow

> **Goal:** Build a multi-service monorepo with enforced quality gates, signed commits, automated releases, and a documented recovery procedure.
> **Time estimate:** 8-10 hours
> **Deliverables:** A working GitHub repo, a passing CI pipeline, signed releases, and `CONTRIBUTING.md` + `RUNBOOK.md`.

#### Phase 1 â€” Repository & Branching (1h)
- [ ] **(B)** Initialize a monorepo `devops-mastery/` with three services: `api` (Node), `worker` (Python), `web` (Go)
- [ ] **(B)** Add `.gitignore`, `.editorconfig`, `.gitattributes`, `CODEOWNERS`, `CONTRIBUTING.md`
- [ ] **(I)** Adopt a branching model (trunk-based + short-lived feature branches) and document it
- [ ] **(I)** Create a `main` branch with a first commit and protect it on GitHub

#### Phase 2 â€” Hooks & Quality (1.5h)
- [ ] **(I)** Install `pre-commit` with gitleaks, prettier, shellcheck, hadolint, ruff â€” Step 9
- [ ] **(I)** Try to commit a fake AWS key + a malformed YAML â€” confirm both are blocked
- [ ] **(I)** Enable Dependabot for npm, pip, go modules, and Docker â€” Step 6
- [ ] **(I)** Open a PR that updates a dependency; confirm Dependabot groups and rebases

#### Phase 3 â€” CI/CD & Signing (2h)
- [ ] **(I)** Write a reusable GitHub Actions workflow: lint â†’ test â†’ build â†’ sign (cosign) â€” Step 6
- [ ] **(I)** Set up SSH-signed commits, then make 5 commits and verify each on GitHub â€” Step 10
- [ ] **(I)** Verify that the cosign signature validates on the container image

#### Phase 4 â€” Releases & GitOps (1.5h)
- [ ] **(I)** Configure semantic-release to auto-tag on merge to `main` â€” Step 11
- [ ] **(I)** Make 5 Conventional Commits and watch a version tag + release be created
- [ ] **(I)** Generate a CHANGELOG.md and confirm every entry is from a commit
- [ ] **(I)** Sketch the GitOps repo structure (apps/ infra/ clusters/) for this monorepo â€” Step 14

#### Phase 5 â€” Operations & Forensics (1.5h)
- [ ] **(I)** Use `git bisect` to find a planted bug in `worker/` â€” Step 3
- [ ] **(A)** Use `git filter-repo` to scrub a planted secret, then rotate and re-push â€” Step 13
- [ ] **(A)** Mirror the repo to a second VM; document the push/pull DR procedure
- [ ] **(A)** Use `git log -S` to find when a specific string was added/removed

#### Phase 6 â€” Documentation (1h)
- [ ] **(I)** Write `CONTRIBUTING.md` covering: branching, commit format, PR template, review SLA
- [ ] **(I)** Write a one-page runbook: "How to recover from a force-push that lost work" using the reflog â€” Step 13
- [ ] **(A)** Set up a `gh` alias set with: `prs`, `mine`, `ready`, `released` â€” Step 12

#### Bonus Challenges
- [ ] **(A)** Set up a private package registry (GH Packages) and publish a signed library
- [ ] **(A)** Implement Signed-off-by trailers (DCO) and enforce in CI
- [ ] **(A)** Configure branch protection with required checks, code owners, signed commits, and linear history

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 22 tasks across 6 phases + 3 bonus challenges**

### Key Takeaways
- Git is a snapshot-based DAG, not a diff system â€” once you see that, the rest clicks
- The three daily trees (working / staging / repo) and the reflog are your safety net
- Pull requests are the unit of review, audit, and compliance â€” small, scoped, and signed
- Branching strategy is a team decision; trunk-based + feature flags is the modern default
- GitHub (and its siblings) is a full platform: Actions, Packages, Code Scanning, Releases
- GitOps â€” declarative + versioned + pulled + reconciled â€” is the modern way to deploy everything

---

## ðŸ Python for DevOps

**Difficulty:** Beginner | **Time:** 25-30 hours

### Overview
Python is the Swiss-army knife of DevOps. It writes the glue between APIs that have no native integration, automates the boring 90% of operational work, ships CLI tools, powers most observability agents, and underpins tools like Ansible, AWS CDK, Pulumi's automation API, Apache Airflow, and Fabric. This tool takes you from idiomatic Python to production-grade scripting, packaging, testing, and async automation against real cloud APIs.

### Why Learn This
- Every cloud provider has a first-class Python SDK (boto3, azure-sdk, google-cloud)
- Most CLI tools in DevOps are written in Python (pip, ansible, aws-cli, gcloud, terraform, kubectl plugins)
- The "operator pattern" in Kubernetes is Python code reacting to cluster state
- Data engineering, log parsing, and ETL pipelines default to Python
- Knowing Python lets you read, patch, and contribute back to the tools you use

### Learn by Doing â€” Step by Step

#### Step 1: The Pythonic Mindset
1. **The Zen of Python â€” read it weekly:**
   ```bash
   python3 -c "import this"
   ```
2. **Idioms you will write 1000 times:**
   ```python
   # Unpacking
   first, *middle, last = [1, 2, 3, 4, 5]

   # Looping with index
   for i, item in enumerate(items, start=1):
       ...

   # Dict iteration
   for k, v in cfg.items():
       ...

   # Comprehensions (use them, but never nested > 2)
   squares = [x * x for x in range(10)]
   by_host = {h["name"]: h for h in hosts}

   # Context managers
   with open("/var/log/app.log") as f:
       for line in f:
           ...

   # The walrus operator (3.8+)
   while (line := f.readline()):
       process(line)

   # Structural pattern matching (3.10+)
   match event:
       case {"type": "created", "resource": r}:
           ...
       case {"type": "deleted"}:
           ...
   ```
3. **The type system â€” start using hints on day one:**
   ```python
   from typing import Iterable

   def healthy(hosts: Iterable[str]) -> bool:
       return all(is_up(h) for h in hosts)
   ```
4. **Hands-on exercise:** Refactor a 30-line shell script into idiomatic Python with types, docstrings, and tests.

#### Step 2: The Standard Library â€” The Real Power
1. **Pathlib over `os.path`:**
   ```python
   from pathlib import Path
   p = Path("/var/log/app")
   p.mkdir(parents=True, exist_ok=True)
   for f in p.glob("*.log"):
       f.rename(f.with_suffix(".log.gz"))
   ```
2. **Subprocess â€” the right way:**
   ```python
   import subprocess
   r = subprocess.run(
       ["kubectl", "get", "pods", "-A"],
       capture_output=True, text=True, timeout=30, check=True
   )
   print(r.stdout)
   ```
3. **Logging â€” never use `print` in production:**
   ```python
   import logging
   logging.basicConfig(
       level=logging.INFO,
       format="%(asctime)s %(levelname)s %(name)s %(message)s",
   )
   log = logging.getLogger(__name__)
   log.info("starting", extra={"host": host})
   ```
4. **argparse â†’ click â†’ typer (the evolution):**
   ```python
   # click
   import click
   @click.command()
   @click.option("--env", required=True, type=click.Choice(["dev", "prod"]))
   @click.option("--dry-run", is_flag=True)
   def deploy(env: str, dry_run: bool):
       """Deploy the service."""
       click.echo(f"deploying to {env} (dry={dry_run})")
   ```
5. **Hands-on exercise:** Write a CLI tool that takes `--config path.yml`, validates the YAML, and prints a JSON summary.

#### Step 3: Packaging & Distribution â€” The Professional Way
1. **`pyproject.toml` (PEP 621) â€” the modern manifest:**
   ```toml
   [build-system]
   requires = ["hatchling"]
   build-backend = "hatchling.build"

   [project]
   name = "devops-cli"
   version = "1.2.3"
   requires-python = ">=3.10"
   dependencies = ["click>=8.1", "pyyaml>=6", "boto3>=1.34"]

   [project.optional-dependencies]
   aws = ["boto3>=1.34"]

   [project.scripts]
   deploy = "devops_cli:main"
   ```
2. **Build and publish:**
   ```bash
   python3 -m venv .venv && source .venv/bin/activate
   pip install -e .                  # editable install
   pip install build twine
   python -m build
   twine upload dist/*
   ```
3. **Version pinning â€” the recipe:**
   ```bash
   pip freeze > requirements.txt                  # snapshot
   pip-compile requirements.in -o requirements.txt  # from pip-tools
   ```
4. **Hands-on exercise:** Package a small CLI and install it with `pip install -e .`; confirm the entry point is on your `PATH`.

#### Step 4: Testing â€” The Discipline
1. **pytest â€” the de facto standard:**
   ```python
   # tests/test_deploy.py
   import pytest
   from devops_cli import deploy

   def test_deploy_dry_run_does_nothing(monkeypatch, tmp_path):
       called = []
       monkeypatch.setattr(deploy, "apply_terraform",
                           lambda *a, **kw: called.append(a))
       deploy.run(env="dev", dry_run=True, dir=tmp_path)
       assert called == []
   ```
2. **Fixtures, parametrize, and the `tmp_path` magic:**
   ```python
   @pytest.fixture
   def aws_credentials(monkeypatch):
       monkeypatch.setenv("AWS_ACCESS_KEY_ID", "testing")
       monkeypatch.setenv("AWS_SECRET_ACCESS_KEY", "testing")
       monkeypatch.setenv("AWS_SECURITY_TOKEN", "testing")
       monkeypatch.setenv("AWS_SESSION_TOKEN", "testing")

   @pytest.mark.parametrize("n,expected", [(1, 1), (2, 4), (3, 9)])
   def test_square(n, expected):
       assert n * n == expected
   ```
3. **Mocking external services with `responses` (HTTP) and `moto` (AWS):**
   ```python
   import responses, requests

   @responses.activate
   def test_get_users():
       responses.add(responses.GET, "https://api.example.com/users",
                     json=[{"id": 1}], status=200)
       r = requests.get("https://api.example.com/users")
       assert r.json() == [{"id": 1}]
   ```
4. **Property-based testing with Hypothesis:**
   ```python
   from hypothesis import given, strategies as st
   @given(st.lists(st.integers()))
   def test_sort_is_idempotent(xs):
       s1 = sorted(xs)
       s2 = sorted(s1)
       assert s1 == s2
   ```
5. **Coverage that matters:**
   ```bash
   pytest --cov=src --cov-report=term-missing --cov-fail-under=80
   ```
6. **Hands-on exercise:** Write tests for a YAML-loading function with parametrize, moto, and a Hypothesis property test.

#### Step 5: HTTP, APIs & The Cloud SDKs
1. **requests vs httpx (sync vs async):**
   ```python
   import httpx
   r = httpx.get("https://api.github.com/repos/python/cpython",
                 timeout=10.0)
   r.raise_for_status()
   data = r.json()
   ```
2. **Retry with exponential backoff â€” use `tenacity`:**
   ```python
   from tenacity import retry, stop_after_attempt, wait_exponential
   @retry(stop=stop_after_attempt(5), wait=wait_exponential(min=1, max=20))
   def fetch(url):
       r = httpx.get(url, timeout=10)
       r.raise_for_status()
       return r.json()
   ```
3. **boto3 â€” AWS â€” the model:**
   ```python
   import boto3
   ec2 = boto3.client("ec2", region_name="us-east-1")
   paginator = ec2.get_paginator("describe_instances")
   for page in paginator.paginate(Filters=[{"Name": "tag:Env", "Values": ["prod"]}]):
       for r in page["Reservations"]:
           for i in r["Instances"]:
               print(i["InstanceId"], i["State"]["Name"])
   ```
4. **Pagination, the universal pattern:**
   ```python
   def paginate(client, method, **kwargs):
       paginator = client.get_paginator(method)
       for page in paginator.paginate(**kwargs):
           for item in page["Items"]:
               yield item
   ```
5. **Rate limiting with `aiolimiter` (or a token bucket you write yourself):**
   ```python
   from aiolimiter import AsyncLimiter
   limit = AsyncLimiter(50, 1)  # 50 req/sec
   async with limit:
       r = await client.get(url)
   ```
6. **Hands-on exercise:** Write a script that lists all EC2 instances tagged `Env=prod`, groups by AZ, and prints a table.

#### Step 6: Async I/O â€” When You Need Concurrency
1. **asyncio fundamentals:**
   ```python
   import asyncio, httpx

   async def fetch(client, url):
       r = await client.get(url, timeout=10)
       return r.json()

   async def main(urls):
       async with httpx.AsyncClient() as client:
           tasks = [fetch(client, u) for u in urls]
           return await asyncio.gather(*tasks)

   if __name__ == "__main__":
       results = asyncio.run(main([
           "https://api.github.com/repos/python/cpython",
           "https://api.github.com/repos/torvalds/linux",
       ]))
   ```
2. **The mental model:** one thread, many coroutines. I/O-bound = win. CPU-bound = use `ProcessPoolExecutor`.
3. **TaskGroup (3.11+) â€” structured concurrency:**
   ```python
   async with asyncio.TaskGroup() as tg:
       for url in urls:
           tg.create_task(fetch(client, url))
   ```
4. **Hands-on exercise:** Fetch 50 URLs in parallel and prove the wall-clock time is ~ 1x (not 50x) of a single request.

#### Step 7: YAML, JSON, TOML, ENV â€” Config That Doesn't Lie
1. **PyYAML â€” the landmines:**
   ```python
   import yaml
   # SAFE: only standard tags
   yaml.safe_load(open("config.yml"))
   # DANGEROUS: arbitrary object instantiation
   # yaml.load(open("config.yml"), Loader=yaml.Loader)
   ```
2. **The order of config precedence (12-factor):**
   ```
   defaults â†’ file â†’ env var â†’ CLI flag
   ```
3. **pydantic-settings â€” typed config from env + file:**
   ```python
   from pydantic import BaseModel
   from pydantic_settings import BaseSettings, SettingsConfigDict

   class Settings(BaseSettings):
       env: str = "dev"
       db_host: str
       db_port: int = 5432
       model_config = SettingsConfigDict(env_prefix="APP_")
   ```
4. **Hands-on exercise:** Build a settings class that reads from `config.yml`, env vars prefixed `APP_`, and CLI args, with a clear precedence.

#### Step 8: Working With the Operating System
1. **Subprocess patterns:**
   ```python
   import subprocess
   r = subprocess.run(["./deploy.sh", "prod"], check=True,
                      capture_output=True, text=True)
   # Streaming
   with subprocess.Popen(["./long.sh"], stdout=subprocess.PIPE, text=True) as p:
       for line in p.stdout:
           print(line, end="")
   ```
2. **Signal handling â€” the right way:**
   ```python
   import signal, sys
   def shutdown(signum, frame):
       print("shutting down")
       sys.exit(0)
   signal.signal(signal.SIGTERM, shutdown)
   ```
3. **File locking with `fcntl`:**
   ```python
   import fcntl
   with open("/var/lock/myapp.lock", "w") as f:
       fcntl.flock(f, fcntl.LOCK_EX | fcntl.LOCK_NB)
       # critical section
   ```
4. **Hands-on exercise:** Write a long-running daemon that traps SIGTERM, drains in-flight work, and exits 0 within 10 seconds.

#### Step 9: Observability for Python â€” OpenTelemetry
1. **The three signals, instrumented once:**
   ```python
   from opentelemetry import trace, metrics
   from opentelemetry.sdk.trace import TracerProvider
   from opentelemetry.sdk.metrics import MeterProvider
   from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
   from opentelemetry.sdk.trace.export import BatchSpanProcessor

   provider = TracerProvider()
   provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
   trace.set_tracer_provider(provider)
   tracer = trace.get_tracer(__name__)

   with tracer.start_as_current_span("deploy"):
       ...
   ```
2. **Structured logs with `structlog`:**
   ```python
   import structlog
   log = structlog.get_logger()
   log.info("user.created", user_id=u.id, tenant=t.id)
   ```
3. **Hands-on exercise:** Instrument a CLI with spans, export to Jaeger, and see the trace.

#### Step 10: Cloud Automation Patterns
1. **The idempotency contract â€” every operation must be safely re-runnable:**
   ```python
   def ensure_bucket(name, region):
       s3 = boto3.client("s3", region_name=region)
       try:
           s3.head_bucket(Bucket=name)
       except ClientError as e:
           if e.response["Error"]["Code"] == "404":
               s3.create_bucket(Bucket=name,
                   CreateBucketConfiguration={"LocationConstraint": region})
           else:
               raise
   ```
2. **The dry-run pattern:**
   ```python
   def destroy(env, dry_run=True):
       for stack in stacks_in(env):
           if dry_run:
               log.info("would destroy", stack=stack)
           else:
               cf.delete_stack(StackName=stack)
   ```
3. **The concurrency pattern â€” Terraform-style parallelism:**
   ```python
   from concurrent.futures import ThreadPoolExecutor
   with ThreadPoolExecutor(max_workers=10) as ex:
       list(ex.map(apply_one, changes))
   ```
4. **Hands-on exercise:** Build a tool that ensures 5 S3 buckets with versioning + encryption + lifecycle rules, and is safely re-runnable.

#### Step 11: Infrastructure as Data â€” Pulumi & AWS CDK
1. **Pulumi â€” Python IaC:**
   ```python
   import pulumi
   import pulumi_aws as aws

   bucket = aws.s3.Bucket("logs",
       versioning=aws.s3.BucketVersioningArgs(enabled=True),
       server_side_encryption_configuration=aws.s3.BucketServerSideEncryptionConfigurationArgs(
           rule=aws.s3.BucketServerSideEncryptionConfigurationRuleArgs(
               apply_server_side_encryption_by_default=aws.s3.BucketServerSideEncryptionConfigurationRuleApplyServerSideEncryptionByDefaultArgs(
                   sse_algorithm="AES256"))))
   pulumi.export("bucket_name", bucket.id)
   ```
2. **AWS CDK (Python):**
   ```python
   from aws_cdk import App, Stack, aws_s3 as s3
   class MyStack(Stack):
       def __init__(self, scope, id, **kwargs):
           super().__init__(scope, id, **kwargs)
           s3.Bucket(self, "Logs", versioned=True, encryption=s3.BucketEncryption.S3_MANAGED)
   app = App()
   MyStack(app, "MyStack")
   app.synth()
   ```
3. **When to use which:**
   - **Terraform:** declarative HCL, huge provider ecosystem, state in cloud
   - **Pulumi/CDK:** general-purpose language, easier abstractions, native testing
4. **Hands-on exercise:** Recreate the same S3 bucket + IAM policy in Terraform, Pulumi, and CDK â€” and compare.

#### Step 12: Production Tooling â€” Observability, Secrets, Errors
1. **Secrets â€” never in code, never in env, never in logs:**
   ```python
   import hvac  # HashiCorp Vault client
   client = hvac.Client(url="https://vault.internal:8200",
                        token=os.environ["VAULT_TOKEN"])
   secret = client.secrets.kv.v2.read_secret_version(path="db/prod")
   password = secret["data"]["data"]["password"]
   ```
2. **Retry, circuit breaker, and rate limiting â€” `tenacity` + `pybreaker` + `aiolimiter`:**
   ```python
   from pybreaker import CircuitBreaker
   breaker = CircuitBreaker(fail_max=5, reset_timeout=30)
   @breaker
   def call_external():
       ...
   ```
3. **The structured error hierarchy:**
   ```python
   class DeployError(Exception): pass
   class ConfigError(DeployError): pass
   class InfraError(DeployError): pass
   # catch narrowly
   try:
       deploy(cfg)
   except ConfigError as e:
       log.error("config bad", err=str(e)); sys.exit(2)
   except InfraError as e:
       log.exception("infra failed"); sys.exit(3)
   ```
4. **Hands-on exercise:** Wrap a flaky API in a circuit breaker and prove the breaker opens after 5 failures.

#### Step 13: Performance â€” When Python Slows You Down
1. **Profile first, optimize second:**
   ```bash
   python -X importtime myscript.py    # import time
   py-spy dump --pid 1234              # sampling profiler
   memray run -m myscript              # memory profiler
   ```
2. **The big wins:**
   - Use `lru_cache` for repeated pure calls
   - Use `orjson` instead of `json` (3-5x)
   - Use `numpy`/`pandas` for bulk numeric
   - Use `multiprocessing` (not threading) for CPU work
   - Move hot paths to Cython, Mypyc, or Rust (PyO3)
3. **Hands-on exercise:** Profile a 10k-row JSON parsing script, find the hotspot, and cut runtime in half.

#### Step 14: The Operator Pattern â€” Python Inside Kubernetes
1. **Kopf â€” Kubernetes Operator Pythonic Framework:**
   ```python
   import kopf
   @kopf.on.create("db.example.com", "v1", "postgreses")
   def create_fn(spec, **_):
       kopf.adopt(spec)  # mark for GC
       # create the underlying resources
       return {"child": "created"}
   ```
2. **The pattern:** a CRD + a controller = a custom API in your cluster.
3. **Hands-on exercise:** Define a `Website` CRD, write an operator that creates a Deployment + Service on `create` and deletes them on `delete`.

#### Step 15: Production-Grade Python Habits
- **Format with `ruff format`, lint with `ruff check`** (replaces black + isort + flake8)
- **Type check with `mypy --strict` or `pyright`**
- **Pin everything** in CI, install with `pip-compile`
- **Run with `python -O` and `PYTHONDONTWRITEBYTECODE=1` in containers**
- **Set `ulimit -n 65535` and `MALLOC_ARENA_MAX=1`** for high-concurrency services
- **Use gunicorn with `uvicorn.workers.UvicornWorker`** for ASGI
- **12-factor all the way**: logs to stdout, config from env, stateless processes
- **Write a healthcheck** (`/healthz` for liveness, `/readyz` for readiness) in every service

### Practice Project â€” Build a Production-Grade Python CLI

> **Goal:** Ship a real CLI tool â€” packaged, tested, containerized, signed, observable, with a Helm deployment.
> **Time estimate:** 12-15 hours
> **Deliverables:** A PyPI-ready package, a signed container image, a Helm chart, and `RUNBOOK.md`.

#### Phase 1 â€” CLI Skeleton (1.5h)
- [ ] **(B)** Set up `pyproject.toml` with hatchling, dependencies, scripts â€” Step 3
- [ ] **(B)** Use `click` to build subcommands: `deploy`, `rollback`, `status`, `logs` â€” Step 2
- [ ] **(I)** Use `pydantic-settings` for config (file + env + CLI precedence) â€” Step 7
- [ ] **(I)** Use `structlog` for structured JSON logs to stdout â€” Step 9

#### Phase 2 â€” Cloud Integration (2.5h)
- [ ] **(I)** Wire the `deploy` command to a real AWS account (or LocalStack) â€” Step 5
- [ ] **(I)** Implement ECS service update OR Lambda alias traffic shifting
- [ ] **(I)** Add `--dry-run` everywhere; print the plan without executing â€” Step 10
- [ ] **(I)** Add health checks and automatic rollback on failure

#### Phase 3 â€” Reliability (2h)
- [ ] **(I)** Add `tenacity` for retries with exponential backoff â€” Step 5
- [ ] **(I)** Add `pybreaker` circuit breaker for external API calls â€” Step 12
- [ ] **(I)** Add `aiolimiter` rate limiting for the cloud APIs
- [ ] **(I)** Implement signal handling for graceful shutdown â€” Step 8

#### Phase 4 â€” Testing (2h)
- [ ] **(I)** Write 30+ pytest tests with > 90% coverage â€” Step 4
- [ ] **(I)** Use `moto` for AWS mocking and `responses` for HTTP â€” Step 4
- [ ] **(I)** Add 5 Hypothesis property-based tests for pure functions
- [ ] **(I)** Add a CI workflow that runs pytest + coverage + mypy strict â€” Step 3

#### Phase 5 â€” Packaging & Distribution (1.5h)
- [ ] **(I)** Containerize with a multi-stage Dockerfile (slim â†’ distroless) â€” Step 3
- [ ] **(I)** Build with buildkit, scan with Trivy, fix any HIGH/CRITICAL
- [ ] **(I)** Sign the image with cosign and verify in CI â€” Step 5 (Python tool)
- [ ] **(I)** Publish a test release to Test PyPI

#### Phase 6 â€” Observability & Deployment (2h)
- [ ] **(I)** Instrument with OpenTelemetry traces and metrics, export to OTLP â€” Step 9
- [ ] **(I)** Add a Prometheus `/metrics` endpoint if the CLI has a serve mode
- [ ] **(I)** Write a Helm chart that runs the CLI as a CronJob â€” Step 14
- [ ] **(I)** Install the chart to a Kind cluster and verify the CronJob runs

#### Phase 7 â€” Documentation (1h)
- [ ] **(B)** Write a README with: install, quickstart, all commands, troubleshooting
- [ ] **(I)** Write `RUNBOOK.md` with: install, rollback, debug, on-call escalation
- [ ] **(A)** Benchmark with `hyperfine` and document performance in the README
- [ ] **(A)** Add a CHANGELOG.md following Keep a Changelog format

#### Bonus Challenges
- [ ] **(A)** Add a Pulumi program in Python that creates the same AWS resources â€” Step 11
- [ ] **(A)** Build a Kubernetes operator with Kopf that creates a CRD-driven deploy â€” Step 14
- [ ] **(A)** Profile a 10k-row JSON parser, find the hotspot, cut runtime in half â€” Step 13

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 25 tasks across 7 phases + 3 bonus challenges**

### Key Takeaways
- Python's standard library + a small set of focused deps (click, pydantic, httpx, tenacity, structlog) is enough to ship production tooling
- Idempotency, dry-run, structured logs, retries, and tests are non-negotiable for any operational script
- The cloud SDKs (boto3, azure-sdk, google-cloud) all follow the same pagination + waiter pattern
- Pulumi and CDK let you write IaC in Python when HCL is too limiting
- Python is the language of Kubernetes operators â€” the future of platform engineering

---

## ðŸ’» Bash & Shell Scripting Mastery

**Difficulty:** Intermediate | **Time:** 20-25 hours

### Overview
Even in 2026, Bash is the lingua franca of the data center, the cloud-init, the CI runner, the container entrypoint, and the operator's escape hatch. Every Dockerfile ends in `CMD ["/bin/sh", "-c", ...]`. Every Kubernetes debug session begins in a shell. Every Ansible task is a thin wrapper around a command. Mastering Bash â€” the modern, safe, portable kind â€” is the difference between scripts that rot and scripts that run unchanged for a decade.

### Why Learn This
- Shell is the only language that runs identically on every Linux, in every container, in every CI
- The cost of a bug in production shell is huge (think `rm -rf` + unquoted variable)
- Most "glue code" in real systems is still 50â€“500 lines of Bash â€” read it, fix it, write it
- Mastering Bash makes you fluent in `set -euo pipefail`, traps, mktemp, and the right way to parse args
- Knowing both Bash and Python lets you pick the right tool per job

### Learn by Doing â€” Step by Step

#### Step 1: The Non-Negotiable Setup
1. **Every production script starts with these lines:**
   ```bash
   #!/usr/bin/env bash
   set -Eeuo pipefail
   IFS=$'\n\t'

   # Even better: detect unset vars vs empty explicitly
   # set -u does this already in modern bash
   ```
2. **What each flag does:**
   - `-E` â€” inherit ERR trap into functions and subshells
   - `-e` â€” exit on any non-zero (use `|| true` to opt out per command)
   - `-u` â€” error on unset variables
   - `-o pipefail` â€” fail the pipeline if any command fails (not just the last)
3. **Trap everything, exit clean:**
   ```bash
   cleanup() {
     local rc=$?
     rm -rf "${TMPDIR:-}"
     exit "$rc"
   }
   trap cleanup EXIT
   trap 'echo "interrupted" >&2; exit 130' INT TERM
   ```
4. **Hands-on exercise:** Write a 20-line script that creates a tempdir, touches a file inside, then cleans up on exit. Verify with `ls` that the tempdir is gone after success, error, and SIGINT.

#### Step 2: Variables, Quoting, and Expansions
1. **Always quote. Always. Except when you intentionally glob.**
   ```bash
   name="world"
   echo "Hello, $name"       # safe
   echo "Hello, ${name}"     # safer in long strings
   echo "$unquoted"          # WRONG â€” word splitting + glob
   echo "$quoted"            # right
   ```
2. **The 8 expansions you'll use daily:**
   | Expansion | What it does | Example |
   |-----------|--------------|---------|
   | `${var}` | value | `${HOME}` |
   | `${var:-default}` | default if unset | `${ENV:-dev}` |
   | `${var:=default}` | set default if unset | `${TAG:=latest}` |
   | `${var:?msg}` | error if unset | `${DB_HOST:?required}` |
   | `${var#prefix}` | remove shortest prefix | `${f##*/}` (basename) |
   | `${var%suffix}` | remove shortest suffix | `${f%.log}` (strip .log) |
   | `${var/from/to}` | replace first | `${path//\/\//\/}` |
   | `$(cmd)` | command substitution | `$(date +%F)` |
3. **Arrays â€” the underused feature:**
   ```bash
   arr=(a b c)
   arr+=("d")
   echo "${arr[0]}" "${arr[@]}" "${#arr[@]}"
   for x in "${arr[@]}"; do ...; done
   ```
4. **Hands-on exercise:** Write a script that takes a comma-separated list and prints each element on its own line, handling empty input gracefully.

#### Step 3: Conditionals, Loops & Functions
1. **The right test syntax â€” `[[ ]]`, not `[ ]`:**
   ```bash
   if [[ -f "$f" && -r "$f" ]]; then ...
   if [[ "$str" =~ ^[0-9]+$ ]]; then ...   # regex match
   if [[ "$a" == "x" || "$b" == "y" ]]; then ...
   ```
2. **Case â€” the idiomatic dispatcher:**
   ```bash
   case "$1" in
     dev|staging|prod) ENV=$1 ;;
     -h|--help) usage; exit 0 ;;
     *) echo "unknown: $1" >&2; usage; exit 2 ;;
   esac
   ```
3. **Functions â€” `local` is non-negotiable:**
   ```bash
   log() {
     local level="$1"; shift
     printf '%s [%s] %s\n' "$(date -Is)" "$level" "$*"
   }
   ```
4. **The trap-as-flag pattern:**
   ```bash
   DEBUG=${DEBUG:-0}
   debug() { [[ "$DEBUG" == "1" ]] && echo "DEBUG: $*" >&2 || true; }
   DEBUG=1 ./script.sh
   ```
5. **Hands-on exercise:** Refactor a 50-line imperative script into well-named functions with local variables and a main dispatch.

#### Step 4: Parsing Arguments â€” Without Reinventing getopt
1. **Manual parsing â€” fine for < 5 options:**
   ```bash
   DRY=0; VERBOSE=0
   while [[ $# -gt 0 ]]; do
     case "$1" in
       --env) ENV=$2; shift 2 ;;
       --env=*) ENV=${1#*=}; shift ;;
       --dry-run) DRY=1; shift ;;
       -v|--verbose) VERBOSE=1; shift ;;
       -h|--help) usage; exit 0 ;;
       --) shift; break ;;
       -*) echo "unknown: $1" >&2; exit 2 ;;
       *) ARGS+=("$1"); shift ;;
     esac
   done
   ```
2. **getopts â€” for short options only:**
   ```bash
   while getopts ":e:vh" opt; do
     case $opt in
       e) ENV=$OPTARG ;;
       v) VERBOSE=1 ;;
       h) usage; exit 0 ;;
       \?) echo "invalid: -$OPTARG" >&2 ;;
     esac
   done
   shift $((OPTIND-1))
   ```
3. **Hands-on exercise:** Implement `--env`, `--tag`, `--dry-run`, `--verbose` with both manual and `getopts` versions; compare.

#### Step 5: Safe File & Process Operations
1. **Always use `mktemp`:**
   ```bash
   TMPDIR=$(mktemp -d -t myapp.XXXXXX)
   TMPFILE=$(mktemp -t myapp.XXXXXX)
   trap 'rm -rf "$TMPDIR"' EXIT
   ```
2. **Atomic writes â€” write-then-rename:**
   ```bash
   write_atomic() {
     local target=$1
     local tmp
     tmp=$(mktemp "$(dirname "$target")/.${target##*/}.XXXXXX")
     cat > "$tmp"
     chmod 0644 "$tmp"
     mv -f "$tmp" "$target"
   }
   ```
3. **Locks with `flock`:**
   ```bash
   exec 9>/var/lock/myapp.lock
   flock -n 9 || { echo "another instance running" >&2; exit 1; }
   # critical section
   ```
4. **Robust process pipelines:**
   ```bash
   while IFS= read -r line; do
     process "$line"
   done < <(generate_lines)
   # NEVER: for line in $(cmd) â€” word splitting
   ```
5. **Hands-on exercise:** Write a script that safely updates `/etc/myapp.conf` atomically, locks against concurrent runs, and survives being killed mid-write.

#### Step 6: Interacting With JSON, YAML & Cloud CLIs
1. **JSON with `jq` â€” the indispensable tool:**
   ```bash
   aws ec2 describe-instances \
     | jq -r '.Reservations[].Instances[]
             | select(.State.Name == "running")
             | "\(.InstanceId) \(.PrivateIpAddress) \(.Tags[]? | select(.Key=="Name") | .Value)"'
   ```
2. **YAML with `yq`:**
   ```bash
   yq '.spec.replicas' deployment.yaml
   yq -i '.spec.template.spec.containers[0].image = "myapp:1.2.4"' deployment.yaml
   ```
3. **Iterate cloud resources safely:**
   ```bash
   while read -r id; do
     [[ -n "$id" ]] || continue
     echo "deleting $id"
     aws ec2 terminate-instances --instance-ids "$id" >/dev/null
   done < <(aws ec2 describe-instances \
              --filters "Name=tag:Env,Values=dev" \
                        "Name=instance-state-name,Values=running" \
              --query 'Reservations[].Instances[].InstanceId' \
              --output text | tr '\t' '\n')
   ```
4. **Hands-on exercise:** Write a `cost-report.sh` that uses `aws ce get-cost-and-usage` + `jq` to produce a 7-day cost per service table.

#### Step 7: The Container-Aware Script
1. **Detect you're in a container:**
   ```bash
   in_container() {
     [[ -f /.dockerenv ]] || \
     grep -qE '/(docker|lxc|containerd)' /proc/1/cgroup 2>/dev/null
   }
   ```
2. **Use `tini` or `dumb-init` as PID 1:**
   ```dockerfile
   RUN apk add --no-cache tini
   ENTRYPOINT ["/sbin/tini", "--"]
   CMD ["./entrypoint.sh"]
   ```
3. **The init script pattern â€” trap, wait, signal:**
   ```bash
   #!/usr/bin/env bash
   set -Eeuo pipefail
   svc1 & PID1=$!
   svc2 & PID2=$!
   shutdown() {
     echo "shutting down"
     kill -TERM "$PID1" "$PID2" 2>/dev/null || true
     wait "$PID1" "$PID2" || true
   }
   trap shutdown TERM INT
   wait -n
   exit $?
   ```
4. **Hands-on exercise:** Write a container entrypoint that starts two background processes, traps SIGTERM, and exits 0 when both have drained.

#### Step 8: Networking in Bash
1. **The toolkit:**
   ```bash
   curl -fsSL -o /dev/null -w "%{http_code}\n" https://example.com
   curl --retry 5 --retry-delay 2 --retry-all-errors https://example.com
   jq -e '.ok' < <(curl -s https://api.example.com/health)
   ```
2. **SSH for automation â€” keys, not passwords, ever:**
   ```bash
   ssh -o StrictHostKeyChecking=accept-new \
       -o UserKnownHostsFile=/dev/null \
       -o BatchMode=yes \
       deploy@host "systemctl restart myapp"
   ```
3. **Parallel SSH with `pssh` or `xargs -P`:**
   ```bash
   cat hosts.txt | xargs -n1 -P20 -I{} ssh {} "uptime"
   ```
4. **The TCP health check that doesn't lie:**
   ```bash
   timeout 5 bash -c 'cat < /dev/tcp/db.internal/5432' || echo "DOWN"
   ```
5. **Hands-on exercise:** Build a script that fans out a deploy across 10 hosts in parallel, waits for all, and rolls back on any failure.

#### Step 9: Production-Grade Bash Patterns
1. **The logger:**
   ```bash
   readonly LOG_FILE=/var/log/myapp.log
   log() {
     local level=$1; shift
     printf '%s [%s] [%d] %s\n' "$(date -Is)" "$level" "$$" "$*" \
       | tee -a "$LOG_FILE" >&2
   }
   ```
2. **Locking + heartbeat for long jobs:**
   ```bash
   exec 9>/var/lock/job.lock
   flock 9
   (
     while :; do
       date -Is > /var/run/job.heartbeat
       sleep 30
     done
   ) &
   HEARTBEAT_PID=$!
   trap 'kill $HEARTBEAT_PID 2>/dev/null || true; flock -u 9' EXIT
   ```
3. **Structured output for log aggregation:**
   ```bash
   log_json() {
     jq -nc --arg ts "$(date -Is)" --arg msg "$*" \
        '{ts: $ts, level: "info", msg: $msg}'
   }
   ```
4. **Idempotency â€” check before change:**
   ```bash
   ensure_user() {
     local user=$1
     id "$user" &>/dev/null || useradd -m -s /bin/bash "$user"
   }
   ```
5. **A real-world script skeleton â€” the production template:**
   ```bash
   #!/usr/bin/env bash
   set -Eeuo pipefail
   IFS=$'\n\t'
   readonly PROG=$(basename "$0")
   readonly VERSION=1.0.0

   usage() { cat <<EOF
   Usage: $PROG [options] <arg>
     -e, --env   ENV    Environment (dev|staging|prod)
     -d, --dry-run      Print actions without executing
     -v, --verbose      Increase output verbosity
     -h, --help         Show this help
     -V, --version      Show version
   EOF
   }

   main() {
     local dry_run=0 verbose=0 env= arg=
     while [[ $# -gt 0 ]]; do
       case "$1" in
         -e|--env) env=$2; shift 2 ;;
         -d|--dry-run) dry_run=1; shift ;;
         -v|--verbose) verbose=1; shift ;;
         -h|--help) usage; exit 0 ;;
         -V|--version) echo "$PROG $VERSION"; exit 0 ;;
         --) shift; break ;;
         -*) echo "unknown: $1" >&2; usage >&2; exit 2 ;;
         *) arg=$1; shift ;;
       esac
     done

     : "${env:?--env is required}"
     : "${arg:?arg is required}"

     log_info "starting" env="$env" arg="$arg" dry_run="$dry_run"

     # ... work ...

     log_info "done"
   }

   log_info() { log INFO "$@"; }
   log_warn() { log WARN "$@" >&2; }
   log_error() { log ERROR "$@" >&2; }
   log() {
     local level=$1; shift
     printf '{"ts":"%s","level":"%s","prog":"%s","msg":"%s"}\n' \
       "$(date -Is)" "$level" "$PROG" "$*" >&2
   }

   main "$@"
   ```
6. **Hands-on exercise:** Take the template and implement a working `backup-rotate.sh` that takes `--src`, `--dest`, `--keep N`, `--dry-run`.

#### Step 10: Testing Bash (Yes, You Can)
1. **bats â€” the de facto standard:**
   ```bash
   # tests/backup.bats
   @test "backup creates a file" {
     run ./backup.sh --src /etc --dest /tmp/bak --keep 1
     [ "$status" -eq 0 ]
     [ -f /tmp/bak/etc.tar.gz ]
   }
   @test "missing --src fails" {
     run ./backup.sh
     [ "$status" -ne 0 ]
     [[ "$output" =~ "required" ]]
   }
   ```
2. **shellcheck â€” the linter that finds every bug you missed:**
   ```bash
   shellcheck -x script.sh
   # in CI
   find . -name "*.sh" -print0 | xargs -0 shellcheck
   ```
3. **shfmt â€” auto-format:**
   ```bash
   shfmt -i 2 -ci -w script.sh
   ```
4. **Hands-on exercise:** Write 10 bats tests for a script, run them in CI, add shellcheck + shfmt as required checks.

#### Step 11: Performance, Portability & Pitfalls
1. **The slow path traps:**
   - `for f in $(ls)` â€” never
   - `cat file | grep` â€” never
   - Spawning subshells in loops â€” almost never
   ```bash
   # WRONG
   while read line; do count=$(grep -c . <<<"$line"); done < file
   # RIGHT
   awk '{c++} END{print c}' file
   ```
2. **Portability check â€” what `#!/usr/bin/env bash` actually buys you:**
   - Always bash, never `/bin/sh` for scripts using arrays, `[[ ]]`, etc.
   - `dash` (the default `/bin/sh` on Debian) lacks them
3. **The 10 bugs that recur in every Bash codebase:**
   | Bug | Fix |
   |-----|-----|
   | Unquoted `$var` | Always quote |
   | Parsing `ls` output | Glob directly or use `find -print0` |
   | Missing `set -e` | Add it first line |
   | Missing `set -u` | Add it (or use `${var:-}`) |
   | Missing `set -o pipefail` | Add it |
   | `cd` without checking | `cd dir || exit` |
   | SIGTERM ignored in container | Trap, forward to child |
   | Race on temp files | `mktemp` |
   | Subshell modifying parent state | Use `{ ...; }` not `( ... )` |
   | `exit 0` masking real failure | Don't add `|| true` to silence errors |
4. **Hands-on exercise:** Take a real-world script (your own or a famous one), run shellcheck, fix everything, and explain each fix.

#### Step 12: Production-Grade Bash Habits
- **Lint with shellcheck, format with shfmt, test with bats** in CI
- **Use the template from Step 9** as a starting point for every script
- **Add a manpage** with `ronn` or `mdocml`; one page per script
- **Version the script** (`VERSION=1.2.3`) and expose `--version`
- **Log structured** (one JSON object per line) â€” log shippers love you
- **Trap INT TERM EXIT** â€” every script, every time
- **Pin the interpreter** with `#!/usr/bin/env bash`, not `#!/bin/bash`
- **Use `set -x` only in dev** (or behind a `DEBUG` flag), never in prod

### Practice Project â€” Build a Production Bash Toolkit

> **Goal:** Ship a small, tested, linted, versioned, container-ready toolkit of real-world scripts.
> **Time estimate:** 10-12 hours
> **Deliverables:** A GitHub repo with 6 scripts, tests, CI, and a signed container image.

#### Phase 1 â€” The Skeleton (1h)
- [ ] **(B)** Create repo `bash-toolkit/` with the Step 9 template as `lib/common.sh`
- [ ] **(B)** Set up shellcheck, shfmt, bats in pre-commit
- [ ] **(I)** Add a CI workflow (GitHub Actions) that runs them on every PR
- [ ] **(I)** Tag the repo with `v0.1.0` and write `CHANGELOG.md`

#### Phase 2 â€” Core Scripts (4h)
- [ ] **(I)** `backup-rotate.sh` â€” tar a source dir, rotate to keep N, atomic moves, locking â€” Step 9
- [ ] **(I)** `healthcheck.sh` â€” TCP probe, JSON output, exit 0/1/2 â€” Step 8
- [ ] **(I)** `cost-report.sh` â€” uses `aws ce` + `jq` to produce a 7-day per-service table â€” Step 6
- [ ] **(I)** `deploy-fanout.sh` â€” parallel SSH deploy to N hosts with rollback â€” Step 8
- [ ] **(A)** `k8s-prune.sh` â€” remove completed/failed Jobs and dangling Pods older than N hours
- [ ] **(A)** `config-render.sh` â€” render a templated config from env + YAML + CLI args â€” Step 7 (Python)

#### Phase 3 â€” Testing (2h)
- [ ] **(I)** Write 10+ bats tests per script (60+ total) â€” Step 10
- [ ] **(I)** Mock external commands (aws, ssh) with shell function overrides
- [ ] **(I)** Add a `make test` target that runs bats, shellcheck, shfmt -d
- [ ] **(A)** Add a mutation test (e.g., `mutmut` analog for bash) for one script

#### Phase 4 â€” Hardening (1.5h)
- [ ] **(I)** Every script: `--help`, `--version`, `--dry-run`, `--verbose` â€” Step 9
- [ ] **(I)** Every script: trap INT/TERM/EXIT, use mktemp, atomic writes
- [ ] **(I)** Every script: structured JSON logs to stderr
- [ ] **(A)** Add a `--diff` mode to the deploy script that shows what would change

#### Phase 5 â€” Distribution (1.5h)
- [ ] **(I)** Containerize: a small `bash:5.3-alpine` image with the toolkit at `/opt/bin` â€” Step 7
- [ ] **(I)** Multi-arch build (amd64 + arm64) with buildx
- [ ] **(I)** Sign with cosign, verify in CI
- [ ] **(I)** Push to GHCR; document `docker pull ghcr.io/you/bash-toolkit:1.0.0`

#### Phase 6 â€” Documentation (1h)
- [ ] **(B)** Write a README per script (usage, examples, exit codes)
- [ ] **(B)** Write a top-level README with the full table of contents
- [ ] **(I)** Generate manpages with `ronn` from the READMEs
- [ ] **(A)** Write `RUNBOOK.md`: install, troubleshoot, on-call escalation

#### Bonus Challenges
- [ ] **(A)** Add a TUI mode to `healthcheck.sh` using `whiptail` or `dialog`
- [ ] **(A)** Implement a state machine pattern in pure Bash for a long-running workflow
- [ ] **(A)** Write a Bash plugin for `kubectl` (e.g., `kubectl cost`) that uses the toolkit

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 22 tasks across 6 phases + 3 bonus challenges**

### Key Takeaways
- Bash is the connective tissue of every Linux system â€” master it and you master the data center
- `set -Eeuo pipefail` + trap + mktemp + quoting is the safety net
- The template in Step 9 is your starting point for every script, full stop
- bats + shellcheck + shfmt is the minimum CI for any serious Bash
- The right tool is sometimes Bash, sometimes Python â€” know when to switch

---

## ðŸ³ Docker

**Difficulty:** Intermediate | **Time:** 25-30 hours

### Overview
Docker popularized the container format that now runs the cloud. Every Kubernetes pod is a constellation of containers. Every CI runner spins up containers. Every modern application ships as an image. This tool takes you from `docker run` to multi-stage builds, from Compose stacks to rootless production deployments, from image security scanning to a reproducible supply chain with cosign and SLSA.

### Why Learn This
- Containers are the deployment artifact of every modern platform
- Docker (or its OCI-compatible siblings) is the first thing Kubernetes runs
- A bad image is a bad deploy â€” security, size, and reproducibility start here
- Compose is the fastest way to stand up a multi-service dev environment
- The same Dockerfile works on a laptop, a CI runner, and an EKS node

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **Containers vs VMs â€” the difference matters:**
   ```
   VM:    App | App | App  â†’  Guest OS | Guest OS | Guest OS  â†’  Hypervisor  â†’  Host OS  â†’  Hardware
   Cont:  App | App | App  â†’  Container Engine (runc)  â†’  Host OS  â†’  Hardware
   ```
2. **The OCI stack â€” what Docker actually is:**
   - **containerd:** the runtime (pulls, manages, supervises)
   - **runc:** the low-level OCI runtime (creates namespaces, cgroups)
   - **buildkit / BuildKit:** the image builder
   - **dockerd:** the daemon orchestrating them
3. **Your first container:**
   ```bash
   docker run --rm -d --name web -p 8080:80 nginx:1.27
   curl localhost:8080
   docker exec -it web sh
   docker logs -f web
   docker stop web
   ```
4. **The three pieces of state on disk:**
   - **Images** â€” read-only layers in `/var/lib/docker/overlay2/`
   - **Containers** â€” writable layer + namespace state
   - **Volumes** â€” persistent, host-backed or named
5. **Hands-on exercise:** Run `nginx`, `redis`, `postgres:16`, and `alpine`; exec into each; inspect `/etc/os-release`.

#### Step 2: Images & Registries
1. **The anatomy of an image:**
   ```bash
   docker pull nginx:1.27
   docker inspect nginx:1.27
   docker history nginx:1.27 --no-trunc
   # Layers are content-addressed (sha256)
   ```
2. **Image naming:**
   ```
   [registry/][namespace/]repository[:tag][@digest]
   ghcr.io/myorg/myapp:1.2.3
   ghcr.io/myorg/myapp@sha256:abcdef...
   ```
3. **The registries you will use:**
   - **Docker Hub:** `docker.io/...` (default, public, rate-limited)
   - **GitHub Container Registry:** `ghcr.io/...` (free for public)
   - **AWS ECR, Azure ACR, GCP Artifact Registry:** cloud-native, IAM-integrated
   - **Harbor, Quay, Registry:** self-hosted, enterprise
4. **Login & pull:**
   ```bash
   echo "$GITHUB_TOKEN" | docker login ghcr.io -u USERNAME --password-stdin
   docker pull ghcr.io/myorg/myapp:1.2.3
   docker pull --platform linux/arm64 myorg/myapp:1.2.3   # multi-arch
   ```
5. **Hands-on exercise:** Pull an image by digest (not tag) and prove the digest is the same on two machines.

#### Step 3: Writing a Production Dockerfile
1. **The multi-stage pattern â€” the single most important rule:**
   ```dockerfile
   # syntax=docker/dockerfile:1.10
   ARG GO_VERSION=1.23

   FROM golang:${GO_VERSION}-alpine AS build
   WORKDIR /src
   COPY go.mod go.sum ./
   RUN go mod download
   COPY . .
   RUN --mount=type=cache,target=/root/.cache/go-build \
       CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags='-s -w' -o /out/app .

   FROM gcr.io/distroless/static-debian12:nonroot
   COPY --from=build --chown=nonroot:nonroot /out/app /app
   USER nonroot:nonroot
   EXPOSE 8080
   ENTRYPOINT ["/app"]
   ```
2. **The 15 rules that make a Dockerfile good:**
   | # | Rule | Why |
   |---|------|-----|
   | 1 | Pin base image by digest | Reproducibility |
   | 2 | Use minimal base (`alpine`, `distroless`, `scratch`) | Smaller attack surface |
   | 3 | Multi-stage | Smaller final image |
   | 4 | `COPY` only what you need | Cache locality |
   | 5 | Order COPY from least-changed to most-changed | Cache hits |
   | 6 | Combine RUN with `&&` and clean in the same layer | Size |
   | 7 | `--no-install-recommends`, `apk add --no-cache` | Size |
   | 8 | `USER nonroot` (or a UID) | Runtime security |
   | 9 | `WORKDIR` not `cd` | Readable, reliable |
   | 10 | `EXPOSE` for documentation | Conventions |
   | 11 | `ENTRYPOINT` exec form `["/app"]` not shell | Signal handling |
   | 12 | `HEALTHCHECK` for orchestrators | Auto-restart decisions |
   | 13 | `LABEL` with metadata (org.opencontainers.image.*) | Discoverability |
   | 14 | Don't bake secrets | Use runtime mounts |
   | 15 | `.dockerignore` properly | Faster builds, smaller context |
3. **The .dockerignore that ships with most repos:**
   ```
   .git
   .github
   .vscode
   .idea
   **/.DS_Store
   **/node_modules
   **/__pycache__
   **/.venv
   **/.pytest_cache
   **/coverage
   **/dist
   **/build
   *.md
   tests/
   ```
4. **BuildKit features that change your life:**
   ```dockerfile
   # syntax=docker/dockerfile:1.10
   RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
       apt update && apt install -y --no-install-recommends build-essential
   RUN --mount=type=ssh \
       git clone git@github.com:private/repo.git
   RUN --mount=type=secret,id=github_token \
       curl -H "Authorization: token $(cat /run/secrets/github_token)" ...
   ```
5. **Hands-on exercise:** Convert a single-stage `FROM ubuntu:22.04` Dockerfile into a multi-stage distroless image and measure the size delta.

#### Step 4: Build, Tag, Push â€” The Daily Loop
1. **Build with BuildKit:**
   ```bash
   DOCKER_BUILDKIT=1 docker build -t myapp:1.2.3 .
   docker buildx create --use --name mybuilder
   docker buildx build --platform linux/amd64,linux/arm64 \
     -t ghcr.io/myorg/myapp:1.2.3 --push .
   ```
2. **Tagging strategy:**
   ```
   :1.2.3          semver
   :1.2            rolling minor
   :1              rolling major (risky)
   :latest         default â€” never what you deploy
   :main-<sha>     traceable to commit
   :stable         curated for prod
   ```
3. **SBOM and provenance â€” built in:**
   ```bash
   docker buildx build --sbom=true --provenance=true \
     -t myapp:1.2.3 --push .
   docker sbom myapp:1.2.3
   docker buildx imagetools inspect myapp:1.2.3 --format '{{json .SBOM}}'
   ```
4. **Hands-on exercise:** Build the same image 10 times with the same source and prove the image ID is identical.

#### Step 5: Volumes, Networking & The Data Layer
1. **The three volume types:**
   - **Named volume** â€” managed by Docker, easiest
   - **Bind mount** â€” a host path injected into the container
   - **tmpfs** â€” in-memory, fast, ephemeral
   ```bash
   docker volume create pgdata
   docker run -d -v pgdata:/var/lib/postgresql/data postgres:16
   docker run -d -v /home/me/app:/app:ro myapp
   docker run -d --tmpfs /tmp:rw,size=64m myapp
   ```
2. **Networking â€” the four drivers:**
   | Driver | Use |
   |--------|-----|
   | `bridge` (default) | Single-host, isolated |
   | `host` | No isolation, max performance |
   | `overlay` | Multi-host (Swarm) |
   | `macvlan` | Container appears on the physical network |
   | `none` | No networking (rarely used) |
   ```bash
   docker network create --driver bridge backend
   docker run -d --network backend --name db postgres:16
   docker run -d --network backend --name api myapp
   # 'db' resolves by container name
   ```
3. **DNS & service discovery:**
   - Containers on the same user-defined network resolve each other by name
   - Use `--network-alias` for multiple names
4. **Hands-on exercise:** Build a 2-tier app (API + postgres) on a user-defined network; prove DNS works and the DB is not exposed on host.

#### Step 6: Docker Compose â€” Local Stacks
1. **The minimal `compose.yml`:**
   ```yaml
   services:
     db:
       image: postgres:16-alpine
       environment:
         POSTGRES_PASSWORD: dev
       volumes:
         - pgdata:/var/lib/postgresql/data
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U postgres"]
         interval: 5s
         retries: 5
     api:
       build: .
       ports:
         - "8080:8080"
       environment:
         DATABASE_URL: postgres://postgres:dev@db:5432/app
       depends_on:
         db:
           condition: service_healthy
   volumes:
     pgdata:
   ```
2. **The Compose v2 features you need:**
   - `include` â€” split a stack across files
   - `extends` â€” DRY shared service config
   - `profiles` â€” opt-in services (e.g., `docker compose --profile debug up`)
   - `develop.watch` â€” hot reload in dev
   - `secrets:` â€” sourced from files or external
3. **Run it:**
   ```bash
   docker compose up -d
   docker compose logs -f api
   docker compose exec db psql -U postgres
   docker compose down -v   # include volumes
   ```
4. **Compose in CI â€” for integration tests:**
   ```yaml
   # .github/workflows/test.yml
   - uses: actions/checkout@v4
   - run: docker compose up -d --wait
   - run: npm test
   - run: docker compose down -v
   ```
5. **Hands-on exercise:** Take a 3-service app (web, api, db) and run it locally with Compose, then run the same `compose.yml` in a CI job.

#### Step 7: Production Patterns â€” Resources, Logging, Health
1. **Resource limits â€” always set them:**
   ```bash
   docker run -d \
     --memory=512m --memory-swap=512m \
     --cpus=1.0 \
     --pids-limit=200 \
     --restart=unless-stopped \
     --name api myapp:1.2.3
   ```
2. **Logging drivers â€” JSON to stdout, or to a collector:**
   ```bash
   docker run --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 ...
   docker run --log-driver=fluentd --log-opt fluentd-address=fluentd:24224 ...
   docker run --log-driver=awslogs --log-opt awslogs-group=myapp ...
   ```
3. **The 12-factor way: stdout/stderr only.**
4. **Healthchecks:**
   ```dockerfile
   HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
     CMD curl -f http://localhost:8080/healthz || exit 1
   ```
5. **Restart policies:**
   | Policy | Behavior |
   |--------|----------|
   | `no` | Default â€” do nothing |
   | `on-failure[:max]` | Restart on non-zero exit |
   | `always` | Restart regardless |
   | `unless-stopped` | Like always, but respect `docker stop` |
6. **Hands-on exercise:** Add a healthcheck, resource limits, and `--log-driver=json-file` to a container; verify with `docker inspect`.

#### Step 8: Image Security â€” The Production Bar
1. **Scanning with Trivy (or Grype, Snyk, Clair):**
   ```bash
   trivy image --severity HIGH,CRITICAL myapp:1.2.3
   trivy image --exit-code 1 --severity CRITICAL myapp:1.2.3   # CI gate
   ```
2. **The most important Dockerfile hardening rules:**
   - `USER nonroot` (or numeric `USER 65532`)
   - `COPY --chown=nonroot:nonroot` so permissions are right
   - `--read-only` filesystem at runtime
   - `--cap-drop=ALL` + add only what you need
   - `--security-opt no-new-privileges:true`
   - `--tmpfs /tmp:rw,size=64m,noexec,nosuid`
3. **Runtime hardening example:**
   ```bash
   docker run -d --read-only \
     --cap-drop=ALL --cap-add=NET_BIND_SERVICE \
     --security-opt no-new-privileges:true \
     --tmpfs /tmp:rw,size=64m,noexec,nosuid \
     myapp:1.2.3
   ```
4. **Sign with cosign, verify in admission:**
   ```bash
   cosign sign --key cosign.key ghcr.io/myorg/myapp:1.2.3
   cosign verify --key cosign.pub ghcr.io/myorg/myapp:1.2.3
   ```
5. **SBOM attestation:**
   ```bash
   syft myapp:1.2.3 -o spdx-json > sbom.spdx.json
   cosign attest --predicate sbom.spdx.json --type spdxjson \
     --key cosign.key ghcr.io/myorg/myapp:1.2.3
   ```
6. **Hands-on exercise:** Scan a vulnerable image, fix the Dockerfile, rebuild, rescan, and prove the CRITICAL count drops to zero.

#### Step 9: Multi-Arch & Build Optimization
1. **Multi-arch builds with buildx:**
   ```bash
   docker buildx create --use
   docker buildx build \
     --platform linux/amd64,linux/arm64,linux/arm/v7,linux/ppc64le \
     -t ghcr.io/myorg/myapp:1.2.3 --push .
   ```
2. **Inspect the manifest list:**
   ```bash
   docker buildx imagetools inspect ghcr.io/myorg/myapp:1.2.3
   ```
3. **Build cache â€” use it well:**
   ```bash
   docker buildx build --cache-from=type=registry,ref=ghcr.io/myorg/myapp:cache \
     --cache-to=type=registry,ref=ghcr.io/myorg/myapp:cache,mode=max .
   ```
4. **Layer caching â€” the order matters:**
   ```dockerfile
   # GOOD: deps change rarely
   COPY package.json package-lock.json ./
   RUN npm ci
   # source changes often
   COPY . .
   RUN npm run build
   ```
5. **Hands-on exercise:** Build the same Dockerfile twice â€” once naive, once with cache mounts + multi-stage â€” and compare build time and final size.

#### Step 10: Networking in Depth â€” CNM, Ports, and Service Mesh
1. **Port mapping modes:**
   - `-p 8080:80` â€” host:container, dynamic host port
   - `-p 127.0.0.1:8080:80` â€” loopback only
   - `--network host` â€” no isolation
2. **The iptables rules Docker creates:**
   - `DOCKER` chain â€” DNAT inbound published ports
   - `DOCKER-USER` â€” your custom rules (drop, log, rate limit)
3. **The IPv6 gotcha:** Docker's default bridge is IPv4-only. Enable IPv6 in `/etc/docker/daemon.json`:
   ```json
   { "ipv6": true, "fixed-cidr-v6": "fc00::/64", "experimental": true }
   ```
4. **Hands-on exercise:** Lock down the host so only a specific source IP can reach a published port, using `DOCKER-USER`.

#### Step 11: Debugging & Operations
1. **The toolkit:**
   ```bash
   docker ps -a
   docker logs --tail 200 -f <id>
   docker inspect <id> | jq
   docker exec -it <id> sh
   docker top <id>            # processes in the container's namespace
   docker stats               # live CPU/mem/net
   docker system df           # disk usage
   docker system prune -a     # nuke everything (use carefully)
   ```
2. **Enter a stopped container's filesystem:**
   ```bash
   docker commit <id> debug
   docker run -it --entrypoint sh debug
   ```
3. **Live network capture from inside a container's namespace:**
   ```bash
   PID=$(docker inspect -f '{{.State.Pid}}' <id>)
   sudo nsenter -t $PID -n tcpdump -i any -w /tmp/cap.pcap
   ```
4. **The "container won't start" checklist:**
   - `docker logs <id>`
   - `docker inspect <id> | jq '.State.Error'`
   - `docker run --entrypoint sh <image>` to bypass CMD
   - Mount the image's filesystem on the host to inspect
5. **Hands-on exercise:** Simulate a hung container, capture its state, find the offending syscall with `nsenter`, and fix it.

#### Step 12: Docker in Production â€” The Right Way
1. **Don't use `docker run` in prod** â€” use Compose, Swarm, Kubernetes, or Nomad
2. **The gold standard runtime configs:**
   - `--read-only` root filesystem
   - All caps dropped except the minimum
   - `no-new-privileges`
   - Memory + CPU limits set
   - Healthcheck defined
   - Non-root user
   - Image pinned by digest
   - Image signed + verified
3. **Logs to stdout, collected by a sidecar or node agent** â€” never let logs fill the container disk
4. **Image update policy:** automate with Dependabot/Renovate; rebuild on base image updates; rescan in CI
5. **Hands-on exercise:** Take a naive `docker run` and harden it to the gold standard, then write a Compose service that captures the same config.

#### Step 13: Beyond Docker â€” The Ecosystem
1. **The OCI-compatible alternatives:**
   - **Podman:** daemonless, rootless, drop-in for most use cases
   - **nerdctl:** Docker-compatible CLI for containerd
   - **Buildah:** build-only, no daemon
   - **containerd:** the runtime Kubernetes actually uses
2. **Migrating from Docker to containerd/nerdctl:**
   ```bash
   nerdctl run -d --name web -p 8080:80 nginx:1.27
   nerdctl compose up -d   # same compose.yml
   ```
3. **When to use which:**
   - **Laptop/CLI:** Docker Desktop (or Podman Desktop) for DX
   - **CI:** BuildKit in a worker, runtime can be anything
   - **Kubernetes:** containerd (the default), with cri-o as an option
   - **Edge/IoT:** K3s + containerd
4. **Hands-on exercise:** Install Podman, run the same Compose stack, and confirm behavior parity.

#### Step 14: Production-Grade Docker Habits
- **Pin every image by digest** in your manifests
- **Multi-stage everything** â€” final image should be < 100 MB if possible
- **Run as non-root**, with read-only FS and dropped caps
- **One process per container** â€” `init`/`tini` if multiple
- **Healthcheck on every long-lived service**
- **Logs to stdout, scraped by a node agent**
- **Scan every image in CI; block on CRITICAL**
- **Sign every image with cosign; verify in admission**
- **Automate base image updates with Renovate**
- **Document the run command and the production config** in the README

### Practice Project â€” Production-Grade Multi-Service App

> **Goal:** Build a 3-service app (web, api, worker + redis), Dockerize it, harden it, scan it, sign it, deploy it to a server with Compose, and run a load test.
> **Time estimate:** 12-15 hours
> **Deliverables:** A GitHub repo, signed images on GHCR, a runbook, and a load test report.

#### Phase 1 â€” Code (1h)
- [ ] **(B)** Pick a small app: web (nginx serving static), api (Node/Go/Python), worker (Node/Go/Python)
- [ ] **(B)** Add a `/healthz` and `/readyz` to the api â€” Step 7
- [ ] **(I)** Add structured JSON logs to stdout
- [ ] **(I)** Add Prometheus `/metrics` to the api

#### Phase 2 â€” Dockerfile (1.5h)
- [ ] **(B)** Write a multi-stage Dockerfile for the api â€” Step 3
- [ ] **(B)** Pin base images by digest, use distroless final
- [ ] **(I)** Use BuildKit cache mounts for the package manager
- [ ] **(I)** Add SBOM + provenance to the build

#### Phase 3 â€” Compose (1.5h)
- [ ] **(I)** Write `compose.yml` with web, api, worker, redis â€” Step 6
- [ ] **(I)** Use named volumes for redis data
- [ ] **(I)** Add healthchecks, depends_on with `condition: service_healthy`
- [ ] **(I)** Add resource limits (mem, cpu) to every service

#### Phase 4 â€” Hardening (1.5h)
- [ ] **(I)** Drop all caps, add only what's needed
- [ ] **(I)** Add `no-new-privileges`, read-only root FS, tmpfs for /tmp
- [ ] **(I)** Scan with Trivy; fail on HIGH/CRITICAL; fix and rebuild until clean
- [ ] **(I)** Sign the image with cosign; verify locally

#### Phase 5 â€” CI (1.5h)
- [ ] **(I)** GitHub Actions: lint â†’ test â†’ build â†’ scan â†’ sign â†’ push to GHCR
- [ ] **(I)** Multi-arch build (amd64 + arm64) with buildx
- [ ] **(I)** Use cache-from/cache-to to keep CI fast
- [ ] **(A)** Add a SLSA L3 provenance attestation

#### Phase 6 â€” Runtime (2h)
- [ ] **(I)** Run on a VM with `docker compose up -d`
- [ ] **(I)** Verify the api is reachable, /healthz returns 200, /metrics scrape works
- [ ] **(I)** Container survives `kill -9` and restarts cleanly
- [ ] **(I)** Logs flow to a file, rotated by logrotate

#### Phase 7 â€” Load & DR (1.5h)
- [ ] **(I)** Run `wrk -t4 -c100 -d60s http://api:8080/` and capture metrics
- [ ] **(I)** Kill the api, verify it restarts in < 10s, and that requests resume
- [ ] **(I)** Tear it all down with `docker compose down -v` and bring it back from a fresh clone
- [ ] **(A)** Practice the upgrade: build a new tag, roll the stack forward with `compose up -d --no-deps api`, roll back

#### Phase 8 â€” Documentation (1h)
- [ ] **(B)** Write a README: build, run, test, deploy, troubleshoot
- [ ] **(I)** Write `RUNBOOK.md`: build, scan, sign, push, deploy, rollback
- [ ] **(A)** Add a `SECURITY.md` describing the image hardening, signing, and verification process
- [ ] **(A)** Benchmark image size + build time, document in the README

#### Bonus Challenges
- [ ] **(A)** Migrate the same stack to Kubernetes (later in the path) and prove parity
- [ ] **(A)** Implement a `Dockerfile` that builds for 4 architectures with emulation
- [ ] **(A)** Write a custom admission controller that verifies cosign signatures before pod start

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 24 tasks across 8 phases + 3 bonus challenges**

### Key Takeaways
- Containers are the deployment artifact of every modern platform â€” master the image, master the deploy
- Multi-stage, distroless, non-root, read-only â€” the production defaults
- Compose is the fastest way to run a multi-service stack locally and in CI
- Image security (scanning + signing + minimal base) is non-negotiable in any regulated industry
- Docker is one of several OCI runtimes; the skills transfer to Podman, containerd, and nerdctl

---

## â˜¸ï¸ Kubernetes

**Difficulty:** Advanced | **Time:** 50-60 hours

### Overview
Kubernetes is the operating system of the cloud. It schedules containers across a cluster, heals them when they fail, scales them with load, and provides a single declarative API for everything from HTTP services to databases to machine-learning jobs. Born at Google, now the de facto standard everywhere â€” AWS EKS, Azure AKS, GCP GKE, Red Hat OpenShift, Rancher, VMWare Tanzu, and on-prem with kubeadm or K3s. This tool takes you from a one-pod cluster to a production multi-tenant platform with the operator pattern, GitOps, autoscaling, network policy, observability, and disaster recovery.

### Why Learn This
- 90%+ of new cloud-native projects run on Kubernetes
- The same skills work on every cloud and on-prem â€” true portability
- It's the platform where every other DevOps tool (Terraform, Helm, ArgoCD, Prometheus, Istio) composes
- Mastering the API + control loop is mastering distributed systems
- The CKA / CKAD / CKS certifications are the industry standard â€” and earnable

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The cluster architecture â€” what's actually running:**
   ```
   Control Plane
   â”œâ”€â”€ kube-apiserver       â† every component talks to this
   â”œâ”€â”€ etcd                 â† the only stateful component
   â”œâ”€â”€ kube-scheduler       â† picks a node for each Pod
   â”œâ”€â”€ kube-controller-manager
   â””â”€â”€ cloud-controller-manager  (cloud-specific)

   Worker Nodes
   â”œâ”€â”€ kubelet              â† the node agent
   â”œâ”€â”€ kube-proxy           â† the network rules
   â”œâ”€â”€ container runtime    â† containerd / CRI-O
   â””â”€â”€ your Pods
   ```
2. **The control loop â€” the heartbeat of Kubernetes:**
   ```
   desired state (YAML) â”€â”€â–¶ apiserver â”€â”€â–¶ etcd
        â–²                                  â”‚
        â”‚                                  â–¼
        â””â”€â”€â”€â”€ controller reconciles â—€â”€â”€ observes (watch)
   ```
3. **The Pod â€” the atomic unit:**
   - 1+ containers, shared network namespace, shared volumes
   - Has an IP, ephemeral, reschedulable
   - **You don't deploy containers in K8s, you deploy Pods**
4. **Set up a local cluster in 60 seconds:**
   ```bash
   # Kind (Kubernetes IN Docker) â€” best for CI
   curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
   chmod +x ./kind && sudo mv ./kind /usr/local/bin/
   kind create cluster --name lab

   # Minikube â€” best for laptop
   minikube start --cpus 4 --memory 8g

   # k3d â€” fastest, lightweight
   k3d cluster create lab --agents 3
   ```
5. **The 10 commands that prove it works:**
   ```bash
   kubectl get nodes
   kubectl get pods -A
   kubectl get events -A --sort-by=.lastTimestamp
   kubectl run nginx --image=nginx:1.27 --port=80
   kubectl get pod nginx -o wide
   kubectl describe pod nginx
   kubectl logs nginx
   kubectl exec -it nginx -- sh
   kubectl delete pod nginx
   kubectl cluster-info
   ```
6. **Hands-on exercise:** Stand up a Kind cluster, run an nginx Pod, exec in, hit a public URL from inside, and tear it down.

#### Step 2: The Workload API â€” Pods, Deployments, StatefulSets
1. **The full type taxonomy â€” what to use when:**
   | Type | Use |
   |------|-----|
   | Pod | Bare unit, never deployed alone in prod |
   | ReplicaSet | Owns N replicas of a Pod (you usually don't write these) |
   | Deployment | ReplicaSet + rolling update strategy â€” stateless apps |
   | StatefulSet | Stable identity, persistent storage â€” databases, queues |
   | DaemonSet | One per node â€” log shippers, CNI, monitoring |
   | Job | Run-to-completion â€” batch, migrations |
   | CronJob | Scheduled Job â€” backups, reports |
   | Custom Resource (CR) | Extend the API with your own type |
2. **Your first Deployment â€” the file you'll write 1000 times:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: web
     labels: { app: web }
   spec:
     replicas: 3
     selector:
       matchLabels: { app: web }
     strategy:
       type: RollingUpdate
       rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }
     template:
       metadata:
         labels: { app: web }
       spec:
         containers:
           - name: web
             image: ghcr.io/myorg/web:1.2.3
             imagePullPolicy: IfNotPresent
             ports: [{ containerPort: 8080 }]
             resources:
               requests: { cpu: 100m, memory: 128Mi }
               limits:   { cpu: 500m, memory: 256Mi }
             readinessProbe:
               httpGet: { path: /readyz, port: 8080 }
               initialDelaySeconds: 2
               periodSeconds: 5
             livenessProbe:
               httpGet: { path: /healthz, port: 8080 }
               initialDelaySeconds: 10
               periodSeconds: 10
             securityContext:
               runAsNonRoot: true
               runAsUser: 65532
               readOnlyRootFilesystem: true
               allowPrivilegeEscalation: false
               capabilities: { drop: ["ALL"] }
   ```
3. **Apply and observe:**
   ```bash
   kubectl apply -f deployment.yaml
   kubectl rollout status deployment/web
   kubectl get pods -l app=web
   kubectl describe deployment web
   ```
4. **Hands-on exercise:** Deploy the example, scale to 5 replicas, roll a new image, and watch the rollout with `kubectl rollout status`.

#### Step 3: Services, Ingress & The Networking Model
1. **The three Service types:**
   | Type | Use |
   |------|-----|
   | ClusterIP | Internal only â€” default |
   | NodePort | Each node exposes a port 30000-32767 â€” dev only |
   | LoadBalancer | Cloud provisions an LB â€” production ingress |
   | ExternalName | CNAME to an external service |
   | Headless (ClusterIP None) | StatefulSet â€” DNS per Pod |
2. **The Service manifest:**
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web
   spec:
     type: ClusterIP
     selector: { app: web }
     ports:
       - port: 80
         targetPort: 8080
   ```
3. **Ingress â€” the L7 router:**
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: web
     annotations:
       cert-manager.io/cluster-issuer: letsencrypt-prod
   spec:
     ingressClassName: nginx
     tls:
       - hosts: [app.example.com]
         secretName: app-tls
     rules:
       - host: app.example.com
         http:
           paths:
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: web
                   port:
                     number: 80
   ```
4. **Ingress controllers â€” the implementations:**
   - **NGINX Ingress:** the default, mature, ubiquitous
   - **Traefik:** dynamic config, great dashboard
   - **HAProxy:** performant, complex
   - **Envoy Gateway / Istio / Cilium:** the service-mesh-aware options
5. **Gateway API â€” the future (and what to use for new projects):**
   ```yaml
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata: { name: web }
   spec:
     parentRefs:
       - name: public
         namespace: gateway
     hostnames: [app.example.com]
     rules:
       - matches: [{ path: { type: PathPrefix, value: / } }]
         backendRefs:
           - name: web
             port: 80
   ```
6. **Hands-on exercise:** Deploy NGINX Ingress Controller, expose a 3-service app with TLS via cert-manager, hit it from your laptop.

#### Step 4: ConfigMap, Secret, Projected Volumes
1. **ConfigMap â€” non-sensitive config:**
   ```bash
   kubectl create configmap app-config --from-file=app.yaml=./app.yaml --from-literal=LOG_LEVEL=info
   ```
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata: { name: app-config }
   data:
     LOG_LEVEL: info
     app.yaml: |
       feature: enabled
   ```
2. **Secret â€” sensitive config (base64, NOT encrypted at rest by default):**
   ```bash
   kubectl create secret generic db --from-literal=password=secret
   ```
3. **Mount as env or volume:**
   ```yaml
   env:
     - name: DB_PASSWORD
       valueFrom:
         secretKeyRef: { name: db, key: password }
   volumeMounts:
     - name: cfg
       mountPath: /etc/app
   volumes:
     - name: cfg
       configMap: { name: app-config }
   ```
4. **The right way to handle secrets in 2026:**
   - **External Secrets Operator** â†’ Vault, AWS SM, GSM, Azure KV
   - **Sealed Secrets** for GitOps with no external dependencies
   - **CSI Secret Driver** for direct mount
5. **Hands-on exercise:** Mount a ConfigMap + a Secret, change them, and watch the Pod pick up the change (with a reload, or by triggering a rollout).

#### Step 5: Storage â€” PV, PVC, StorageClass
1. **The three layers:**
   - **PersistentVolume (PV):** the actual storage (NFS, EBS, Ceph, local)
   - **PersistentVolumeClaim (PVC):** a request for storage
   - **StorageClass:** dynamic provisioning template
2. **A StorageClass for AWS EBS:**
   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata: { name: gp3 }
   provisioner: ebs.csi.aws.com
   parameters:
     type: gp3
     fsType: ext4
   volumeBindingMode: WaitForFirstConsumer
   reclaimPolicy: Delete
   ```
3. **A claim:**
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata: { name: data }
   spec:
     storageClassName: gp3
     accessModes: [ReadWriteOnce]
     resources: { requests: { storage: 10Gi } }
   ```
4. **StatefulSet â€” stable identity + per-pod storage:**
   ```yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata: { name: db }
   spec:
     serviceName: db
     replicas: 3
     selector: { matchLabels: { app: db } }
     template:
       metadata: { labels: { app: db } }
       spec:
         containers:
           - name: db
             image: postgres:16
             volumeMounts: [{ name: data, mountPath: /var/lib/postgresql/data }]
     volumeClaimTemplates:
       - metadata: { name: data }
         spec:
           accessModes: [ReadWriteOnce]
           storageClassName: gp3
           resources: { requests: { storage: 100Gi } }
   ```
5. **The 6 things you must know about storage:**
   - Access mode: `RWO` (one node), `ROX` (many read), `RWX` (many read+write)
   - Reclaim policy: `Retain` (prod), `Delete` (dev)
   - Volume binding: `Immediate` vs `WaitForFirstConsumer` (use the latter for topology)
   - Snapshots: `VolumeSnapshot` + driver
   - Expansion: `allowVolumeExpansion: true` on the StorageClass
   - Topology: zone-aware for cloud, node-aware for on-prem
6. **Hands-on exercise:** Deploy a 3-replica Postgres StatefulSet on a cloud cluster, kill a node, and prove the data survives.

#### Step 6: Scheduling â€” Taints, Tolerations, Affinity, Topology
1. **The 4 scheduling primitives:**
   - **nodeSelector:** simple label match
   - **Affinity / Anti-Affinity:** required or preferred rules
   - **Taints + Tolerations:** repel pods from nodes
   - **Topology Spread:** even distribution across zones/nodes
2. **Spread across zones:**
   ```yaml
   topologySpreadConstraints:
     - maxSkew: 1
       topologyKey: topology.kubernetes.io/zone
       whenUnsatisfiable: DoNotSchedule
       labelSelector:
         matchLabels: { app: web }
   ```
3. **Anti-affinity â€” don't put two of the same Pod on one node:**
   ```yaml
   affinity:
     podAntiAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
         - labelSelector: { matchLabels: { app: web } }
           topologyKey: kubernetes.io/hostname
   ```
4. **Dedicated nodes via taint + toleration:**
   ```bash
   kubectl taint nodes node1 app=db:NoSchedule
   ```
   ```yaml
   tolerations:
     - key: app
       operator: Equal
       value: db
       effect: NoSchedule
   ```
5. **Hands-on exercise:** Spread a 6-replica Deployment across 3 nodes and 2 zones; prove it with `kubectl get pods -o wide`.

#### Step 7: Resource Management, QoS & Limits
1. **The three QoS classes:**
   - **Guaranteed:** requests == limits for every container
   - **Burstable:** requests set, limits higher
   - **BestEffort:** no requests or limits
2. **Requests vs limits â€” the mental model:**
   - **requests:** "guaranteed to be available" â€” used for scheduling
   - **limits:** "kubelet will kill the container if it exceeds" â€” used for runtime
3. **LimitRange + ResourceQuota â€” for multi-tenant clusters:**
   ```yaml
   apiVersion: v1
   kind: LimitRange
   metadata: { name: default, namespace: team-a }
   spec:
     limits:
       - type: Container
         default: { cpu: 500m, memory: 512Mi }
         defaultRequest: { cpu: 100m, memory: 128Mi }
         max: { cpu: 2, memory: 4Gi }
   ```
4. **Vertical Pod Autoscaler â€” auto-tune the requests:**
   ```bash
   vpa recommender --recommendation=memory,cpu
   kubectl get vpa
   ```
5. **Hands-on exercise:** Apply a LimitRange to a namespace, try to deploy a Pod that exceeds it, and confirm it's rejected.

#### Step 8: Probes, Lifecycle & The Real Production Patterns
1. **The three probes:**
   | Probe | Purpose | Failure action |
   |-------|---------|----------------|
   | `startupProbe` | Slow-starting apps | Don't run liveness/readiness until success |
   | `livenessProbe` | "Am I still alive?" | Restart the container |
   | `readinessProbe` | "Am I ready to serve?" | Remove from Service endpoints |
2. **Liveness vs readiness â€” the most common bug:**
   - Liveness failure â†’ restart loop, no recovery
   - Use **startupProbe** for slow apps; use **readinessProbe** for "needs warmup"
3. **Lifecycle hooks:**
   ```yaml
   lifecycle:
     preStop:
       exec:
         command: ["/bin/sh", "-c", "sleep 10"]
     postStart:
       exec:
         command: ["/bin/sh", "-c", "touch /tmp/started"]
   ```
4. **Graceful shutdown â€” the recipe:**
   - Set `terminationGracePeriodSeconds: 30`
   - `preStop` hook drains in-flight requests
   - Pod stays in Service endpoints until the grace period ends
5. **Hands-on exercise:** Add probes + a `preStop` hook to your Deployment, kill the Pod, and watch `kubectl describe pod` show the termination flow.

#### Step 9: RBAC & Security Context
1. **ServiceAccount â€” the identity of a Pod:**
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata: { name: app, namespace: prod }
   ```
2. **Role + RoleBinding â€” namespaced permissions:**
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata: { namespace: prod, name: app-reader }
   rules:
     - apiGroups: [""]
       resources: ["pods", "services", "configmaps"]
       verbs: ["get", "list", "watch"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata: { namespace: prod, name: app-reader }
   subjects:
     - kind: ServiceAccount
       name: app
       namespace: prod
   roleRef:
     kind: Role
     name: app-reader
     apiGroup: rbac.authorization.k8s.io
   ```
3. **ClusterRole + ClusterRoleBinding â€” cluster-wide.**
4. **Security context â€” the 6 fields that matter:**
   ```yaml
   securityContext:
     runAsNonRoot: true
     runAsUser: 65532
     runAsGroup: 65532
     fsGroup: 65532
     seccompProfile: { type: RuntimeDefault }
   ```
5. **Pod Security Standards â€” the three levels:**
   - **Privileged:** unrestricted
   - **Baseline:** blocks the most egregious
   - **Restricted:** hardened best practice (the goal)
6. **Hands-on exercise:** Apply the `restricted` PSS to a namespace; deploy a Pod that violates it; fix the manifest.

#### Step 10: Networking Deep Dive â€” CNI, DNS, NetworkPolicy
1. **Every cluster has a CNI â€” the plugin that wires Pods:**
   | CNI | Use |
   |-----|-----|
   | Calico | L3 + policy, BGP, the classic |
   | Cilium | eBPF, replaces kube-proxy, Hubble observability |
   | Flannel | L3 only, simplest |
   | Weave | L3 + policy, simple mesh |
   | AWS VPC CNI | Pods get real VPC IPs (no NAT) |
   | Azure CNI Overlay | Pods in overlay, scale |
2. **The Pod-to-Pod contract:**
   - Every Pod gets an IP from the cluster CIDR
   - Pod-to-Pod is direct, no NAT
   - Services are virtual IPs implemented by kube-proxy (iptables/IPVS) or Cilium (eBPF)
3. **CoreDNS â€” the cluster DNS:**
   - `db.prod.svc.cluster.local` â†’ the Service IP
   - `10-0-0-5.db.prod.svc.cluster.local` â†’ a Pod IP (headless)
4. **NetworkPolicy â€” the L4 firewall:**
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata: { name: web-allow, namespace: prod }
   spec:
     podSelector: { matchLabels: { app: web } }
     policyTypes: [Ingress, Egress]
     ingress:
       - from:
           - podSelector: { matchLabels: { app: lb } }
         ports: [{ protocol: TCP, port: 8080 }]
     egress:
       - to:
           - podSelector: { matchLabels: { app: db } }
         ports: [{ protocol: TCP, port: 5432 }]
   ```
5. **Hands-on exercise:** With Calico or Cilium, write a default-deny NetworkPolicy for a namespace, then allow only the paths you need.

#### Step 11: Auto-scaling â€” HPA, VPA, KEDA, Cluster Autoscaler
1. **The four scalers and when to use each:**
   | Scaler | Scales | Signal |
   |--------|--------|--------|
   | HPA | Pods | CPU, memory, custom metrics |
   | VPA | Pod requests/limits | Past usage |
   | KEDA | Pods (event-driven) | Kafka lag, queue depth, cron |
   | Cluster Autoscaler | Nodes | Pending Pods |
   | Karpenter | Nodes (faster, smarter) | Pending Pods, pod shape |
2. **HPA on CPU:**
   ```bash
   kubectl autoscale deployment web --cpu-percent=60 --min=3 --max=20
   ```
   ```yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata: { name: web }
   spec:
     scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: web }
     minReplicas: 3
     maxReplicas: 20
     metrics:
       - type: Resource
         resource:
           name: cpu
           target: { type: Utilization, averageUtilization: 60 }
   ```
3. **KEDA â€” event-driven:**
   ```yaml
   apiVersion: keda.sh/v1alpha1
   kind: ScaledObject
   metadata: { name: worker }
   spec:
     scaleTargetRef: { name: worker }
     triggers:
       - type: kafka
         metadata:
           bootstrapServers: kafka:9092
           consumerGroup: my-group
           lagThreshold: "100"
   ```
4. **Karpenter â€” the modern node autoscaler:**
   ```yaml
   apiVersion: karpenter.sh/v1beta1
   kind: NodePool
   metadata: { name: default }
   spec:
     template:
       spec:
         requirements:
           - key: karpenter.sh/capacity-type
             operator: In
             values: ["spot", "on-demand"]
           - key: kubernetes.io/arch
             operator: In
             values: ["amd64", "arm64"]
         nodeClassRef: { name: default }
     limits: { cpu: "1000", memory: 4000Gi }
   ```
5. **Hands-on exercise:** Deploy an HPA, run a load test with `wrk`, watch the replica count grow, then drop the load and watch it shrink.

#### Step 12: Observability â€” Metrics, Logs, Traces
1. **The three signals in K8s:**
   - **Metrics:** Prometheus / Datadog / CloudWatch
   - **Logs:** stdout â†’ Fluent Bit / Vector / Promtail
   - **Traces:** OpenTelemetry Collector â†’ Jaeger / Tempo / Datadog
2. **kube-prometheus-stack â€” the canonical install:**
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm install kube-prom prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
   ```
3. **The 10 dashboards you'll have day one:**
   - Cluster overview, node resources, pod resources
   - API server latency, etcd, scheduler
   - Per-namespace, per-workload
   - Network throughput, errors, DNS
4. **The pattern â€” every Pod exports `/metrics`:**
   ```yaml
   annotations:
     prometheus.io/scrape: "true"
     prometheus.io/port: "8080"
     prometheus.io/path: "/metrics"
   ```
5. **Loki + Promtail (or Vector) for logs:**
   ```bash
   helm install loki grafana/loki-stack -n logging --create-namespace
   # LogQL is grep+more: {namespace="prod"} |= "error" | json
   ```
6. **The Golden Signals â€” for every service:**
   - **Latency:** histogram `http_request_duration_seconds`
   - **Traffic:** counter `http_requests_total`
   - **Errors:** counter `http_requests_total{status=~"5.."}`
   - **Saturation:** gauge of pool depth, queue length
7. **Hands-on exercise:** Install kube-prometheus-stack + Loki, view the cluster dashboard, query a metric, and grep a log line by pod name.

#### Step 13: Helm â€” The Package Manager
1. **The structure of a chart:**
   ```
   mychart/
   â”œâ”€â”€ Chart.yaml
   â”œâ”€â”€ values.yaml
   â”œâ”€â”€ values.schema.json
   â”œâ”€â”€ templates/
   â”‚   â”œâ”€â”€ deployment.yaml
   â”‚   â”œâ”€â”€ service.yaml
   â”‚   â”œâ”€â”€ ingress.yaml
   â”‚   â”œâ”€â”€ _helpers.tpl
   â”‚   â””â”€â”€ NOTES.txt
   â””â”€â”€ charts/         (dependencies)
   ```
2. **The minimum `Chart.yaml`:**
   ```yaml
   apiVersion: v2
   name: mychart
   version: 1.2.3
   appVersion: "1.2.3"
   description: My service
   type: application
   ```
3. **A template with helpers:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: {{ include "mychart.fullname" . }}
     labels: {{- include "mychart.labels" . | nindent 4 }}
   spec:
     replicas: {{ .Values.replicaCount }}
     selector:
       matchLabels: {{- include "mychart.selectorLabels" . | nindent 6 }}
     template:
       spec:
         containers:
           - name: {{ .Chart.Name }}
             image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
   ```
4. **The commands you use daily:**
   ```bash
   helm create mychart
   helm template mychart ./mychart
   helm install myapp ./mychart -n prod --create-namespace -f values.prod.yaml
   helm upgrade myapp ./mychart -n prod -f values.prod.yaml
   helm history myapp -n prod
   helm rollback myapp 3 -n prod
   helm uninstall myapp -n prod
   helm list -A
   helm search repo nginx
   ```
5. **Hands-on exercise:** Package your multi-service app as a Helm chart with values per environment, install to dev, and roll back.

#### Step 14: Kustomize â€” The Overlay Alternative
1. **When to pick Kustomize over Helm:**
   - Pure YAML, no templating needed
   - GitOps-friendly (no `helm template` step)
   - Less powerful, more predictable
2. **The structure:**
   ```
   base/
   â”œâ”€â”€ kustomization.yaml
   â”œâ”€â”€ deployment.yaml
   â”œâ”€â”€ service.yaml
   â””â”€â”€ ingress.yaml
   overlays/
   â”œâ”€â”€ dev/kustomization.yaml
   â”œâ”€â”€ staging/kustomization.yaml
   â””â”€â”€ prod/kustomization.yaml
   ```
3. **The base kustomization:**
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   resources:
     - deployment.yaml
     - service.yaml
     - ingress.yaml
   commonLabels:
     app: web
   ```
4. **The prod overlay:**
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   namespace: prod
   replicas:
     - name: web
     count: 10
   images:
     - name: web
       newName: ghcr.io/myorg/web
       newTag: 1.2.3
   patches:
     - target: { kind: Deployment, name: web }
       patch: |-
         - op: replace
           path: /spec/template/spec/containers/0/resources/limits/cpu
           value: "2"
   ```
5. **Apply:**
   ```bash
   kubectl apply -k overlays/prod
   ```
6. **Hands-on exercise:** Convert your Helm chart to Kustomize and prove the dev/staging/prod overlays produce the right resources.

#### Step 15: Operators & Custom Resources
1. **The operator pattern:** encode human ops knowledge into software that runs in the cluster.
2. **The pieces:**
   - **CRD:** the schema of your custom resource
   - **Controller:** the loop that reconciles CRs to reality
3. **The CRD:**
   ```yaml
   apiVersion: apiextensions.k8s.io/v1
   kind: CustomResourceDefinition
   metadata: { name: websites.example.com }
   spec:
     group: example.com
     scope: Namespaced
     names:
       plural: websites
       singular: website
       kind: Website
     versions:
       - name: v1
         served: true
         storage: true
         schema:
           openAPIV3Schema:
             type: object
             properties:
               spec:
                 type: object
                 properties:
                   image: { type: string }
                   replicas: { type: integer, minimum: 1, maximum: 10 }
   ```
4. **The controller (sketch, real impl uses controller-runtime):**
   ```python
   import kopf
   @kopf.on.create("example.com", "v1", "websites")
   def create(spec, **_):
       ensure_deployment(spec["image"], spec.get("replicas", 1))
       ensure_service()
   ```
5. **The OperatorHub ecosystem:** Postgres Operator, Redis Operator, Cert-Manager, ArgoCD, Crossplane â€” all operators.
6. **Hands-on exercise:** Define a `Website` CRD and write a Python (Kopf) operator that creates a Deployment + Service on `create`.

#### Step 16: Production Operations
1. **The maintenance commands you'll use:**
   ```bash
   kubectl cordon <node>      # drain (no new pods)
   kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
   kubectl uncordon <node>
   kubectl get nodes -o wide
   kubectl top nodes
   kubectl top pods -A --sort-by=memory
   kubectl describe node <node>
   ```
2. **Backup â€” Velero for cluster state:**
   ```bash
   velero backup create daily --include-namespaces prod
   velero schedule create daily --schedule="0 2 * * *"
   velero restore create --from-backup daily
   ```
3. **Disaster recovery drills â€” the runbook:**
   - Kill a node â†’ reschedule
   - Delete a namespace â†’ restore from backup
   - Lose etcd â†’ restore from snapshot
   - Lose an AZ â†’ failover
4. **Cost â€” Kubecost for visibility, or OpenCost:**
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubecost/cost-analyzer-helm-chart/master/kubecost.yaml
   ```
5. **Upgrades â€” control plane then nodes:**
   ```bash
   # EKS
   aws eks update-cluster-version --name prod --version 1.31
   # Self-managed
   kubeadm upgrade plan
   kubeadm upgrade apply v1.31.0
   ```
6. **Hands-on exercise:** Cordon a node, drain it, simulate a node failure, and watch pods reschedule. Then restore the node.

#### Step 17: Production-Grade Kubernetes Habits
- **Pin every image by digest** in your manifests
- **Run as non-root**, with read-only FS and dropped caps
- **Set requests AND limits** for every container
- **Use startup, liveness, and readiness probes** â€” never just one
- **Topology spread across zones** for HA
- **PodDisruptionBudget** for every Deployment
- **NetworkPolicy default-deny**, allow only what's needed
- **ResourceQuota per namespace** in multi-tenant clusters
- **GitOps** for everything (ArgoCD / Flux, later in the path)
- **Backup the cluster state** with Velero on a schedule
- **Drill the failure modes** quarterly

### Practice Project â€” Production Multi-Tenant Platform

> **Goal:** Build a 3-tier (web/api/db) app on a real cluster (EKS/AKS/GKE or Kind), with HPA, NetworkPolicy, GitOps, observability, TLS, and a DR drill.
> **Time estimate:** 25-30 hours
> **Deliverables:** A GitOps repo, a working cluster, a load test report, and a full runbook.

#### Phase 1 â€” Cluster & Bootstrap (2h)
- [ ] **(I)** Stand up a cluster (EKS, AKS, GKE, or Kind on 3 nodes)
- [ ] **(I)** Install the CNI (Calico or Cilium)
- [ ] **(I)** Install NGINX Ingress Controller + cert-manager
- [ ] **(I)** Install kube-prometheus-stack + Loki

#### Phase 2 â€” Workloads (3h)
- [ ] **(I)** Deploy a 3-service app (web/api/db) with Helm â€” Step 13
- [ ] **(I)** Use HPA, topology spread, anti-affinity
- [ ] **(I)** Add liveness + readiness + startup probes
- [ ] **(I)** Use Secrets via External Secrets Operator â†’ Vault (or AWS SM) â€” Step 4

#### Phase 3 â€” Networking & Security (3h)
- [ ] **(I)** Configure Ingress + cert-manager + Let's Encrypt for a public hostname
- [ ] **(I)** Apply Pod Security Standards `restricted` to the namespace
- [ ] **(I)** Add NetworkPolicy default-deny + explicit allow
- [ ] **(I)** Drop caps, read-only FS, no-new-privileges on every Pod

#### Phase 4 â€” Storage & Data (2h)
- [ ] **(I)** Deploy Postgres as a StatefulSet with a 10Gi PVC
- [ ] **(I)** Snapshot the volume, restore to a new PVC, prove the data
- [ ] **(A)** Set up a 3-replica Postgres with a leader/follower controller (e.g., Zalando, CloudNativePG)

#### Phase 5 â€” GitOps (2h)
- [ ] **(I)** Install ArgoCD (later in the path), point it at your Git repo
- [ ] **(I)** Move the app to a Kustomize structure with overlays/dev,staging,prod
- [ ] **(I)** Make a config change in Git; ArgoCD reconciles automatically
- [ ] **(A)** Use Image Updater to bump versions automatically

#### Phase 6 â€” Observability (2h)
- [ ] **(I)** Confirm Prometheus scrapes every Pod's /metrics
- [ ] **(I)** Build a Grafana dashboard for the 4 Golden Signals
- [ ] **(I)** Configure Loki to ingest cluster logs; query by namespace, pod, label
- [ ] **(A)** Add a SLO dashboard with error budget burn rate alerts

#### Phase 7 â€” Autoscaling & Load (2h)
- [ ] **(I)** Install KEDA + Cluster Autoscaler (or Karpenter)
- [ ] **(I)** Run `wrk` against the api; watch HPA + nodes scale
- [ ] **(I)** Drop the load; watch scale-down happen cleanly
- [ ] **(A)** Add KEDA on a Kafka consumer (later in the path) for event-driven scaling

#### Phase 8 â€” DR & Operations (2h)
- [ ] **(I)** Install Velero; schedule a daily backup of your namespace
- [ ] **(I)** Restore the namespace from backup; prove the app is identical
- [ ] **(I)** Cordon + drain a node; prove rescheduling works
- [ ] **(I)** Practice upgrading the cluster by one minor version

#### Phase 9 â€” Documentation (1h)
- [ ] **(I)** Write `RUNBOOK.md`: build, deploy, scale, debug, restore, upgrade
- [ ] **(I)** Write `SECURITY.md` describing hardening, PSS, NetworkPolicy, image verification
- [ ] **(A)** Generate an SBOM for your workloads; document the supply chain

#### Bonus Challenges
- [ ] **(A)** Add a service mesh (Istio or Linkerd) and use mTLS between all services
- [ ] **(A)** Write a Python operator (Kopf) for a custom CRD (e.g., `ScheduledJob`)
- [ ] **(A)** Implement cluster federation (Karmada or Kubefed) across 2 clusters

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 24 tasks across 9 phases + 3 bonus challenges**

### Key Takeaways
- Kubernetes is a declarative API + a control loop â€” once you see that, everything else is a special case
- The workload types, the Service API, and the storage model are the core
- Networking (CNI, Service, Ingress, NetworkPolicy) is where most production bugs live
- Observability is non-optional: metrics, logs, traces for every service
- GitOps + Helm/Kustomize + operators is the modern app delivery model
- Production means: non-root, requests+limits, probes, topology spread, NetworkPolicy, PSS, DR drills

---

## âŽˆ Helm

**Difficulty:** Intermediate | **Time:** 12-15 hours

### Overview
Helm is the package manager for Kubernetes â€” the equivalent of `apt` for clusters. It templates manifests, manages releases, supports rollbacks, and provides a chart distribution ecosystem (Artifact Hub, OCI registries). Mastering Helm is the difference between copy-pasting YAML and shipping a maintainable, versioned, multi-environment application.

### Why Learn This
- 80%+ of the K8s ecosystem is distributed as Helm charts
- It is the lingua franca for templating, packaging, and releasing K8s apps
- Every cloud provider offers first-party charts (ingress-nginx, cert-manager, ArgoCD, prometheus)
- Knowing Helm internals lets you debug, fork, and contribute to charts
- Helm 3 removed Tiller, making the model much simpler and safer

### Learn by Doing â€” Step by Step

#### Step 1: Install, Tour, First Chart
1. **Install:**
   ```bash
   curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   chmod 700 get_helm.sh && ./get_helm.sh
   helm version
   ```
2. **Add a repo, install a chart:**
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   helm install my-redis bitnami/redis --set auth.password=secret
   helm list
   helm status my-redis
   helm uninstall my-redis
   ```
3. **Create your own chart:**
   ```bash
   helm create myapp
   tree myapp
   ```
   ```
   myapp/
   â”œâ”€â”€ Chart.yaml
   â”œâ”€â”€ values.yaml
   â”œâ”€â”€ templates/
   â”‚   â”œâ”€â”€ deployment.yaml
   â”‚   â”œâ”€â”€ service.yaml
   â”‚   â”œâ”€â”€ ingress.yaml
   â”‚   â”œâ”€â”€ _helpers.tpl
   â”‚   â”œâ”€â”€ hpa.yaml
   â”‚   â”œâ”€â”€ serviceaccount.yaml
   â”‚   â”œâ”€â”€ NOTES.txt
   â”‚   â””â”€â”€ tests/test-connection.yaml
   â””â”€â”€ .helmignore
   ```
4. **Inspect what a chart will do:**
   ```bash
   helm template myapp ./myapp
   helm template myapp ./myapp -f my-values.yaml --set replicaCount=3
   helm install myapp ./myapp --dry-run --debug
   ```
5. **Hands-on exercise:** Create `myapp`, install it, and uninstall it. Render with `helm template` and compare to a manual `kubectl apply`.

#### Step 2: values.yaml â€” The Heart of the Chart
1. **A well-organized values.yaml:**
   ```yaml
   # -- Number of replicas
   replicaCount: 3

   image:
     # -- Image repository
     repository: ghcr.io/myorg/myapp
     # -- Image tag (overrides Chart.appVersion)
     tag: ""
     # -- Image pull policy
     pullPolicy: IfNotPresent

   imagePullSecrets: []

   nameOverride: ""
   fullnameOverride: ""

   serviceAccount:
     create: true
     annotations: {}
    name: ""

   podAnnotations: {}
   podSecurityContext:
     runAsNonRoot: true
     runAsUser: 65532
     fsGroup: 65532

   service:
     type: ClusterIP
     port: 80
     targetPort: 8080

   ingress:
     enabled: true
     className: nginx
     annotations:
       cert-manager.io/cluster-issuer: letsencrypt-prod
     hosts:
       - host: app.example.com
         paths:
           - path: /
             pathType: Prefix
     tls:
       - hosts: [app.example.com]
         secretName: app-tls

   resources:
     requests: { cpu: 100m, memory: 128Mi }
     limits:   { cpu: 500m, memory: 256Mi }

   autoscaling:
     enabled: true
     minReplicas: 3
     maxReplicas: 20
     targetCPUUtilizationPercentage: 60

   nodeSelector: {}
   tolerations: []
   affinity: {}

   # -- Application config
   config:
     logLevel: info
     feature: enabled
   ```
2. **Override at install/upgrade:**
   ```bash
   helm upgrade --install myapp ./myapp -n prod --create-namespace \
     -f values.yaml -f values.prod.yaml \
     --set replicaCount=5 \
     --set config.logLevel=warn
   ```
3. **Hands-on exercise:** Override replicaCount, resources, and ingress at install time. Render with template and confirm the change.

#### Step 3: Templates & The Go Templating Language
1. **The basic syntax:**
   - `{{ .Values.x }}` â€” value access
   - `{{ include "myapp.name" . }}` â€” call a named template
   - `{{- include "myapp.labels" . | nindent 4 }}` â€” pipe + indent
   - `{{ if .Values.ingress.enabled }}...{{ end }}` â€” conditional
   - `{{ range .Values.hosts }}...{{ end }}` â€” loop
2. **The Deployment template:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: {{ include "myapp.fullname" . }}
     labels:
       {{- include "myapp.labels" . | nindent 4 }}
   spec:
     {{- if not .Values.autoscaling.enabled }}
     replicas: {{ .Values.replicaCount }}
     {{- end }}
     selector:
       matchLabels:
         {{- include "myapp.selectorLabels" . | nindent 6 }}
     template:
       metadata:
         labels:
           {{- include "myapp.selectorLabels" . | nindent 8 }}
         {{- with .Values.podAnnotations }}
         annotations:
           {{- toYaml . | nindent 8 }}
         {{- end }}
       spec:
         {{- with .Values.imagePullSecrets }}
         imagePullSecrets:
           {{- toYaml . | nindent 8 }}
         {{- end }}
         securityContext:
           {{- toYaml .Values.podSecurityContext | nindent 8 }}
         containers:
           - name: {{ .Chart.Name }}
             securityContext:
               runAsNonRoot: true
               allowPrivilegeEscalation: false
               readOnlyRootFilesystem: true
               capabilities: { drop: ["ALL"] }
             image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
             imagePullPolicy: {{ .Values.image.pullPolicy }}
             ports:
               - name: http
                 containerPort: {{ .Values.service.targetPort }}
                 protocol: TCP
             livenessProbe:
               httpGet: { path: /healthz, port: http }
             readinessProbe:
               httpGet: { path: /readyz, port: http }
             resources:
               {{- toYaml .Values.resources | nindent 12 }}
             envFrom:
               - configMapRef: { name: {{ include "myapp.fullname" . }} }
   ```
3. **The `_helpers.tpl` â€” name the patterns:**
   ```gotemplate
   {{/* Expand the name of the chart. */}}
   {{- define "myapp.name" -}}
   {{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
   {{- end -}}

   {{- define "myapp.fullname" -}}
   {{- if .Values.fullnameOverride -}}
   {{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" -}}
   {{- else -}}
   {{- $name := default .Chart.Name .Values.nameOverride -}}
   {{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" -}}
   {{- end -}}
   {{- end -}}

   {{- define "myapp.labels" -}}
   app.kubernetes.io/name: {{ include "myapp.name" . }}
   app.kubernetes.io/instance: {{ .Release.Name }}
   app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
   app.kubernetes.io/managed-by: {{ .Release.Service }}
   helm.sh/chart: {{ printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" }}
   {{- end -}}
   ```
4. **The 10 gotchas you'll hit:**
   - `nil` vs empty map â€” always check before ranging
   - Whitespace â€” use `{{- ` and ` -}}` carefully
   - Indentation â€” use `| nindent N`
   - YAML strings need quoting: `{{ .Values.x | quote }}`
   - `lookup` to read live cluster state (use sparingly)
   - CRDs in `crds/` are installed once, not templated
   - `helm.sh/hook` for pre-install, post-upgrade jobs
   - Dependencies are installed to `charts/`
5. **Hands-on exercise:** Add a ConfigMap template, an HPA template, and a ServiceAccount template; render and confirm.

#### Step 4: Hooks â€” Pre/Post Install, Upgrade, Rollback
1. **The hook annotation:**
   ```yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: {{ include "myapp.fullname" . }}-migrate
     annotations:
       "helm.sh/hook": pre-upgrade,pre-install
       "helm.sh/hook-weight": "-5"
       "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
   spec:
     template:
       spec:
         restartPolicy: Never
         containers:
           - name: migrate
             image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
             command: ["./migrate.sh"]
   ```
2. **Hook types:**
   - `pre-install` â€” run before any resources
   - `post-install` â€” run after all resources
   - `pre-upgrade` â€” migrations
   - `post-upgrade` â€” smoke tests
   - `pre-delete` â€” graceful drain
   - `post-delete` â€” cleanup
   - `test` â€” `helm test` runs these
3. **Hands-on exercise:** Add a `pre-upgrade` Job that runs a database migration, and a `test` Pod that curls the deployed service.

#### Step 5: Dependencies & Library Charts
1. **The `Chart.yaml` deps:**
   ```yaml
   dependencies:
     - name: postgresql
       version: "16.x.x"
       repository: "https://charts.bitnami.com/bitnami"
       condition: postgresql.enabled
     - name: redis
       version: "20.x.x"
       repository: "https://charts.bitnami.com/bitnami"
       condition: redis.enabled
   ```
2. **Pull deps:**
   ```bash
   helm dependency update
   helm dependency list
   ```
3. **Library charts â€” share templates, no Deployment:**
   ```yaml
   # Chart.yaml of mylibrary
   type: library
   ```
4. **Hands-on exercise:** Add PostgreSQL + Redis as dependencies, render with both enabled and disabled.

#### Step 6: Helm OCI â€” The New Default
1. **Login to an OCI registry:**
   ```bash
   helm registry login ghcr.io -u USERNAME
   ```
2. **Package and push:**
   ```bash
   helm package ./myapp
   helm push myapp-1.2.3.tgz oci://ghcr.io/myorg/charts
   ```
3. **Install from OCI:**
   ```bash
   helm install myapp oci://ghcr.io/myorg/charts/myapp --version 1.2.3
   ```
4. **Hands-on exercise:** Package your chart, push to GHCR, and install from there.

#### Step 7: Production Patterns
1. **values per environment:**
   ```
   myapp/
   â”œâ”€â”€ values.yaml          (defaults)
   â”œâ”€â”€ values-dev.yaml
   â”œâ”€â”€ values-staging.yaml
   â””â”€â”€ values-prod.yaml
   ```
   ```bash
   helm upgrade --install myapp ./myapp -n prod -f values.yaml -f values-prod.yaml
   ```
2. **CI/CD integration â€” render in CI, apply in GitOps:**
   ```bash
   helm template myapp ./myapp -f values-prod.yaml > /tmp/manifests.yaml
   ```
3. **Atoms vs molecules â€” the chart layering pattern:**
   - **Atom charts:** one resource (e.g., `myapp-deployment`)
   - **Molecule charts:** compose atoms (e.g., `webapp` = deployment + service + ingress)
   - **Organism charts:** full app (e.g., `platform` = molecule + db + monitoring)
4. **Test your chart:**
   ```bash
   helm install myapp ./myapp --dry-run --debug
   helm install myapp ./myapp
   helm test myapp
   helm lint ./myapp
   ```
5. **Schema validation:**
   ```json
   // values.schema.json
   {
     "$schema": "http://json-schema.org/draft-07/schema#",
     "type": "object",
     "required": ["image"],
     "properties": {
       "replicaCount": { "type": "integer", "minimum": 1, "maximum": 50 },
       "image": {
         "type": "object",
         "required": ["repository"],
         "properties": {
           "repository": { "type": "string" },
           "tag": { "type": "string" }
         }
       }
     }
   }
   ```
6. **Hands-on exercise:** Add a schema, lint, test, and lint with the chart-testing tool (`ct lint`).

#### Step 8: Helmfile, Monocular, and the Ecosystem
1. **helmfile â€” declarative multi-chart releases:**
   ```yaml
   # helmfile.yaml
   releases:
     - name: cert-manager
       chart: jetstack/cert-manager
       version: 1.15.0
       namespace: cert-manager
       createNamespace: true
     - name: ingress-nginx
       chart: ingress-nginx/ingress-nginx
       version: 4.10.0
       namespace: ingress-nginx
     - name: myapp
       chart: ./myapp
       version: 1.2.3
       namespace: prod
       values:
         - values.yaml
         - values-prod.yaml
   ```
   ```bash
   helmfile sync
   ```
2. **chart-testing + chart-releaser â€” CI for charts:**
   - `ct lint` â€” runs `helm lint` on all charts
   - `ct install --all` â€” installs to a kind cluster
   - GitHub Action: `helm/chart-releaser-action`
3. **Hands-on exercise:** Use `helmfile` to install 3 charts (cert-manager, ingress-nginx, myapp) in order.

#### Step 9: Production-Grade Helm Habits
- **Pin every chart version** in `requirements.yaml` / `Chart.yaml`
- **Pin every image tag** in values
- **One chart per app** â€” don't combine unrelated services
- **Version your chart** with semver; bump on every change
- **Use `helm diff` in CI** to preview upgrades
- **Render in CI, apply via GitOps** â€” never `helm install` from a laptop
- **Lint, test, and schema-validate** on every PR
- **Document values in values.yaml** with `# --` comments
- **Provide sensible defaults** so `helm install` "just works"

### Practice Project â€” Production-Grade Helm Chart

> **Goal:** Package a real 3-service app as a polished, tested, multi-env Helm chart with dependencies, hooks, and CI.
> **Time estimate:** 6-8 hours
> **Deliverables:** A chart repo, OCI-published chart, and a CI workflow.

#### Phase 1 â€” Chart Skeleton (1h)
- [ ] **(B)** Create `charts/myapp/` with `helm create`
- [ ] **(B)** Restructure `values.yaml` with the schema from Step 2
- [ ] **(I)** Add `_helpers.tpl` for fullname, labels, selectorLabels
- [ ] **(I)** Add ConfigMap, HPA, ServiceAccount, NetworkPolicy, PDB templates

#### Phase 2 â€” Templating (1.5h)
- [ ] **(I)** Add conditionals for optional resources (autoscaling, ingress, serviceAccount)
- [ ] **(I)** Use `toYaml` and `nindent` everywhere
- [ ] **(I)** Add a NOTES.txt that prints the access URL
- [ ] **(A)** Add a helper to render a PodDisruptionBudget from values

#### Phase 3 â€” Hooks & Tests (1.5h)
- [ ] **(I)** Add a `pre-upgrade` Job for DB migrations
- [ ] **(I)** Add a `test-connection` Pod in `templates/tests/`
- [ ] **(I)** Run `helm test` against a real cluster
- [ ] **(A)** Add a `pre-delete` Job that drains in-flight work

#### Phase 4 â€” Values per Env (1h)
- [ ] **(I)** Create `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml`
- [ ] **(I)** Render each with `helm template` and diff
- [ ] **(I)** Install dev/staging to a kind cluster
- [ ] **(A)** Use `helmfile` to install both in order

#### Phase 5 â€” Schema, Lint, CI (1h)
- [ ] **(I)** Write `values.schema.json`; verify `helm lint` enforces it
- [ ] **(I)** Add `helm/chart-testing-action` to GitHub Actions
- [ ] **(I)** Add `helm/chart-releaser-action` to package + push to GHCR
- [ ] **(A)** Add a release workflow that opens a PR with version bumps on every chart change

#### Phase 6 â€” Polish (1h)
- [ ] **(I)** Add `helm-docs` to auto-generate README from values
- [ ] **(I)** Document every value with `# --` comments
- [ ] **(A)** Add a `Makefile` with: `lint`, `test`, `install`, `uninstall`, `package`, `push`
- [ ] **(A)** Add a `CHANGELOG.md` per chart with semantic-release

#### Bonus Challenges
- [ ] **(A)** Convert the chart to a library + molecule + organism structure
- [ ] **(A)** Implement multi-tenancy with chart values per tenant
- [ ] **(A)** Add a `policy` template (OPA/Kyverno) that enforces the chart's security baseline

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 22 tasks across 6 phases + 3 bonus challenges**

### Key Takeaways
- Helm is templating + releases + ecosystem â€” the package manager of K8s
- `values.yaml` is the public API of your chart; document every field
- Hooks let you run code before/after install, upgrade, delete
- OCI registries are the modern distribution model
- Always render in CI, always apply via GitOps

---

## ðŸ—ï¸ Terraform

**Difficulty:** Intermediate | **Time:** 30-35 hours

### Overview
Terraform is the de facto standard for declarative infrastructure as code. It describes the desired state of any cloud (AWS, Azure, GCP, K8s, Cloudflare, GitHub, Datadog, Vault â€” 3000+ providers) in HCL, plans the diff to reality, and applies it idempotently. State lives in a remote backend, modules let you compose, and a mature ecosystem of tools (terragrunt, atlantis, spacelift, env0) wrap it for teams. Mastering Terraform is mastering the cloud.

### Why Learn This
- 70%+ of cloud teams use Terraform or OpenTofu as their IaC
- The HCL model and state engine are the basis for OpenTofu, Pulumi, CloudFormation CDK, Crossplane
- Mastering state, modules, and workspaces is what makes IaC work in teams
- Security and compliance scanning (Snyk, Checkov, tfsec) runs against Terraform
- The same skills transfer to OpenTofu, Pulumi, and CDKTF

### Learn by Doing â€” Step by Step

#### Step 1: Install, First Resource, The Mental Model
1. **Install:**
   ```bash
   # Terraform
   wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
   echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
   sudo apt update && sudo apt install terraform

   # Or OpenTofu (the open-source fork)
   sudo apt install -y tofu

   terraform version
   tofu version
   ```
2. **The three commands you live by:**
   ```bash
   terraform init     # download providers, configure backends
   terraform plan     # diff desired vs actual
   terraform apply    # make it so (with confirmation)
   terraform destroy  # tear it down
   ```
3. **Your first config:**
   ```hcl
   terraform {
     required_version = ">= 1.9"
     required_providers {
       aws = {
         source  = "hashicorp/aws"
         version = "~> 5.70"
       }
     }
   }

   provider "aws" {
     region = "us-east-1"
   }

   resource "aws_s3_bucket" "logs" {
     bucket = "myorg-logs-${var.env}"
   }

   resource "aws_s3_bucket_versioning" "logs" {
     bucket = aws_s3_bucket.logs.id
     versioning_configuration { status = "Enabled" }
   }

   resource "aws_s3_bucket_server_side_encryption_configuration" "logs" {
     bucket = aws_s3_bucket.logs.id
     rule {
       apply_server_side_encryption_by_default {
         sse_algorithm = "AES256"
       }
     }
   }
   ```
4. **Variables and outputs:**
   ```hcl
   variable "env" {
     type        = string
     description = "Environment name"
     validation {
       condition     = contains(["dev", "staging", "prod"], var.env)
       error_message = "env must be dev, staging, or prod."
   }
   }
   output "bucket_name" { value = aws_s3_bucket.logs.bucket }
   ```
5. **The state â€” what makes Terraform different:**
   ```bash
   terraform state list
   terraform state show aws_s3_bucket.logs
   terraform state mv aws_s3_bucket.logs aws_s3_bucket.main
   terraform state rm aws_s3_bucket.logs
   terraform refresh   # reconcile state with reality
   ```
6. **Hands-on exercise:** Provision an S3 bucket (or any cloud resource) end-to-end; then `terraform destroy`.

#### Step 2: HCL â€” The Language In Depth
1. **Blocks, arguments, expressions:**
   ```hcl
   resource "type" "name" {
     arg = expression
   }
   ```
2. **The 5 expression types:**
   - **Literals:** `42`, `"hello"`, `true`, `null`, `["a", "b"]`, `{ key = "val" }`
   - **References:** `aws_s3_bucket.logs.id`
   - **Arithmetic / logical:** `count.index`, `!var.enabled`
   - **Conditionals:** `var.env == "prod" ? "gp3" : "standard"`
   - **For expressions:** `[for s in var.subnets : s.id]`
3. **Dynamic blocks â€” the underused feature:**
   ```hcl
   ingress {
     for_each = var.allowed_ports
     content {
       from_port   = ingress.value
       to_port     = ingress.value
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }
   }
   ```
4. **Functions â€” the 100+ you should know by category:**
   - **String:** `format`, `join`, `split`, `upper`, `lower`, `trimspace`, `substr`, `regex`
   - **Collection:** `length`, `merge`, `concat`, `flatten`, `distinct`, `keys`, `values`
   - **Numeric:** `max`, `min`, `ceil`, `floor`, `abs`
   - **Type:** `tostring`, `tonumber`, `tolist`, `tomap`, `try`
   - **Filesystem:** `file`, `fileexists`, `templatefile`, `filebase64`
   - **IP/CIDR:** `cidrsubnet`, `cidrhost`, `cidrnetmask`
   - **Date:** `formatdate`, `timeadd`
5. **The `for` expression â€” three forms:**
   ```hcl
   # list comprehension
   output "names" { value = [for u in var.users : u.name] }
   # map comprehension
   output "by_id" { value = { for u in var.users : u.id => u.name } }
   # with filter
   output "admins" { value = { for u in var.users : u.id => u if u.role == "admin" } }
   ```
6. **Hands-on exercise:** Generate 16 `/24` subnets inside a `/20` using `cidrsubnet`.

#### Step 3: State â€” The Heart of Terraform
1. **The two ways to store state:**
   - **Local:** default, `terraform.tfstate` in cwd
   - **Remote:** S3 + DynamoDB, Azure Storage, GCS, Terraform Cloud, Consul, etcd
2. **The S3 backend (the standard):**
   ```hcl
   terraform {
     backend "s3" {
       bucket         = "myorg-tfstate"
         key            = "prod/vpc/terraform.tfstate"
         region         = "us-east-1"
         dynamodb_table = "myorg-tflock"
         encrypt        = true
     }
   }
   ```
3. **The 6 rules of state:**
   1. **Never edit state by hand** â€” use `terraform state`
   2. **Never lose state** â€” back up the backend
   3. **Lock state during apply** â€” DynamoDB does this
   4. **Encrypt state at rest** â€” backend-side
   5. **One state per environment** â€” never share
   6. **Use `terraform import` to bring existing infra into TF**
4. **State operations:**
   ```bash
   terraform state list
   terraform state show <addr>
   terraform state mv <from> <to>
   terraform state rm <addr>
   terraform state pull > backup.tfstate
   terraform state push <file>
   terraform import <addr> <id>
   ```
5. **Hands-on exercise:** Configure an S3 + DynamoDB backend, import an existing resource, and prove state lock works by running two `apply` concurrently.

#### Step 4: Modules â€” The Building Blocks
1. **The 3 sources of modules:**
   - **Local:** `./modules/vpc`
   - **Git:** `git::https://github.com/myorg/modules.git//vpc?ref=v1.2.3`
   - **Registry:** `terraform-aws-modules/vpc/aws`
2. **A module's structure:**
   ```
   modules/vpc/
   â”œâ”€â”€ main.tf
   â”œâ”€â”€ variables.tf
   â”œâ”€â”€ outputs.tf
   â”œâ”€â”€ versions.tf
   â”œâ”€â”€ README.md
   â””â”€â”€ examples/
       â””â”€â”€ complete/
   ```
3. **The root module calling a child:**
   ```hcl
   module "vpc" {
     source  = "terraform-aws-modules/vpc/aws"
     version = "5.13.0"

     name = "prod"
     cidr = "10.0.0.0/16"
     azs  = ["us-east-1a", "us-east-1b", "us-east-1c"]
     public_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
     private_subnets = ["10.0.10.0/24", "10.0.20.0/24", "10.0.30.0/24"]
     enable_nat_gateway     = true
     single_nat_gateway     = false
     one_nat_gateway_per_az = true
     enable_dns_hostnames   = true
   }
   ```
4. **Module best practices:**
   - One purpose per module
   - Pin versions (use `?ref=v1.2.3` for git)
   - Document every variable and output
   - Provide examples
   - Use `for_each` / `count` to instantiate multiple
   - Don't pass `provider` aliases when avoidable
5. **Hands-on exercise:** Use the AWS VPC module; add a public and private subnet; expose the VPC ID as an output.

#### Step 5: Loops, count, for_each, and The Data Trick
1. **`count` â€” when you need indexed instances:**
   ```hcl
   resource "aws_iam_user" "this" {
     count = length(var.users)
     name  = var.users[count.index]
   }
   ```
2. **`for_each` â€” when you need a map or set:**
   ```hcl
   resource "aws_s3_bucket" "this" {
     for_each = toset(var.bucket_names)
     bucket   = each.value
   }
   ```
3. **The `for_each` gotcha:** set/map keys must be known at plan time.
4. **Dynamic resources â€” generate from data:**
   ```hcl
   data "aws_availability_zones" "available" { state = "available" }
   resource "aws_subnet" "public" {
     for_each = toset(data.aws_availability_zones.available.names)
     availability_zone = each.value
     cidr_block        = cidrsubnet(var.vpc_cidr, 8, index(data.aws_availability_zones.available.names, each.value))
     vpc_id            = aws_vpc.main.id
   }
   ```
5. **Hands-on exercise:** Create one S3 bucket per name in a list using `for_each`; add a lifecycle policy to each.

#### Step 6: Workspaces, Remote Backends, and Team Workflows
1. **Workspaces â€” multiple state files in one config:**
   ```bash
   terraform workspace new prod
   terraform workspace select prod
   terraform workspace list
   ```
2. **The 2025 consensus: don't use workspaces for env separation.** Use directory + backend key instead.
3. **The recommended structure:**
   ```
   infra/
   â”œâ”€â”€ modules/
   â”‚   â”œâ”€â”€ vpc/
   â”‚   â”œâ”€â”€ eks/
   â”‚   â””â”€â”€ rds/
   â”œâ”€â”€ live/
   â”‚   â”œâ”€â”€ dev/
   â”‚   â”‚   â”œâ”€â”€ main.tf
   â”‚   â”‚   â”œâ”€â”€ variables.tf
   â”‚   â”‚   â”œâ”€â”€ backend.hcl
   â”‚   â”‚   â””â”€â”€ prod.tfvars
   â”‚   â”œâ”€â”€ staging/
   â”‚   â””â”€â”€ prod/
   â””â”€â”€ README.md
   ```
4. **The backend config pattern:**
   ```bash
   terraform init -backend-config=backend.hcl
   # backend.hcl
   bucket         = "myorg-tfstate"
   key            = "live/prod/vpc/terraform.tfstate"
   region         = "us-east-1"
   dynamodb_table = "myorg-tflock"
   ```
5. **Hands-on exercise:** Replicate the standard structure; deploy dev/staging/prod into separate state files.

#### Step 7: CI/CD for Terraform â€” Atlantis, Spacelift, or DIY
1. **The pipeline:**
   ```
   PR opened â†’ terraform fmt -check â†’ terraform validate â†’ tflint â†’ checkov â†’ tfsec â†’ terraform plan
   PR merged â†’ terraform apply
   ```
2. **A minimal GitHub Actions workflow:**
   ```yaml
   # .github/workflows/tf.yml
   name: Terraform
   on: [pull_request]
   jobs:
     plan:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: hashicorp/setup-terraform@v3
           with: { terraform_version: 1.9.0 }
         - run: terraform fmt -check -recursive
         - run: terraform init -backend=false
         - run: terraform validate
         - uses: terraform-linters/setup-tflint@v4
           with: { tflint_version: latest }
         - run: tflint --recursive
         - uses: aquasecurity/trivy-action@master
           with:
             terraform_modules: true
             terraform_var_files: prod.tfvars
         - run: terraform plan -var-file=prod.tfvars -out=tfplan
         - uses: actions/upload-artifact@v4
           with: { name: tfplan, path: tfplan }
   ```
3. **Atlantis â€” Terraform automation server:**
   - `atlantis plan` on PR
   - `atlantis apply` on merge
   - Locking, policy, comments
4. **Spacelift / env0 / Scalr â€” SaaS for Terraform:**
   - Drift detection, policy as code, audit, self-service
5. **Hands-on exercise:** Add the GitHub Actions workflow; open a PR; review the plan; merge; verify apply.

#### Step 8: Security & Policy as Code
1. **tfsec / trivy â€” static analysis:**
   ```bash
   tfsec .
   trivy config .
   ```
2. **checkov â€” the deepest scanner:**
   ```bash
   checkov -d .
   checkov -f main.tf --framework terraform
   ```
3. **OPA / Conftest â€” generic policy:**
   ```rego
   # policy/s3.rego
   package terraform.s3
   deny[msg] {
     resource := input.resource.aws_s3_bucket[name]
     resource.acl == "public-read"
     msg := sprintf("bucket %s is public-read", [name])
   }
   ```
4. **Sentinel (Terraform Cloud/Enterprise) â€” HashiCorp's policy language.**
5. **Hands-on exercise:** Add a `checkov` step to CI; fix everything HIGH/CRITICAL; add an OPA rule that denies public S3 ACLs.

#### Step 9: Advanced Patterns
1. **`for_each` over a map with computed key:**
   ```hcl
   for_each = { for idx, n in var.subnets : n.name => n }
   ```
2. **Sensitive variables and outputs:**
   ```hcl
   variable "db_password" {
     type      = string
     sensitive = true
   }
   output "conn" {
     value     = "postgres://${var.user}:${var.db_password}@${aws_rds.main.endpoint}"
     sensitive = true
   }
   ```
3. **Provisioners â€” last resort, avoid:**
   ```hcl
   resource "aws_instance" "web" {
     provisioner "remote-exec" {
       inline = ["sudo apt update", "sudo apt install -y nginx"]
     }
   }
   ```
   - Use `user_data` (cloud-init) or Ansible instead.
4. **Move / refactor without destroy:**
   - `terraform state mv` to rename
   - `lifecycle { create_before_destroy = true }` for zero-downtime
   - `lifecycle { prevent_destroy = true }` for the must-not-touch
5. **Import existing infrastructure:**
   ```bash
   terraform import aws_s3_bucket.logs my-existing-bucket
   # Or use the new import block (TF 1.5+)
   ```
   ```hcl
   import {
     to = aws_s3_bucket.logs
     id = "my-existing-bucket"
   }
   ```
6. **Test with `terraform test` (TF 1.6+):**
   ```hcl
   # tests/vpc.tftest.hcl
   run "test_vpc" {
     command = plan
     assert {
       condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
       error_message = "VPC CIDR must be 10.0.0.0/16"
   }
   }
   ```
7. **Hands-on exercise:** Use `terraform test` to write 5 assertions on your VPC module; write an OPA policy that denies a public S3 ACL.

#### Step 10: Multi-Cloud & Cross-Provider Patterns
1. **Multiple providers in one config:**
   ```hcl
   provider "aws" { region = "us-east-1" }
   provider "github" {
     owner = "myorg"
     token = var.gh_token
   }
   ```
2. **Provider aliases â€” multi-region:**
   ```hcl
   provider "aws" {
     alias  = "east"
     region = "us-east-1"
   }
   provider "aws" {
     alias  = "west"
     region = "us-west-2"
   }
   resource "aws_s3_bucket" "east" { provider = aws.east ... }
   resource "aws_s3_bucket" "west" { provider = aws.west ... }
   ```
3. **Provider pass-through (in modules):**
   ```hcl
   module "eks" {
     source     = "./modules/eks"
     providers  = {
       aws = aws.east
     }
   }
   ```
4. **Cloud-agnostic abstractions â€” when not to:**
   - Most teams: write per-cloud modules
   - Multi-cloud platforms: abstract behind a `cloud` variable (with a clear cost)
5. **Hands-on exercise:** Deploy the same S3 bucket pattern to AWS and Azure (or GCP), and document the differences.

#### Step 11: OpenTofu, Terraform Cloud, and the Ecosystem
1. **OpenTofu â€” the open-source fork (since 2023):**
   - Drop-in compatible with most Terraform configs
   - Adds: early variable evaluation, provider-defined functions, state encryption
2. **Terraform Cloud / Enterprise:**
   - Remote state, runs, policy, registry, drift detection
   - Free tier for small teams
3. **Terragrunt â€” DRY wrapper around Terraform:**
   ```hcl
   # terragrunt.hcl
   remote_state {
     backend = "s3"
     config = {
       bucket = "myorg-tfstate"
       key    = "${path_relative_to_include()}/terraform.tfstate"
       region = "us-east-1"
     }
   }
   include "root" { path = find_in_parent_folders() }
   ```
4. **Pulumi â€” IaC in general-purpose languages:**
   - Same state model, same plan/apply
   - Python, TypeScript, Go
5. **Hands-on exercise:** Convert one module from Terraform to OpenTofu; install Terragrunt; configure remote state for 3 environments.

#### Step 12: Production-Grade Terraform Habits
- **State is sacred** â€” S3 + DynamoDB lock, encrypted, backed up
- **Pin provider and module versions** (`~> 5.70`, `?ref=v1.2.3`)
- **One state per env** â€” never share state across envs
- **Plan in CI, apply via merge** â€” no laptop applies
- **Lint + scan + validate on every PR** â€” fmt, validate, tflint, checkov, tfsec
- **Document every variable and output** â€” they're the public API
- **Use `lifecycle { prevent_destroy = true }` for the critical stuff**
- **Use modules for repeatability** â€” atomic, single-purpose, versioned
- **Drift detection** â€” `terraform plan` on a schedule
- **Test modules with `terraform test`**
- **Never put secrets in state** if you can avoid it â€” use data sources, Vault, or AWS SM

### Practice Project â€” Production Multi-Account Infrastructure

> **Goal:** Stand up a real production-shaped multi-account AWS (or Azure/GCP) infra with VPC, EKS/AKS/GKE, RDS, S3, IAM, and CI/CD.
> **Time estimate:** 20-25 hours
> **Deliverables:** A GitOps repo with modules + live, a CI pipeline, and a runbook.

#### Phase 1 â€” Foundation (2h)
- [ ] **(B)** Set up the live/dev, live/staging, live/prod structure â€” Step 6
- [ ] **(B)** Configure S3 + DynamoDB backend with state lock â€” Step 3
- [ ] **(I)** Add a Makefile: `init`, `plan`, `apply`, `destroy`, `fmt`, `validate`, `lint`
- [ ] **(I)** Add pre-commit with terraform fmt, tflint, checkov, tfsec

#### Phase 2 â€” Networking (2.5h)
- [ ] **(I)** Create a `modules/vpc` module with public + private subnets, NAT, flow logs
- [ ] **(I)** Use the module in dev/staging/prod with different CIDRs
- [ ] **(I)** Add VPC endpoints for S3 and DynamoDB
- [ ] **(A)** Add VPC peering to a second VPC and route between them

#### Phase 3 â€” Compute (3h)
- [ ] **(I)** Create a `modules/eks` module (or AKS/GKE) with managed node groups
- [ ] **(I)** Use the module in prod
- [ ] **(I)** Add Karpenter for autoscaling
- [ ] **(I)** Add IRSA (IAM Roles for Service Accounts) for one workload

#### Phase 4 â€” Data (2.5h)
- [ ] **(I)** Create a `modules/rds` module for Postgres
- [ ] **(I)** Use a DB subnet group, encryption at rest, deletion protection
- [ ] **(I)** Use `aws_secretsmanager` for the password; reference from the cluster
- [ ] **(A)** Add a read replica in a second region

#### Phase 5 â€” Storage & CDN (1.5h)
- [ ] **(I)** Create a `modules/s3` module with versioning, encryption, lifecycle, public access block
- [ ] **(I)** Add CloudFront (or Cloud CDN) in front of an S3 static site
- [ ] **(I)** Use a `for_each` to create 3 buckets from a map
- [ ] **(A)** Add cross-region replication for the logs bucket

#### Phase 6 â€” IAM & Security (2h)
- [ ] **(I)** Use a `modules/iam` to create least-privilege roles
- [ ] **(I)** Add an OPA / Conftest policy that denies public S3 ACLs
- [ ] **(I)** Enable AWS Config rules; check the S3-bucket-public-read-prohibited rule
- [ ] **(A)** Add SCPs (Service Control Policies) at the OU level

#### Phase 7 â€” Observability (1.5h)
- [ ] **(I)** Deploy the Prometheus + Grafana module to EKS (or equivalent)
- [ ] **(I)** Configure CloudWatch log groups for RDS, Lambda, etc.
- [ ] **(A)** Add a CloudWatch alarm on RDS CPU; route to SNS â†’ PagerDuty (or email)

#### Phase 8 â€” CI/CD (2h)
- [ ] **(I)** Add the GitHub Actions workflow from Step 7
- [ ] **(I)** Configure OIDC between GitHub and AWS (no static keys)
- [ ] **(I)** Implement the prod-apply manual approval gate
- [ ] **(A)** Add Atlantis (or Spacelift) for plan-on-PR + apply-on-merge

#### Phase 9 â€” DR & Operations (1.5h)
- [ ] **(I)** Add a `make drift-check` target that runs `terraform plan` on a schedule
- [ ] **(I)** Backup the state bucket; document the restore procedure
- [ ] **(A)** Add a canary deployment to one workload (CodeDeploy or Argo Rollouts)
- [ ] **(A)** Implement cluster recovery with Velero (or equivalent)

#### Phase 10 â€” Documentation (1h)
- [ ] **(I)** Write `README.md` per module and per env
- [ ] **(I)** Write `RUNBOOK.md`: plan, apply, destroy, debug state, restore
- [ ] **(A)** Add `terraform-docs` to auto-generate variable docs
- [ ] **(A)** Write a `MIGRATION.md` for OpenTofu

#### Bonus Challenges
- [ ] **(A)** Add Terragrunt on top of the structure
- [ ] **(A)** Add a `terraform test` suite with 20+ assertions
- [ ] **(A)** Implement a self-service module catalog with a generated web UI

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 24 tasks across 10 phases + 3 bonus challenges**

### Key Takeaways
- Terraform = HCL + state + providers; the state is the real artifact
- Modules are the unit of reuse; pin everything
- One state per environment; never share
- Plan in CI, apply via GitOps; never apply from a laptop
- Scan on every PR; prevent destroy on the critical stuff
- The same model works in OpenTofu and is the basis for Pulumi and CDKTF

---

## ðŸ”§ Ansible

**Difficulty:** Intermediate | **Time:** 20-25 hours

### Overview
Ansible is the de facto standard for agentless configuration management. It connects over SSH (or WinRM), pushes small Python modules to each target, and ensures the system matches the declared state. It is also widely used for orchestration (running an ordered play across many hosts), app deployment, network device config, and security baseline enforcement. Mastery of Ansible means mastery of declarative ops at any scale â€” from a single VM to 10,000.

### Why Learn This
- 60%+ of enterprise ops teams use Ansible or its descendants
- It is the standard for network automation (Cisco, Arista, Juniper, F5)
- The Red Hat ecosystem (RHEL, OpenShift, AAP) is built around Ansible
- Playbooks are the most readable form of ops code â€” engineers can review them
- AWX / Ansible Automation Platform is the open-core controller for large teams

### Why Learn This
- Same as before + agentless, push model, fast to start
- Idempotency and YAML make it auditable and reviewable
- Jinja2 templating is the same engine as Terraform, Salt, and many other tools

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model & First Playbook
1. **The three concepts:**
   - **Inventory:** the list of hosts
   - **Playbook:** the desired state in YAML
   - **Module:** the atomic unit of work (apt, copy, service, etc.)
2. **Install:**
   ```bash
   sudo apt install -y ansible
   ansible --version
   ```
3. **The minimal inventory:**
   ```ini
   # inventory/hosts.ini
   [web]
   web1.example.com ansible_host=10.0.0.11
   web2.example.com ansible_host=10.0.0.12

   [db]
   db1.example.com ansible_host=10.0.0.21

   [prod:children]
   web
   db

   [all:vars]
   ansible_user=ubuntu
   ansible_ssh_private_key_file=~/.ssh/id_ed25519
   ```
4. **Ad-hoc commands â€” your fastest feedback:**
   ```bash
   ansible all -i inventory/hosts.ini -m ping
   ansible web -i inventory/hosts.ini -m shell -a "uptime"
   ansible web -i inventory/hosts.ini -m apt -a "name=nginx state=present" --become
   ```
5. **Your first playbook:**
   ```yaml
   # playbooks/nginx.yml
   - name: Install and start nginx
     hosts: web
     become: true
     tasks:
       - name: Install nginx
         ansible.builtin.apt:
           name: nginx
           state: present
           update_cache: true
       - name: Ensure nginx is running
         ansible.builtin.service:
           name: nginx
           state: started
           enabled: true
   ```
6. **Run it:**
   ```bash
   ansible-playbook -i inventory/hosts.ini playbooks/nginx.yml
   # check mode (dry-run)
   ansible-playbook -i inventory/hosts.ini playbooks/nginx.yml --check
   # diff mode
   ansible-playbook -i inventory/hosts.ini playbooks/nginx.yml --check --diff
   # limit
   ansible-playbook -i inventory/hosts.ini playbooks/nginx.yml --limit web1
   # syntax check
   ansible-playbook -i inventory/hosts.ini playbooks/nginx.yml --syntax-check
   ```
7. **Hands-on exercise:** Install nginx across 2 VMs with a single playbook; verify it serves a page.

#### Step 2: The Module Library â€” 100+ You'll Use
1. **The categories:**
   | Category | Examples |
   |----------|----------|
   | Package | `apt`, `yum`, `dnf`, `pip`, `npm` |
   | File | `copy`, `template`, `file`, `lineinfile`, `fetch` |
   | Service | `service`, `systemd` |
   | User | `user`, `group` |
   | Network | `uri`, `get_url`, `firewalld`, `ufw` |
   | Cloud | `amazon.aws.ec2`, `azure.azcollection`, `google.cloud.gcp_compute` |
   | Docker | `community.docker.docker_container` |
   | K8s | `kubernetes.core.k8s` |
   | DB | `community.postgresql.postgresql_db` |
   | Crypto | `community.crypto.*` |
2. **Use the FQCN (`ansible.builtin.*`):** always â€” protects against collection shadowing.
3. **Hands-on exercise:** Use 5 different modules in one playbook: copy a file, template a config, ensure a service, open a port, create a user.

#### Step 3: Variables, Facts & Precedence
1. **Variable sources (lowest to highest precedence):**
   ```
   role defaults â†’ group_vars â†’ host_vars â†’ play vars â†’ extra-vars (-e) â†’ task vars
   ```
2. **Define them:**
   ```yaml
   # group_vars/all.yml
   nginx_port: 80
   ```
   ```yaml
   # group_vars/web.yml
   nginx_worker_processes: 4
   ```
   ```yaml
   # host_vars/web1.yml
   nginx_worker_connections: 1024
   ```
3. **Facts â€” discovered at runtime:**
   ```bash
   ansible web -i inventory/hosts.ini -m setup
   ansible web -i inventory/hosts.ini -m setup --filter ansible_eth0
   ```
   - Disable with `gather_facts: false` when not needed
4. **Jinja2 templating:**
   ```yaml
   - name: Render nginx config
     ansible.builtin.template:
       src: nginx.conf.j2
       dest: /etc/nginx/nginx.conf
       owner: root
       mode: "0644"
   ```
   ```jinja
   # templates/nginx.conf.j2
   user www-data;
   worker_processes {{ nginx_worker_processes | default(ansible_processor_vcpus) }};
   events { worker_connections {{ nginx_worker_connections }}; }
   http {
     server {
       listen {{ nginx_port }};
       server_name {{ inventory_hostname }};
       {% for loc in nginx_locations %}
       location {{ loc.path }} { proxy_pass {{ loc.upstream }}; }
       {% endfor %}
     }
   }
   ```
5. **Hands-on exercise:** Use facts + variables + Jinja to render a per-host nginx config that includes the hostname in the response.

#### Step 4: Conditionals, Loops, Handlers & Blocks
1. **Conditionals:**
   ```yaml
   - name: Install redis on Debian
     ansible.builtin.apt:
       name: redis
       state: present
     when: ansible_os_family == "Debian"
   ```
2. **Loops:**
   ```yaml
   - name: Create users
     ansible.builtin.user:
       name: "{{ item.name }}"
       groups: "{{ item.groups | default([]) }}"
     loop:
       - { name: alice, groups: [sudo, docker] }
       - { name: bob }
   ```
3. **Handlers â€” run only on change:**
   ```yaml
   tasks:
     - name: Copy nginx config
       ansible.builtin.template:
         src: nginx.conf.j2
         dest: /etc/nginx/nginx.conf
       notify: reload nginx
   handlers:
     - name: reload nginx
       ansible.builtin.service:
         name: nginx
         state: reloaded
   ```
4. **Blocks â€” group with `when` / `rescue` / `always`:**
   ```yaml
   - name: Deploy with rollback
     block:
       - name: Upload new code
         ansible.builtin.copy: ...
       - name: Restart service
         ansible.builtin.service: ...
     rescue:
       - name: Roll back
         ansible.builtin.copy: ... # previous version
     always:
       - name: Notify
         ansible.builtin.slack: ...
   ```
5. **Hands-on exercise:** Write a playbook that creates 3 users from a list, with handlers to restart sshd when `/etc/ssh/sshd_config` changes.

#### Step 5: Roles â€” The Reuse Pattern
1. **The standard role layout:**
   ```
   roles/
   â””â”€â”€ nginx/
       â”œâ”€â”€ defaults/main.yml
       â”œâ”€â”€ files/
       â”œâ”€â”€ handlers/main.yml
       â”œâ”€â”€ meta/main.yml
       â”œâ”€â”€ tasks/main.yml
       â”œâ”€â”€ templates/
       â”œâ”€â”€ vars/main.yml
       â””â”€â”€ README.md
   ```
2. **Create one with `ansible-galaxy`:**
   ```bash
   ansible-galaxy init roles/nginx
   ```
3. **Use a role:**
   ```yaml
   # site.yml
   - hosts: web
     become: true
     roles:
       - nginx
   ```
4. **Galaxy â€” the public registry:**
   ```bash
   ansible-galaxy collection install community.docker
   ansible-galaxy role install geerlingguy.docker
   ```
5. **A `meta/main.yml` for documentation:**
   ```yaml
   galaxy_info:
     role_name: nginx
     author: you
     description: Production nginx role
     min_ansible_version: "2.15"
     license: MIT
     platforms: [{ name: Ubuntu, versions: [focal, jammy] }]
   dependencies: []
   ```
6. **Hands-on exercise:** Convert the nginx playbook into a role; call it from a site playbook.

#### Step 6: Ansible Vault â€” Secrets
1. **Encrypt:**
   ```bash
   ansible-vault create secrets.yml
   ansible-vault edit secrets.yml
   ansible-vault encrypt existing.yml
   ansible-vault view secrets.yml
   ```
2. **Use a password file in CI:**
   ```bash
   ansible-vault view --vault-password-file ~/.vault_pass secrets.yml
   ```
3. **Encrypt in a playbook:**
   ```yaml
   - hosts: web
     vars_files:
       - secrets.yml
     tasks:
       - name: Use secret
         ansible.builtin.debug:
           msg: "password is {{ db_password }}"
   ```
4. **The 2026 recommendation:** don't keep secrets in Vault at rest in the repo. Use **Ansible Vault with AWS KMS / Azure KV / GCP KMS** to encrypt with a cloud-managed key. Or fetch at runtime from Vault / AWS SM.
5. **Hands-on exercise:** Vault a YAML file with a fake DB password; reference it in a playbook; decrypt with `--vault-password-file`.

#### Step 7: Dynamic Inventory & The Cloud
1. **AWS dynamic inventory â€” the plugin:**
   ```yaml
   # inventory/aws_ec2.yml
   plugin: amazon.aws.aws_ec2
   regions: [us-east-1]
   filters:
     tag:Env: prod
   keyed_groups:
     - key: tags.Role
       prefix: role
   groups:
     web: "'web' in tags.Role"
     db:  "'db' in tags.Role"
   ```
2. **Use it:**
   ```bash
   ansible-inventory -i inventory/aws_ec2.yml --graph
   ansible-playbook -i inventory/aws_ec2.yml site.yml
   ```
3. **Other plugins:** `azure.azcollection.azure_rm`, `google.cloud.gcp_compute`, `community.kubernetes.k8s`.
4. **Hands-on exercise:** Use the AWS dynamic inventory to fetch 5 tagged instances; render a config per host.

#### Step 8: Advanced Execution
1. **Forks, serial, throttle:**
   ```yaml
   - hosts: web
     serial: 30%        # rolling batches
     throttle: 10       # at most 10 parallel per host
   ```
2. **Strategies:**
   - `linear` (default) â€” one host at a time, all tasks
   - `free` â€” each host finishes independently
   - `debug` â€” dry run with step-by-step
3. **Delegation, run_once, delegate_to:**
   ```yaml
   - name: Run only on the first host
     ansible.builtin.command: deploy.sh
     run_once: true
     delegate_to: web1
   ```
4. **Async / poll â€” for long tasks:**
   ```yaml
   - name: Long task
     ansible.builtin.command: ./long_job.sh
     async: 3600
     poll: 0
   ```
5. **Hands-on exercise:** Use `serial: 30%` to roll out an app across 10 hosts in batches; use `run_once` to update a load balancer.

#### Step 9: Collections & The Ecosystem
1. **The collection list you need:**
   - `community.general`
   - `community.docker`
   - `community.kubernetes` (renamed `kubernetes.core`)
   - `community.crypto`
   - `community.postgresql`
   - `community.mysql`
   - `amazon.aws`
   - `azure.azcollection`
   - `google.cloud`
2. **Install from a `requirements.yml`:**
   ```yaml
   collections:
     - name: amazon.aws
       version: 8.0.0
     - name: community.docker
       version: 3.6.0
   roles:
     - name: geerlingguy.docker
       version: 7.0.0
   ```
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```
3. **Hands-on exercise:** Install 3 collections, use one module from each in a real playbook.

#### Step 10: Testing â€” molecule, ansible-lint, CI
1. **molecule â€” the test framework:**
   ```bash
   python3 -m pip install --user molecule molecule-plugins[docker] ansible-lint
   molecule init role myorg.nginx -d docker
   cd ~/.ansible/roles/myorg.nginx
   molecule test
   ```
2. **The molecule scenarios:**
   - `default` â€” converge + idempotence + verify
   - `side_effect` â€” post-converge checks
   - `regression` â€” confirm idempotency
3. **ansible-lint â€” the linter:**
   ```bash
   ansible-lint playbooks/
   ```
4. **CI integration:**
   ```yaml
   - uses: actions/checkout@v4
   - uses: ansible/ansible-lint-action@v6
   - run: molecule test
   ```
5. **Hands-on exercise:** Set up molecule for your nginx role; add 3 verify checks; run in CI.

#### Step 11: Ansible Automation Platform (AAP) & AWX
1. **AWX â€” the open-source upstream:**
   - Web UI + REST API + RBAC + scheduling
   - Run playbooks from the UI, audit them, give teams self-service
2. **AAP â€” the enterprise distribution:**
   - Adds: workflows, mesh, automation analytics, EDA, RHEL support
3. **The patterns:**
   - **Job template:** a single playbook run
   - **Workflow:** a DAG of job templates
   - **Inventory source:** dynamic from cloud
   - **Credentials:** vault + cloud + GitHub
4. **Hands-on exercise:** Install AWX in Docker, create a project, add a job template, run it.

#### Step 12: Production-Grade Ansible Habits
- **Always use FQCN** (`ansible.builtin.copy`)
- **Always use roles** for anything > 1 playbook
- **Always use a dynamic inventory** in the cloud
- **Always keep state external** â€” don't store facts in playbooks
- **Always lint, always test with molecule**
- **Always use Vault, never plain-text secrets**
- **Document roles with README, defaults, and examples**
- **Idempotency first** â€” if it can't be rerun safely, refactor
- **Use `ansible-lint` profiles** â€” `moderate` for prod, `min` for prototypes
- **Use `strategy: free`** for web tiers; `serial: 25%` for DBs

### Practice Project â€” Production-Grade Ansible Automation

> **Goal:** Stand up a real, versioned, tested, CI-validated Ansible repo that manages a 10-host web tier, a 3-host DB tier, and 2 load balancers.
> **Time estimate:** 12-15 hours
> **Deliverables:** A repo with 4 roles, a site playbook, AWX-compatible config, molecule tests, and a CI pipeline.

#### Phase 1 â€” Repo & Inventory (1.5h)
- [ ] **(B)** Create `ansible-infra/` with `inventory/`, `playbooks/`, `roles/`, `group_vars/`, `host_vars/`
- [ ] **(B)** Add `requirements.yml` for collections and roles
- [ ] **(I)** Build a static inventory for 5 web + 2 db + 1 lb
- [ ] **(I)** Set up `ansible.cfg` with roles_path, host_key_checking, retry files

#### Phase 2 â€” First Role: Common (1.5h)
- [ ] **(I)** Create `roles/common/` with: packages, timezone, NTP, user, SSH hardening
- [ ] **(I)** Use a Jinja2 template for `/etc/ssh/sshd_config`
- [ ] **(I)** Add a handler to restart sshd
- [ ] **(A)** Add a kustomize-style overrides pattern for per-env defaults

#### Phase 3 â€” Web Tier Role (2h)
- [ ] **(I)** Create `roles/nginx/` with: install, config, sites, TLS, logrotate
- [ ] **(I)** Use `for` loop to render multiple server blocks
- [ ] **(I)** Add an HAProxy role with backend health checks
- [ ] **(A)** Add a firewalld / ufw role that opens only what's needed

#### Phase 4 â€” DB Tier Role (2h)
- [ ] **(I)** Create `roles/postgres/` with: install, init, config, replication
- [ ] **(I)** Add a backup role that runs `pg_dump` + uploads to S3
- [ ] **(A)** Add a Patroni role for HA (or use `community.postgresql` with a master/replica)

#### Phase 5 â€” Site Playbook (1.5h)
- [ ] **(I)** Build `site.yml` with plays per group, in order: common â†’ lb â†’ web â†’ db
- [ ] **(I)** Use `serial: 30%` on the web tier
- [ ] **(I)** Use `run_once` for one-time tasks (deploy notifications, etc.)
- [ ] **(A)** Add a `rolling-upgrade.yml` that takes a new image tag and rolls in batches

#### Phase 6 â€” Vault & Secrets (1h)
- [ ] **(I)** Create `group_vars/db/vault.yml` with the db password
- [ ] **(I)** Add a `~/.vault_pass` file; reference in `ansible.cfg`
- [ ] **(A)** Wire AWS KMS to encrypt the Vault (or fetch from AWS SM at runtime)

#### Phase 7 â€” Testing (2h)
- [ ] **(I)** Add molecule scenarios for the 4 roles
- [ ] **(I)** Add verify steps: `curl` the nginx page, `psql` to the db, `pg_isready`
- [ ] **(I)** Add ansible-lint to CI; require `moderate` profile
- [ ] **(A)** Add a side-effect test that fails the role when an upstream regression occurs

#### Phase 8 â€” CI/CD (1.5h)
- [ ] **(I)** GitHub Actions: lint â†’ molecule test â†’ syntax check
- [ ] **(I)** Add a `--check` run on every PR
- [ ] **(A)** Deploy via AWX job template; trigger from CI on merge
- [ ] **(A)** Add a `make` target set: `bootstrap`, `deploy`, `rollback`, `check`

#### Phase 9 â€” AWX/AAP (1h)
- [ ] **(A)** Install AWX in Docker
- [ ] **(A)** Create a project pointing at the Git repo
- [ ] **(A)** Create an inventory, credentials, and a job template for `site.yml`
- [ ] **(A)** Run a deploy from the AWX UI

#### Phase 10 â€” Documentation (1h)
- [ ] **(B)** Write a top-level `README.md` with the architecture diagram
- [ ] **(I)** Write a per-role `README.md` with usage examples
- [ ] **(I)** Write `RUNBOOK.md` with: bootstrap, deploy, rollback, debug
- [ ] **(A)** Document the Vault KMS integration and rotation procedure

#### Bonus Challenges
- [ ] **(A)** Convert one role to use `ansible-core` execution environment containers
- [ ] **(A)** Add a network role that configures Cisco / Arista switches
- [ ] **(A)** Build a self-service portal on top of AWX for non-engineers

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 24 tasks across 10 phases + 3 bonus challenges**

### Key Takeaways
- Ansible = inventory + playbooks + modules; push model, agentless
- Roles are the unit of reuse; molecule + ansible-lint are the test/lint bar
- Dynamic inventory is the only way to manage a cloud fleet
- Vault is the secrets tool, but pair it with KMS for production
- AWX/AAP gives you the control plane: RBAC, scheduling, audit, workflows

---

## ðŸ” CI/CD â€” Jenkins, GitLab CI, GitHub Actions

**Difficulty:** Intermediate | **Time:** 30-35 hours

### Overview
Continuous Integration and Continuous Delivery (CI/CD) is the automation layer that turns a commit into a production change. The three dominant systems â€” Jenkins (the granddaddy, self-hosted, plugin-rich), GitLab CI (integrated with GitLab, powerful pipelines-as-code), and GitHub Actions (the cloud-native, marketplace-rich challenger) â€” all implement the same core model: on every change, run a pipeline of stages (build, test, scan, package, deploy). This tool covers all three, the patterns they share, and the production-grade behaviors that make them reliable.

### Why Learn This
- CI/CD is the backbone of every modern software delivery
- Each system has its own philosophy â€” knowing all three makes you platform-flexible
- The patterns (caching, artifacts, secrets, matrix builds, environments) transfer across systems
- Pipelines are now code â€” reviewable, versioned, testable
- Production-grade pipelines are secure (least-privilege, ephemeral, signed) and fast (caching, parallelism)

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The CI/CD loop:**
   ```
   commit â†’ build â†’ test â†’ scan â†’ package â†’ sign â†’ deploy â†’ observe
   ```
2. **The shared concepts across all 3 systems:**
   | Concept | What it means |
   |---------|---------------|
   | Trigger | Push, PR, schedule, webhook, manual |
   | Stage | A logical group (build, test) |
   | Step / Job / Action | A unit of work |
   | Runner / Agent / Controller | The machine that executes |
   | Artifact | A file produced and reused (image, binary) |
   | Cache | Reused data (deps) to speed builds |
   | Secret | Injected env var, never logged |
   | Environment | A deployment target (dev, staging, prod) |
   | Approval | A manual gate before prod |
   | Matrix | Multiple variants in parallel (OS, version) |
3. **Trunk-based + CI/CD = the modern default.** Every merge to `main` is potentially deployable.
4. **Hands-on exercise:** Pick a small app; write a CI pipeline in YAML; push a commit; watch the pipeline run.

#### Step 2: GitHub Actions â€” The Cloud-Native Default
1. **The minimum workflow:**
   ```yaml
   # .github/workflows/ci.yml
   name: CI
   on:
     push:
       branches: [main]
     pull_request:
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-node@v4
           with:
             node-version: 20
             cache: 'npm'
         - run: npm ci
         - run: npm test
   ```
2. **The 15 actions you need on day one:**
   - `actions/checkout@v4`
   - `actions/setup-node|setup-python|setup-go|setup-java`
   - `actions/cache@v4`
   - `actions/upload-artifact@v4` / `actions/download-artifact@v4`
   - `docker/build-push-action@v5`
   - `aquasecurity/trivy-action`
   - `sigstore/cosign-installer`
   - `aws-actions/configure-aws-credentials@v4`
   - `azure/login@v2`
   - `google-github-actions/auth@v2`
   - `github/codeql-action`
   - `softprops/action-gh-release`
3. **The matrix build:**
   ```yaml
   strategy:
     fail-fast: false
     matrix:
       os: [ubuntu-latest, macos-latest, windows-latest]
       node: [18, 20, 22]
   runs-on: ${{ matrix.os }}
   steps:
     - uses: actions/setup-node@v4
       with: { node-version: ${{ matrix.node }} }
   ```
4. **Caching:**
   ```yaml
   - uses: actions/cache@v4
     with:
       path: |
         ~/.npm
         ~/.cache/pip
       key: ${{ runner.os }}-${{ hashFiles('**/lockfiles') }}
   ```
5. **Reusable workflows â€” DRY:**
   ```yaml
   # .github/workflows/reusable-build.yml
   on:
     workflow_call:
       inputs:
         image:
           required: true
           type: string
   ```
   ```yaml
   # caller
   uses: myorg/.github/.github/workflows/reusable-build.yml@main
   with: { image: myapp }
   ```
6. **Composite actions â€” package your own steps:**
   ```yaml
   # action.yml
   name: Setup
   description: My setup
   runs:
     using: composite
     steps:
       - run: echo hello
         shell: bash
   ```
7. **Hands-on exercise:** Set up a Node app with matrix build (3 OS Ã— 3 Node versions), cache npm, and publish a release artifact.

#### Step 3: GitHub Actions â€” Production-Grade
1. **OIDC to the cloud â€” no static keys:**
   ```yaml
   permissions:
     id-token: write
     contents: read
   steps:
     - uses: aws-actions/configure-aws-credentials@v4
       with:
         role-to-assume: arn:aws:iam::123456789012:role/github-actions
         aws-region: us-east-1
   ```
2. **Environments with protection rules + secrets:**
   ```yaml
   environment:
     name: production
     url: https://app.example.com
   ```
   (Define `production` in Settings â†’ Environments; add reviewers, wait timer, secrets.)
3. **The build â†’ scan â†’ sign â†’ push pipeline:**
   ```yaml
   jobs:
     build:
       steps:
         - uses: docker/build-push-action@v5
           with: { push: false, tags: myapp:${{ github.sha }} }
         - name: Trivy scan
           uses: aquasecurity/trivy-action@master
           with:
             image-ref: myapp:${{ github.sha }}
             severity: 'CRITICAL,HIGH'
             exit-code: '1'
         - name: Install cosign
           uses: sigstore/cosign-installer@v3
         - name: Sign image
           env: { COSIGN_KEY: ${{ secrets.COSIGN_KEY }} }
           run: cosign sign --key env://COSIGN_KEY myapp:${{ github.sha }}
         - name: Push
           run: docker push myapp:${{ github.sha }}
   ```
4. **Self-hosted runners â€” when you need them:**
   - In a VM, in K8s (actions-runner-controller), in a cloud (ARC, runner.scale)
   - Use ephemeral, label-scheduled runners
5. **Concurrency â€” don't run two of the same:**
   ```yaml
   concurrency:
     group: ${{ github.workflow }}-${{ github.ref }}
     cancel-in-progress: true
   ```
6. **Hands-on exercise:** Add OIDC, env protection, signing, and concurrency to the build pipeline.

#### Step 4: GitLab CI â€” Integrated Power
1. **The minimum `.gitlab-ci.yml`:**
   ```yaml
   stages: [build, test, deploy]

   build:
     stage: build
     image: node:20
     script:
       - npm ci
       - npm run build
     artifacts:
       paths: [dist/]

   test:
     stage: test
     image: node:20
     script:
       - npm ci
       - npm test
     coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

   deploy:prod:
     stage: deploy
     image: bitnami/kubectl
     script:
       - kubectl apply -f k8s/
     environment:
       name: production
     when: manual
     only: [main]
   ```
2. **The DAG â€” needs / dependencies:**
   ```yaml
   build:app:
     stage: build
   build:docs:
     stage: build
   test:integration:
     stage: test
     needs: [build:app, build:docs]
   ```
3. **The `extends` + `include` â€” DRY:**
   ```yaml
   # .gitlab/ci/build.yml
   .build_template:
     image: node:20
     before_script: [npm ci]
   ```
   ```yaml
   include: [.gitlab/ci/build.yml]
   test:
     extends: .build_template
     script: [npm test]
   ```
4. **Parent-child pipelines â€” for very large repos.**
5. **Hands-on exercise:** Build a 3-stage pipeline (build, test, deploy) with artifacts and an environment.

#### Step 5: GitLab CI â€” Production-Grade
1. **CI/CD variables â€” the four scopes:**
   - Instance (admin)
   - Group
   - Project
   - Pipeline (set in `variables:`)
   - **Protected + masked** for secrets
2. **Runners â€” shared, group, project, instance:**
   ```toml
   # /etc/gitlab-runner/config.toml
   [[runners]]
     name = "docker-runner"
     url = "https://gitlab.com/"
     token = "..."
     executor = "docker"
     [runners.docker]
       image = "alpine"
       privileged = false
       volumes = ["/cache"]
   ```
3. **The auto-scaling runner with machine-learning:**
   - `docker-machine` executor
   - `kubernetes` executor (runners as pods)
   - `instance` executor
4. **The deploy board + canary with `canary`/`production` environments.**
5. **Hands-on exercise:** Register a group runner, set protected/masked variables, run a deploy with manual approval.

#### Step 6: Jenkins â€” The Workhorse
1. **Install + first pipeline:**
   ```bash
   # Docker
   docker run -d -p 8080:8080 -v jenkins_data:/var/jenkins_home \
     --name jenkins jenkins/jenkins:lts
   ```
2. **The Jenkinsfile (declarative):**
   ```groovy
   pipeline {
     agent any
     stages {
       stage('Build') {
         steps { sh 'npm ci && npm run build' }
       }
       stage('Test') {
         steps { sh 'npm test' }
       }
       stage('Scan') {
         steps {
           sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
         }
       }
       stage('Deploy') {
         when { branch 'main' }
         steps {
           input 'Deploy to production?'
           sh './deploy.sh production'
         }
       }
     }
   }
   ```
3. **The shared library â€” for DRY:**
   ```groovy
   // vars/buildNode.groovy
   def call(String version) {
     sh "nvm use ${version} && npm ci"
   }
   ```
   ```groovy
   // Jenkinsfile
   @Library('my-shared-lib') _
   buildNode('20')
   ```
4. **The shared library structure:**
   ```
   shared-lib/
   â”œâ”€â”€ vars/        # buildNode.groovy
   â”œâ”€â”€ src/         # org/myorg/Pipeline.groovy
   â”œâ”€â”€ resources/   # templates, files
   â””â”€â”€ README.md
   ```
5. **The agents â€” declarative:**
   ```groovy
   pipeline {
     agent { label 'linux && docker' }
   }
   ```
6. **Hands-on exercise:** Stand up Jenkins in Docker, run a 3-stage pipeline, install 2 plugins (Pipeline, Blue Ocean).

#### Step 7: Jenkins â€” Production-Grade
1. **The controller-agent architecture:**
   - Controller = UI + scheduling
   - Agents = workers (static, JNLP, Kubernetes, EC2)
2. **The Kubernetes plugin â€” agents as pods:**
   ```groovy
   agent {
     kubernetes {
       label 'myapp'
       yaml '''
       apiVersion: v1
       kind: Pod
       spec:
         containers:
         - name: jnlp
           image: jenkins/inbound-agent:latest
         - name: node
           image: node:20
           command: [cat]
           tty: true
       '''
     }
   }
   ```
3. **Credentials â€” the right way:**
   - Store in the credentials store, never in code
   - Use withCredentials in the Jenkinsfile
   - Rotate on a schedule
4. **The 30 plugins you need (the curated list):**
   - Pipeline, Blue Ocean, Git, Credentials, Job DSL
   - Kubernetes, Docker Pipeline, Docker
   - HashiCorp Vault, AWS Steps, Azure Credentials
   - Configuration as Code (JCasC), JobDSL
   - Warnings NG, Cobertura, JUnit
5. **Configuration as Code (JCasC) â€” `jenkins.yaml`:**
   ```yaml
   jenkins:
     numExecutors: 0
     mode: EXCLUSIVE
     securityRealm: ...
   ```
6. **Backup â€” the must-haves:**
   - `$JENKINS_HOME` (jobs, plugins, credentials, config)
   - `thinBackup` plugin for scheduled backups
7. **Hands-on exercise:** Configure JCasC, set up a Kubernetes agent, back up Jenkins.

#### Step 8: The 20 Patterns That Make Any Pipeline Better
1. **Cache dependencies** â€” pip, npm, go mod, maven
2. **Parallelize tests** â€” shard, matrix, fan-out
3. **Build once, promote many** â€” image tagged with `git sha`, deployed to each env
4. **Use ephemeral environments** â€” PR preview, per-branch
5. **Sign + attest every artifact** â€” cosign, SLSA provenance
6. **Scan everything** â€” SCA (Snyk, Trivy fs), container, code (CodeQL, Semgrep)
7. **Use OIDC to the cloud** â€” no long-lived keys
8. **Run untrusted code in isolation** â€” Docker, K8s, Firecracker
9. **Fail fast** â€” lint â†’ test â†’ build â†’ scan â†’ deploy
10. **Make manual gates explicit** â€” production deploys
11. **Auto-cancel outdated runs** â€” concurrency groups
12. **Limit secrets to jobs that need them** â€” environment-scoped
13. **Reuse workflows** â€” DRY
14. **Use `pull_request_target` carefully** â€” security risk
15. **Pin action versions by SHA** â€” not by tag
16. **Audit pipeline changes** â€” pipelines are code
17. **Version your runners** â€” Docker image, K8s pod spec
18. **Persist build logs** â€” ship to S3, GCS, Loki
19. **Measure DORA metrics** â€” deployment frequency, lead time, change fail rate, MTTR
20. **Practice rollbacks** â€” every deploy must have a tested rollback
21. **Hands-on exercise:** Pick 3 patterns you don't currently use; add them to your existing pipeline.

#### Step 9: DORA & The Engineering Metrics That Matter
1. **The four DORA metrics:**
   - **Deployment frequency** â€” how often
   - **Lead time for changes** â€” commit to prod
   - **Change failure rate** â€” % of deploys causing failures
   - **Mean time to recovery** â€” incident to fix
2. **Elite vs low performers:**
   | Metric | Elite | Low |
   |--------|-------|-----|
   | Deploy freq | multiple/day | monthly+ |
   | Lead time | < 1 day | 1-6 months |
   | Change fail rate | 0-15% | 46-60% |
   | MTTR | < 1 hour | 1 week+ |
3. **Measure with:**
   - GitHub Actions: Insights tab
   - GitLab: Value Stream Analytics
   - Jenkins: `metrics-plugin` + Prometheus
   - Cross-system: LinearB, Jellyfish, Sleuth, Waydev
4. **Hands-on exercise:** Calculate your team's current DORA metrics from the last 90 days of data.

#### Step 10: Production-Grade CI/CD Habits
- **Pipelines are code** â€” review changes via PR
- **Pin every action/version by SHA** in GitHub Actions
- **No long-lived secrets** â€” use OIDC, Vault, or short-lived tokens
- **Build once, promote many** â€” same image, different envs
- **Sign + scan every artifact** â€” cosign + Trivy on every build
- **Run untrusted code in isolated runners** â€” Docker, K8s, Firecracker
- **Test the pipeline** â€” `act` for GitHub Actions, GitLab CI Lint API
- **Monitor the runners** â€” Prometheus + alerting
- **Back up the controller** â€” JCasC + thinBackup (Jenkins) or IaC (Actions/GL)
- **Measure DORA** â€” the only way to know you're improving
- **Practice rollbacks** â€” every deploy must have a tested rollback

### Practice Project â€” Multi-Platform CI/CD Platform

> **Goal:** Build the same app's pipeline in all 3 systems (GitHub Actions, GitLab CI, Jenkins) and prove they produce identical signed artifacts. Add OIDC, env protection, matrix builds, and DORA tracking.
> **Time estimate:** 15-20 hours
> **Deliverables:** A repo with 3 working pipelines, signed images on 2 registries, and a DORA dashboard.

#### Phase 1 â€” App (1h)
- [ ] **(B)** Pick a small app (Node API) with /healthz, /readyz, /metrics
- [ ] **(B)** Add a Dockerfile (multi-stage, distroless) and a Helm chart
- [ ] **(I)** Add SCA, container, and code scanning configs

#### Phase 2 â€” GitHub Actions (3h)
- [ ] **(I)** Lint â†’ test (matrix 3 OS Ã— 3 Node) â†’ build â†’ scan â†’ sign â†’ push â€” Step 2
- [ ] **(I)** Use OIDC to push to ECR / GCR / ACR â€” Step 3
- [ ] **(I)** Add an `environment: production` with reviewers and wait timer
- [ ] **(I)** Use a reusable workflow for the build
- [ ] **(A)** Set up self-hosted runners via actions-runner-controller on K8s

#### Phase 3 â€” GitLab CI (3h)
- [ ] **(I)** Build â†’ test â†’ scan â†’ sign â†’ push with .gitlab-ci.yml â€” Step 4
- [ ] **(I)** Use `include` to DRY common jobs
- [ ] **(I)** Add protected/masked variables and a manual prod gate
- [ ] **(I)** Register a group runner and prove it picks up the job
- [ ] **(A)** Set up a Kubernetes executor and a parent-child pipeline

#### Phase 4 â€” Jenkins (3h)
- [ ] **(I)** Install Jenkins, set up the K8s plugin â€” Step 7
- [ ] **(I)** Build a Jenkinsfile with 5 stages
- [ ] **(I)** Use a shared library for the build step
- [ ] **(I)** Use `withCredentials` to pull from Vault
- [ ] **(A)** Use JCasC to declaratively configure Jenkins

#### Phase 5 â€” Multi-Cloud Signing (1.5h)
- [ ] **(I)** Sign with cosign in all 3 pipelines; prove same artifact
- [ ] **(I)** Verify the signature in the deploy stage
- [ ] **(A)** Add SLSA L3 provenance in GitHub Actions
- [ ] **(A)** Mirror the signed image to a second registry

#### Phase 6 â€” DORA (1.5h)
- [ ] **(I)** Enable DORA tracking in GitHub / GitLab
- [ ] **(I)** Export the metrics to a Grafana dashboard
- [ ] **(I)** Set targets: deploy freq = daily, lead time = < 1 day, CFR < 10%, MTTR < 1h
- [ ] **(A)** Add a Slack alert when a metric regresses

#### Phase 7 â€” Production-Grade Hardening (2h)
- [ ] **(I)** Pin every action by SHA in GitHub Actions
- [ ] **(I)** Use OIDC everywhere; remove long-lived keys
- [ ] **(I)** Scan every PR with CodeQL + Semgrep
- [ ] **(A)** Add a pipeline test (act for Actions) that catches a planted misconfig
- [ ] **(A)** Add SBOM attestation; store as release asset

#### Phase 8 â€” Documentation (1h)
- [ ] **(I)** Write a top-level `README.md` with how to onboard a new service
- [ ] **(I)** Write a `RUNBOOK.md` with: build failure, deploy failure, rollback, secret rotation
- [ ] **(A)** Document the DORA dashboard and targets
- [ ] **(A)** Write a "pipeline style guide" for the team

#### Bonus Challenges
- [ ] **(A)** Implement a canary deploy (Argo Rollouts or CodeDeploy) wired to a pipeline
- [ ] **(A)** Add progressive delivery (Flagger) with auto-rollback on SLO violation
- [ ] **(A)** Build a self-service pipeline generator that scaffolds a new service from a template

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 25 tasks across 8 phases + 3 bonus challenges**

### Key Takeaways
- All 3 systems implement the same loop: build, test, scan, package, deploy
- GitHub Actions = cloud-native, marketplace, YAML. GitLab CI = integrated, DAG. Jenkins = mature, plugin-rich, self-hosted
- The production bar is the same everywhere: OIDC, ephemeral runners, signed artifacts, matrix, cache
- DORA metrics tell you if your CI/CD is actually working
- Pipelines are code â€” review, version, test them

---

## â˜ï¸ Amazon Web Services (AWS)

**Difficulty:** Advanced | **Time:** 50-60 hours

### Overview
AWS is the largest cloud platform and the reference architecture for cloud-native. With 200+ services spanning compute (EC2, Lambda, ECS, EKS), storage (S3, EBS, FSx, Glacier), network (VPC, Transit Gateway, CloudFront, Global Accelerator), database (RDS, DynamoDB, Aurora, Redshift), security (IAM, KMS, Secrets Manager, WAF), and observability (CloudWatch, X-Ray, CloudTrail), mastering AWS is mastering the modern cloud. This tool covers every major service, the well-architected framework, multi-account design, and production patterns.

### Why Learn This
- ~32% market share â€” the industry default
- The services are the reference implementation that other clouds copy
- AWS certifications (SA Associate, SA Pro, Specialty) are the most-recognized in the industry
- The patterns (multi-AZ, multi-account, IAM least-privilege) apply everywhere
- Cost, security, and operational maturity in AWS is a deep specialty on its own

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model & The 5-Minute Tour
1. **The 5-minute tour of the console:**
   - **Regions:** isolated geographic areas (us-east-1, eu-west-1, ap-southeast-2)
   - **Availability Zones (AZs):** 2-6 per region, isolated data centers
   - **Edge locations:** CloudFront, Route 53, Global Accelerator
2. **The Well-Architected Framework â€” the 6 pillars:**
   | Pillar | Question |
   |--------|----------|
   | Operational Excellence | Are we running and monitoring well? |
   | Security | Are we protecting data, systems, and assets? |
   | Reliability | Can we recover and meet demand? |
   | Performance Efficiency | Are we using cloud resources efficiently? |
   | Cost Optimization | Are we delivering value at the right price? |
   | Sustainability | Are we minimizing environmental impact? |
3. **The 4 account types in a production org:**
   ```
   Root
   â”œâ”€â”€ OU: Security      (Log archive, Audit, tooling)
   â”œâ”€â”€ OU: Infrastructure (network, shared services)
   â”œâ”€â”€ OU: Workloads      (per-app or per-team)
   â””â”€â”€ OU: Sandbox        (dev experimentation)
   ```
4. **The control tower â€” your landing zone:**
   ```bash
   aws controltower list-landing-zone-operations
   aws sso login --profile myorg-admin
   ```
5. **Hands-on exercise:** Create an AWS account; enable IAM Identity Center; create 2 IAM users; set up MFA; explore the console.

#### Step 2: Identity & Access Management (IAM) â€” The Foundation
1. **The five IAM building blocks:**
   - **User:** a person or service
   - **Group:** a collection of users
   - **Role:** an identity assumable by anyone
   - **Policy:** a JSON document of permissions
   - **Permission boundary:** a max-permission guardrail
2. **The principle of least privilege â€” never `*:*`.**
3. **A least-privilege policy (not the default `AdministratorAccess`):**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "ListMyBuckets",
         "Effect": "Allow",
         "Action": ["s3:ListAllMyBuckets"],
         "Resource": "*"
       },
       {
         "Sid": "ReadWriteMyBucket",
         "Effect": "Allow",
         "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
         "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"]
       }
     ]
   }
   ```
4. **Roles â€” the AWS-native way to grant temporary credentials:**
   ```bash
   aws iam create-role --role-name EC2-S3-Read \
     --assume-role-policy-document file://trust.json
   aws iam attach-role-policy --role-name EC2-S3-Read \
     --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
   # Use it: an EC2 instance profile, or OIDC from K8s (IRSA), or CI
   ```
5. **IAM Identity Center (SSO) â€” the modern way:**
   - Federate to Okta, Azure AD, Google Workspace
   - Assign permission sets to users/groups across accounts
   - No long-lived keys â€” temporary credentials
6. **Access Analyzer â€” find unintended access:**
   ```bash
   aws accessanalyzer create-analyzer --analyzer-name myorg --type ACCOUNT
   aws accessanalyzer list-findings --analyzer-name myorg
   ```
7. **Hands-on exercise:** Create 3 users, 2 groups, 1 role; write a least-privilege policy for S3; test by assuming the role.

#### Step 3: The Network â€” VPC, Subnets, Route Tables, Gateways
1. **The 5 VPC primitives:**
   - **VPC:** the L3 boundary
   - **Subnet:** a slice of a VPC, in one AZ
   - **Route table:** determines where traffic goes
   - **Internet Gateway (IGW):** the public internet attachment
   - **NAT Gateway / Instance:** outbound-only for private subnets
2. **The production VPC design (already covered in the Networking tool):**
   ```
   VPC 10.0.0.0/16
   â”œâ”€â”€ public-a  10.0.1.0/24   â†’ IGW
   â”œâ”€â”€ public-b  10.0.2.0/24   â†’ IGW
   â”œâ”€â”€ private-a 10.0.10.0/24  â†’ NAT
   â”œâ”€â”€ private-b 10.0.20.0/24  â†’ NAT
   â”œâ”€â”€ data-a    10.0.100.0/24 â†’ no internet
   â””â”€â”€ data-b    10.0.200.0/24 â†’ no internet
   ```
3. **Security Groups vs NACLs:**
   - **SG:** stateful, instance-level, allow rules only
   - **NACL:** stateless, subnet-level, allow + deny rules
4. **VPC Peering vs Transit Gateway:**
   - **Peering:** 1-to-1, no transitive routing, low cost
   - **TGW:** hub-and-spoke, scales, route tables per attachment
5. **PrivateLink â€” the way to expose a service privately:**
   - No internet traversal, no route table changes, no NAT
6. **The 5 advanced constructs:**
   - **VPC Endpoints:** Gateway (S3, DynamoDB) and Interface (PrivateLink)
   - **VPC Flow Logs:** every packet accepted/rejected, to CloudWatch or S3
   - **VPC Lattice:** service-to-service auth, no mesh needed
   - **Reachability Analyzer:** the network debugger
   - **Network Firewall:** L7 inspection, IDS/IPS
7. **Hands-on exercise:** Build a 3-AZ VPC with public/private/data subnets, NAT, and flow logs; deploy a workload in private subnets; verify no public IP.

#### Step 4: Compute â€” EC2, ASG, Launch Templates
1. **EC2 â€” the foundational compute:**
   ```bash
   aws ec2 run-instances \
     --image-id ami-0abcdef1234567890 \
     --instance-type t3.medium \
     --key-name my-key \
     --security-group-ids sg-... \
     --subnet-id subnet-... \
     --user-data file://init.sh \
     --iam-instance-profile Name=my-role
   ```
2. **Instance types â€” the categories:**
   - General (M, T) â€” web, small DB
   - Compute (C) â€” CPU-bound
   - Memory (R, X) â€” caches, in-memory DB
   - Storage (I, D) â€” large local SSD
   - Accelerated (P, G, Inf) â€” ML, GPU
   - Arm (Graviton â€” 40% better price/perf) â€” when compatible
3. **Launch Templates â€” the immutable, versioned, instance config:**
   ```bash
   aws ec2 create-launch-template \
     --launch-template-name web \
     --version-description "1.0.0" \
     --launch-template-data file://lt.json
   ```
4. **Auto Scaling Groups â€” capacity that follows demand:**
   ```bash
   aws autoscaling create-auto-scaling-group \
     --auto-scaling-group-name web \
     --launch-template LaunchTemplateName=web,Version=1 \
     --min-size 3 --max-size 20 --desired-capacity 3 \
     --vpc-zone-identifier "subnet-a,subnet-b" \
     --target-group-arns arn:aws:elasticloadbalancing:... \
     --health-check-type ELB \
     --health-check-grace-period 300
   ```
5. **Scaling policies â€” the 3 kinds:**
   - **Target tracking:** keep CPU at 60% (recommended)
   - **Step:** add N when CPU > 80%
   - **Simple:** add 1 when CPU > 80%
6. **Spot + On-Demand + Savings Plans â€” the cost toolkit:**
   - **Spot:** up to 90% off, can be reclaimed
   - **On-Demand:** highest cost, no commitment
   - **Savings Plans:** 1-3 year commit, up to 72% off
   - **Reserved Instances:** legacy, similar to SP
7. **Hands-on exercise:** Build a launch template, an ASG with target tracking, deploy across 2 AZs, run a load test, watch it scale.

#### Step 5: Load Balancing & Auto Scaling
1. **The 3 load balancer types:**
   | Type | Layer | Use |
   |------|-------|-----|
   | ALB (Application) | L7 | HTTP/HTTPS, path/host routing, gRPC |
   | NLB (Network) | L4 | TCP/UDP, static IPs, ultra-low latency |
   | GLB (Gateway) | L3 | Third-party virtual appliances |
   | CLB (Classic) | legacy | don't use |
2. **An ALB with target group + health check:**
   ```bash
   aws elbv2 create-load-balancer --name web --type application --scheme internet-facing \
     --subnets subnet-a subnet-b --security-groups sg-alb
   aws elbv2 create-target-group --name web --protocol HTTP --port 8080 \
     --vpc-id vpc-... --health-check-path /healthz
   ```
3. **The ASG â†” ALB â†” WAF â†” ACM stack â€” the production web tier:**
   ```
   Route 53 â†’ CloudFront (WAF, ACM) â†’ ALB (ACM) â†’ ASG (EC2) â†’ RDS
   ```
4. **Hands-on exercise:** Deploy a web app behind an ALB, target group with health check, ASG that scales on CPU, Route 53 record.

#### Step 6: Storage â€” S3, EBS, EFS, FSx
1. **S3 â€” the most-used AWS service:**
   ```bash
   aws s3 mb s3://my-bucket
   aws s3 cp file.txt s3://my-bucket/
   aws s3 sync ./build s3://my-bucket/
   aws s3 presign s3://my-bucket/private.pdf --expires-in 3600
   ```
2. **S3 storage classes:**
   | Class | Use | $/GB |
   |-------|-----|------|
   | S3 Standard | Hot, frequent | $0.023 |
   | S3 Intelligent-Tiering | Auto | $0.023 + mgmt |
   | S3 Standard-IA | Infrequent, rapid | $0.0125 |
   | S3 One Zone-IA | Single AZ, infrequent | $0.01 |
   | S3 Glacier Instant | Archive, ms retrieval | $0.004 |
   | S3 Glacier Flexible | Archive, minutes-hours | $0.0036 |
   | S3 Glacier Deep | Archive, hours | $0.00099 |
3. **S3 features you'll use:**
   - **Versioning:** required for serious use
   - **Lifecycle rules:** auto-transition to cheaper classes
   - **Replication:** cross-region (CRR) or same-region (SRR)
   - **Event notifications:** to SQS, SNS, Lambda
   - **Object Lock:** for compliance
   - **Access points:** per-app per-bucket policies
   - **Multi-region access points:** global S3 namespace
4. **EBS â€” block storage for EC2:**
   - gp3, io2, st1, sc1
   - Snapshot to S3, copy cross-region, encrypt at rest
5. **EFS â€” NFS, shared, multi-AZ:**
   - Pay per GB, mount from many EC2s
6. **FSx â€” managed Windows, Lustre, NetApp, OpenZFS**
7. **Hands-on exercise:** Build an S3 bucket with versioning, lifecycle (Standard â†’ IA after 30d â†’ Glacier after 90d), replication, and a presigned URL.

#### Step 7: Database â€” RDS, Aurora, DynamoDB
1. **RDS â€” managed relational:**
   ```bash
   aws rds create-db-instance --db-instance-identifier app \
     --db-instance-class db.t3.medium --engine postgres --engine-version 16 \
     --master-username app --master-user-password "$DB_PASS" \
     --allocated-storage 100 --storage-type gp3 \
     --multi-az --storage-encrypted \
     --backup-retention-period 14 --deletion-protection \
     --vpc-security-group-ids sg-db --db-subnet-group-name app
   ```
2. **Aurora â€” the cloud-native relational:**
   - 5x faster than RDS MySQL, 3x faster than RDS Postgres
   - Storage auto-scales 10GB â†’ 128TB
   - Up to 15 read replicas, sub-10ms lag
   - Aurora Serverless v2 for variable workloads
   - Aurora Global Database for cross-region DR
3. **DynamoDB â€” the serverless NoSQL:**
   - Single-digit-ms latency at any scale
   - On-demand or provisioned
   - DAX for microsecond reads
   - Global tables for multi-region
   - Streams for change-data-capture
4. **The right database for the job:**
   | Need | Pick |
   |------|------|
   | Transactional, relational, ACID | RDS Postgres, Aurora |
   | Key-value, single-digit ms | DynamoDB |
   | In-memory, sub-ms | ElastiCache (Redis) |
   | Search | OpenSearch |
   | Graph | Neptune |
   | Time-series | Timestream |
   | Ledger | QLDB (or Hyperledger) |
5. **Hands-on exercise:** Deploy an RDS Postgres Multi-AZ, write from an EC2 in the same VPC, take a snapshot, restore it.

#### Step 8: Serverless â€” Lambda, API Gateway, Step Functions, EventBridge
1. **Lambda â€” functions as a service:**
   ```bash
   aws lambda create-function --function-name my-fn \
     --runtime python3.12 --role arn:aws:iam::123:role/lambda \
     --handler index.handler --zip-file fileb://fn.zip
   ```
2. **The right memory size + provisioned concurrency:**
   - Memory: 128 MB to 10,240 MB
   - Compute scales linearly with memory
   - Provisioned Concurrency: eliminate cold starts
3. **API Gateway â€” the front door:**
   - REST API: full-featured, $3.50/M
   - HTTP API: simple, $1.00/M (the modern default)
   - WebSocket: real-time
4. **Step Functions â€” the orchestrator:**
   ```json
   {
     "Comment": "Order workflow",
     "StartAt": "Validate",
     "States": {
       "Validate": { "Type": "Task", "Resource": "arn:aws:lambda:...:validate", "Next": "Process" },
       "Process":  { "Type": "Task", "Resource": "arn:aws:lambda:...:process",  "Next": "Notify" },
       "Notify":   { "Type": "Task", "Resource": "arn:aws:lambda:...:notify",   "End": true }
     }
   }
   ```
5. **EventBridge â€” the event bus:**
   - AWS service events â†’ Lambda
   - Scheduled rules (cron)
   - Cross-account event buses
6. **SAM / CDK / Terraform â€” the IaC for serverless.**
7. **Hands-on exercise:** Build a 3-step Step Function: S3 upload â†’ Lambda validates â†’ DynamoDB writes â†’ SNS notifies.

#### Step 9: Containers on AWS â€” ECS, EKS, Fargate
1. **ECS â€” the AWS-native orchestrator:**
   - Task = pod equivalent
   - Service = deployment equivalent
   - Cluster = pool of EC2 or Fargate
2. **Fargate â€” serverless containers:**
   - No nodes, no kubelet
   - Pay per vCPU + GB-second
   - The right choice for most new workloads
3. **EKS â€” managed Kubernetes:**
   - Control plane managed by AWS
   - Worker nodes: managed node groups, self-managed, Fargate profiles, Karpenter
   - IRSA (IAM Roles for Service Accounts) â€” the OIDC integration
4. **The choice matrix:**
   | Need | Pick |
   |------|------|
   | Tight AWS integration, simple | ECS Fargate |
   | Portability, K8s ecosystem | EKS |
   | ML training, GPU-heavy | EKS on EC2 with GPUs |
   | Batch jobs | AWS Batch on ECS or EKS |
5. **ECR â€” the registry:**
   ```bash
   aws ecr create-repository --repository-name myapp
   aws ecr get-login-password | docker login --username AWS --password-stdin 123.dkr.ecr.us-east-1.amazonaws.com
   docker push 123.dkr.ecr.us-east-1.amazonaws.com/myapp:1.2.3
   ```
6. **Hands-on exercise:** Deploy a containerized app on ECS Fargate behind an ALB, with auto-scaling and a CI pipeline that pushes to ECR.

#### Step 10: Security & Identity Services
1. **KMS â€” the encryption backbone:**
   - Per-region, per-account keys
   - Customer-managed CMKs, AWS-managed, AWS-owned
   - Envelope encryption for large data
2. **Secrets Manager â€” secrets with rotation:**
   ```bash
   aws secretsmanager create-secret --name prod/db --secret-string "$(cat db.json)"
   aws secretsmanager rotate-secret --secret-id prod/db --rotation-lambda-arn ...
   ```
3. **WAF â€” the L7 firewall:**
   - Managed rule sets (AWS, marketplace)
   - Rate limiting, geo, IP lists, SQLi/XSS rules
4. **Shield â€” the DDoS protection:**
   - Standard: free, L3/L4
   - Advanced: paid, L7, with DRT
5. **GuardDuty â€” the threat detector:**
   - ML-based, looks at CloudTrail, VPC Flow Logs, DNS logs
6. **Inspector â€” the vulnerability scanner:**
   - For EC2, ECR images, Lambda
7. **Security Hub â€” the central security posture.**
8. **Macie â€” the data privacy scanner (S3).**
9. **Hands-on exercise:** Encrypt an S3 bucket with a CMK, store a DB password in Secrets Manager, attach a WAF to the ALB, enable GuardDuty.

#### Step 11: Observability â€” CloudWatch, X-Ray, CloudTrail
1. **CloudWatch â€” the metrics + logs + alarms platform:**
   ```bash
   aws cloudwatch put-metric-alarm --alarm-name high-cpu \
     --metric-name CPUUtilization --namespace AWS/EC2 \
     --statistic Average --period 60 --threshold 70 \
     --comparison-operator GreaterThanThreshold --evaluation-periods 3 \
     --alarm-actions arn:aws:sns:us-east-1:123:alerts
   ```
2. **Logs Insights â€” query your logs:**
   ```
   fields @timestamp, @message
   | filter @message like /ERROR/
   | stats count() by bin(5m)
   ```
3. **X-Ray â€” distributed tracing:**
   - Auto-instrumented SDKs
   - Service map
   - Latency, error, fault analysis
4. **CloudTrail â€” the audit log:**
   - Every API call, in every account
   - Store in S3, query with Athena
5. **The observability bar:**
   - **Metrics:** golden signals (latency, traffic, errors, saturation)
   - **Logs:** structured JSON, indexed
   - **Traces:** for any request > 100ms
6. **Hands-on exercise:** Set up CloudWatch alarms on ASG CPU, ALB 5xx, RDS connections; send a custom metric from a Lambda.

#### Step 12: Cost & Governance
1. **Cost Explorer â€” visualize spend:**
   ```bash
   aws ce get-cost-and-usage --time-period Start=2026-01-01,End=2026-01-31 \
     --granularity MONTHLY --metrics BlendedCost
   ```
2. **Budgets â€” alert when spending exceeds target:**
   ```bash
   aws budgets create-budget --account-id 123 --budget file://budget.json \
     --notifications-with-subscribers file://notify.json
   ```
3. **Cost Allocation Tags â€” the only way to know who spends what.**
4. **Organizations + SCPs â€” guardrails across accounts:**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       { "Sid": "DenyUnusedRegions", "Effect": "Deny",
         "Action": "*", "Resource": "*",
         "Condition": { "StringNotEquals": { "aws:RequestedRegion": ["us-east-1", "eu-west-1"] } } }
     ]
   }
   ```
5. **Trusted Advisor â€” the AWS-native recommendation engine.**
6. **Hands-on exercise:** Tag every resource in your account; create a budget alert; apply an SCP that denies services you don't use.

#### Step 13: Multi-Account & Networking at Scale
1. **The hub-and-spoke network:**
   ```
   TGW in network account
   â”œâ”€â”€ VPC-Workload-1 (account 111)
   â”œâ”€â”€ VPC-Workload-2 (account 222)
   â””â”€â”€ VPC-Shared-Services (account 333)
   ```
2. **Transit Gateway + RAM â€” share across accounts:**
   ```bash
   aws ram create-resource-share --name tgw-share --resource-arns tgw-arn \
     --principals "123" "456"
   ```
3. **Direct Connect â€” the dedicated line:**
   - 1, 10, 100 Gbps
   - To a Direct Connect location
4. **Site-to-Site VPN â€” the encrypted alternative.**
5. **Cloud WAN â€” the managed global network (2026+).**
6. **Hands-on exercise:** Create 2 VPCs in 2 accounts, share a TGW, route between them.

#### Step 14: Production-Grade AWS Habits
- **Multi-account from day one** â€” even small orgs benefit
- **IAM Identity Center + permission sets** â€” no long-lived keys
- **MFA on the root account**, locked away
- **SCPs for guardrails** â€” region lock, service allowlist
- **Encrypt everything** â€” at rest + in transit, with CMKs
- **Tag everything** â€” for cost, audit, automation
- **CloudTrail in every account, in one log-archive account**
- **Back up with AWS Backup** â€” centralized policy
- **Cost + usage alarms** before you hit the invoice
- **DR with Pilot Light / Warm Standby / Multi-Region Active-Active**
- **Practice incident response** â€” Game Days quarterly

### Practice Project â€” Production-Grade 3-Tier on AWS

> **Goal:** Build a production-shape AWS environment â€” multi-AZ, multi-account, full observability, IaC, CI/CD, DR.
> **Time estimate:** 30-40 hours
> **Deliverables:** A Terraform-managed infra, a deployed app, a working CI/CD, and a full runbook.

#### Phase 1 â€” Org & Identity (2h)
- [ ] **(I)** Create an AWS Organization; add 3 accounts: security, network, prod
- [ ] **(I)** Set up IAM Identity Center with 3 users
- [ ] **(I)** Apply SCPs: deny unused regions, deny root actions
- [ ] **(I)** Enable CloudTrail in all accounts; aggregate to log-archive

#### Phase 2 â€” Network (3h)
- [ ] **(I)** Build a 3-AZ VPC in the network account via Terraform
- [ ] **(I)** Add a TGW; share via RAM to the prod account
- [ ] **(I)** Add VPC flow logs to CloudWatch + S3
- [ ] **(A)** Add VPC Lattice for service-to-service

#### Phase 3 â€” Compute & ALB (3h)
- [ ] **(I)** Build a launch template + ASG in private subnets
- [ ] **(I)** Add an ALB with ACM cert and WAF
- [ ] **(I)** Add target tracking scaling on CPU
- [ ] **(A)** Add a Graviton-based mixed instances policy

#### Phase 4 â€” Database (2h)
- [ ] **(I)** Deploy Aurora Postgres Serverless v2 (or RDS Multi-AZ)
- [ ] **(I)** Store creds in Secrets Manager; reference from app
- [ ] **(I)** Enable RDS Proxy for connection pooling
- [ ] **(A)** Add a read replica in a second region

#### Phase 5 â€” Storage & CDN (1.5h)
- [ ] **(I)** S3 bucket with versioning, encryption, lifecycle, public access block
- [ ] **(I)** CloudFront in front of the S3 static site
- [ ] **(A)** Add cross-region replication for the logs bucket

#### Phase 6 â€” Observability (2.5h)
- [ ] **(I)** CloudWatch alarms: ALB 5xx, ASG CPU, RDS CPU, S3 4xx
- [ ] **(I)** SNS topic for alerts; route to email or Slack
- [ ] **(I)** CloudWatch Logs Insights queries for app logs
- [ ] **(I)** X-Ray tracing in the app
- [ ] **(A)** Add a CloudWatch dashboard with golden signals

#### Phase 7 â€” Serverless Add-On (2h)
- [ ] **(I)** Build a Lambda + API Gateway for a `/webhook` endpoint
- [ ] **(I)** Step Function: S3 â†’ Lambda â†’ DynamoDB
- [ ] **(A)** Schedule a Lambda with EventBridge to run every 5 min
- [ ] **(A)** Add provisioned concurrency to eliminate cold starts

#### Phase 8 â€” Security (2h)
- [ ] **(I)** Enable GuardDuty in all accounts
- [ ] **(I)** Enable Security Hub; subscribe to standards
- [ ] **(I)** WAF: rate limit + AWS managed rules
- [ ] **(I)** Inspector: scan all ECR images
- [ ] **(A)** Add IAM Access Analyzer findings to the security dashboard

#### Phase 9 â€” CI/CD (3h)
- [ ] **(I)** GitHub Actions with OIDC â†’ AWS
- [ ] **(I)** Build â†’ scan â†’ sign â†’ push to ECR
- [ ] **(I)** CodeDeploy blue/green deploy to the ASG
- [ ] **(A)** Add CodePipeline for end-to-end

#### Phase 10 â€” DR & Operations (2h)
- [ ] **(I)** Cross-region read replica + Route 53 failover
- [ ] **(I)** AWS Backup: nightly snapshots of EBS, RDS, EFS
- [ ] **(I)** Practice a region failover: flip Route 53, verify app
- [ ] **(A)** Add a Game Day runbook: simulate RDS failure, AZ failure, region failure

#### Phase 11 â€” Cost (1.5h)
- [ ] **(I)** Tag every resource with env, team, app
- [ ] **(I)** Cost Explorer report per team
- [ ] **(I)** Budget alarm at 80% of monthly target
- [ ] **(A)** Compute Savings Plan recommendation; commit

#### Phase 12 â€” Documentation (1.5h)
- [ ] **(I)** Architecture diagram in `architecture.drawio`
- [ ] **(I)** `RUNBOOK.md`: deploy, scale, fail, restore
- [ ] **(I)** `SECURITY.md`: IAM, SCPs, GuardDuty, WAF
- [ ] **(A)** `COST.md`: per-team, per-app, optimization plan

#### Bonus Challenges
- [ ] **(A)** Build a multi-region active-active app with Aurora Global + Route 53 latency-based
- [ ] **(A)** Implement service-to-service auth with IAM Sigv4 (no IAM user keys)
- [ ] **(A)** Build a self-service platform using Service Catalog + Lambda

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 30 tasks across 12 phases + 3 bonus challenges**

### Key Takeaways
- AWS is 200+ services, but the foundational 25 cover 95% of production
- Multi-account + Organizations + SCPs is the only way to scale a cloud org
- IAM is the foundation â€” least privilege, roles everywhere, no long-lived keys
- Multi-AZ by default, multi-region for HA, Pilot Light for cost-sensitive DR
- Cost + security + observability are first-class concerns, not afterthoughts

---

## ðŸ”· Microsoft Azure

**Difficulty:** Advanced | **Time:** 45-55 hours

### Overview
Azure is Microsoft's cloud and the second-largest platform worldwide. It is the natural choice for enterprises standardized on Microsoft, the only major cloud with strong Active Directory integration, and the home of services like Azure AD (now Entra ID), Azure SQL, Azure Functions, and Azure Kubernetes Service. This tool covers every major Azure service, the Azure Resource Manager model, Entra ID, networking, compute, data, serverless, observability, and enterprise-grade patterns.

### Why Learn This
- ~25% market share and growing in the enterprise
- Entra ID is the de facto identity provider for the Microsoft ecosystem
- Azure Arc extends Azure management to on-prem and other clouds
- Hybrid (Azure Stack) is unique among hyperscalers
- The certifications (AZ-104, AZ-305, AZ-400) are highly valued in enterprise IT

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The hierarchy â€” Azure's nested scopes:**
   ```
   Tenant (Entra ID)
   â””â”€â”€ Management Group
       â””â”€â”€ Subscription
           â””â”€â”€ Resource Group
               â””â”€â”€ Resources
   ```
2. **The Azure portal, CLI, and PowerShell â€” pick your weapon:**
   ```bash
   # Azure CLI
   az login
   az account list
   az account set --subscription "Production"
   az group create --name rg-prod --location eastus

   # PowerShell
   Connect-AzAccount
   Get-AzSubscription
   Set-AzContext -Subscription "Production"
   New-AzResourceGroup -Name rg-prod -Location eastus
   ```
3. **The 4 services to know on day one:**
   - **Entra ID** (Azure AD): identity
   - **Azure Resource Manager (ARM):** the API for everything
   - **Azure Monitor:** metrics + logs + alerts
   - **Cost Management + Billing:** the cost toolkit
4. **Regions + Availability Zones:**
   - 60+ regions, 3+ AZs each (most regions)
   - Region pairs (East US â†” West US) for DR
5. **The Azure Architecture Framework â€” the 5 pillars:**
   | Pillar | Focus |
   |--------|-------|
   | Cost Optimization | Spend efficiency |
   | Operational Excellence | DevOps, monitoring, automation |
   | Performance Efficiency | Scale, capacity |
   | Reliability | Resiliency, availability, DR |
   | Security | Confidentiality, integrity |
6. **Hands-on exercise:** Create a free account, install Azure CLI, log in, create a resource group, deploy a "hello world" via the CLI.

#### Step 2: Entra ID (Azure AD) â€” The Identity Backbone
1. **The 5 building blocks:**
   - **User:** a person
   - **Group:** a collection of users/devices
   - **Application:** an identity for an app
   - **Service Principal:** an identity for a service
   - **Managed Identity:** an auto-managed SP for Azure resources
2. **The RBAC model:**
   - Built-in roles: Owner, Contributor, Reader, User Access Admin
   - Custom roles: defined by `actions` and `dataActions`
   - Assignments: user/group/SP â†’ role â†’ scope (sub, RG, resource)
3. **A role assignment at the resource group:**
   ```bash
   az role assignment create \
     --assignee alice@myorg.com \
     --role "Contributor" \
     --resource-group rg-prod
   ```
4. **Managed Identity â€” the best practice:**
   ```bash
   # System-assigned
   az vm identity assign --name myvm --resource-group rg-prod
   # User-assigned
   az identity create --name myid --resource-group rg-prod
   az vm identity assign --name myvm --identities myid
   ```
5. **Conditional Access â€” the security layer:**
   - Require MFA, compliant device, specific location
   - Block legacy auth
   - Define in Entra ID â†’ Security â†’ Conditional Access
6. **App registrations â€” the OAuth/OIDC surface:**
   ```bash
   az ad app create --display-name myapp --web-redirect-uris https://app.example.com/callback
   az ad sp create --id <app-id>
   ```
7. **Hands-on exercise:** Create a user, a group, a managed identity on a VM; grant the MI access to a storage account; verify the VM can read it.

#### Step 3: The Network â€” VNet, Subnets, NSG, Route Tables
1. **The primitives:**
   - **VNet:** the L3 boundary
   - **Subnet:** a slice of a VNet, in an AZ
   - **NSG (Network Security Group):** stateful L4 rules
   - **Route Table:** user-defined routes
   - **Public IP:** static or dynamic
2. **The production design â€” same as AWS, different names:**
   ```
   VNet 10.0.0.0/16
   â”œâ”€â”€ public-a  10.0.1.0/24   â†’ UDR to IGW
   â”œâ”€â”€ public-b  10.0.2.0/24
   â”œâ”€â”€ private-a 10.0.10.0/24  â†’ UDR to NAT
   â”œâ”€â”€ private-b 10.0.20.0/24
   â”œâ”€â”€ data-a    10.0.100.0/24 â†’ no internet
   â””â”€â”€ data-b    10.0.200.0/24
   ```
3. **A network security group with rules:**
   ```bash
   az network nsg create -g rg-prod -n nsg-web
   az network nsg rule create -g rg-prod --nsg-name nsg-web \
     -n allow-https --priority 100 \
     --source-address-prefixes Internet --destination-port-ranges 443 \
     --access Allow --protocol Tcp --direction Inbound
   az network nsg rule create -g rg-prod --nsg-name nsg-web \
     -n deny-all-in --priority 4096 --access Deny --direction Inbound
   ```
4. **Azure Firewall vs NSG vs WAF:**
   - **NSG:** L4, basic
   - **Azure Firewall:** L7, threat intel, FQDN tags
   - **WAF:** L7 for App Gateway / Front Door
5. **VNet Peering vs VPN Gateway vs ExpressRoute:**
   - **Peering:** 1-to-1, low latency
   - **VPN Gateway:** encrypted over internet
   - **ExpressRoute:** dedicated private line
6. **Private Link â€” the Azure way to expose a service privately.**
7. **Hands-on exercise:** Build a 3-AZ VNet, NSGs, NAT Gateway, Azure Firewall, peering to a second VNet.

#### Step 4: Compute â€” VMs, VMSS, and the Bastion
1. **Create a VM:**
   ```bash
   az vm create -g rg-prod -n web \
     --image Ubuntu2204 --size Standard_D2s_v5 \
     --admin-username azureuser --generate-ssh-keys \
     --vnet-name vnet --subnet private-a \
     --public-ip-address "" \
     --nsg nsg-web \
     --assign-identity
   ```
2. **Azure Bastion â€” the secure jump host:**
   ```bash
   az network bastion create -g rg-prod -n bastion \
     --public-ip-address pip-bastion --vnet-name vnet
   # RDP/SSH via the portal, no public IP on the VM
   ```
3. **Virtual Machine Scale Sets (VMSS) â€” Azure's ASG:**
   ```bash
   az vmss create -g rg-prod -n web \
     --image Ubuntu2204 --vm-sku Standard_D2s_v5 \
     --instance-count 3 \
     --vnet vnet --subnet private-a \
     --upgrade-policy-mode automatic \
     --automatic-repairs true
   ```
4. **Autoscale rules:**
   ```bash
   az monitor autoscale create -g rg-prod --resource web \
     --resource-type Microsoft.Compute/virtualMachineScaleSets \
     --min-count 3 --max-count 20
   az monitor autoscale rule create -g rg-prod --autoscale-name web \
     --condition "CpuPercentage > 60 avg 5m" \
     --scale out 1
   ```
5. **Spot + Reserved + Savings Plan â€” the cost toolkit:**
   - **Spot VMs:** up to 90% off
   - **Reserved Instances:** 1-3 year commit, up to 72% off
   - **Savings Plan:** flexible, hourly commit
6. **Hands-on exercise:** Create a VMSS in a private subnet, autoscale on CPU, access via Bastion.

#### Step 5: Load Balancing â€” App Gateway, Load Balancer, Front Door, Traffic Manager
1. **The 4 load balancers:**
   | Service | Layer | Use |
   |---------|-------|-----|
   | Azure Load Balancer | L4 | TCP/UDP, internal or public |
   | Application Gateway | L7 | HTTP, WAF, path routing |
   | Front Door | L7 global | Multi-region, CDN, WAF |
   | Traffic Manager | DNS | Global, failover, geo |
2. **An App Gateway with WAF:**
   ```bash
   az network application-gateway create -g rg-prod -n appgw \
     --sku WAF_v2 --capacity 2 --vnet-name vnet --subnet public-a \
     --frontend-port 443 --backend-pool-name web --http-settings-port 8080 \
     --cert-file /path/cert.pfx --cert-password "$CERT_PASS"
   ```
3. **Front Door â€” the global L7:**
   ```bash
   az afd profile create -g rg-prod --profile-name fd --sku Premium_AzureFrontDoor
   az afd origin create -g rg-prod --profile-name fd \
     --origin-group og --origin-name web1 --host-name app1.example.com
   az afd route create -g rg-prod --profile-name fd \
     --endpoint-name ep --origin-group og --route-name r \
     --patterns-to-match "/*" --https-redirect Enabled
   ```
4. **Hands-on exercise:** Put a VMSS behind App Gateway with WAF; put a multi-region deploy behind Front Door.

#### Step 6: Storage â€” Blob, File, Queue, Table
1. **The 4 storage types:**
   - **Blob:** object storage (the S3 equivalent)
   - **File:** SMB/NFS file shares
   - **Queue:** messaging
   - **Table:** NoSQL key-value (now superseded by Cosmos DB)
2. **A storage account with 4 layers of security:**
   ```bash
   az storage account create -g rg-prod -n myorgstore \
     --sku Standard_ZRS --kind StorageV2 \
     --https-only true --min-tls-version TLS1_2 \
     --allow-blob-public-access false \
     --bypass AzureServices
   ```
3. **Blob tiers:**
   | Tier | Use |
   |------|-----|
   | Hot | Frequent access |
   | Cool | Infrequent, 30+ days |
   | Cold | 90+ days, faster access than Archive |
   | Archive | 180+ days, hours to rehydrate |
4. **Lifecycle management:**
   ```json
   {
     "rules": [{
       "name": "archive-old",
       "enabled": true,
       "definition": {
         "actions": {
           "baseBlob": {
             "tierToCool":    { "daysAfterModificationGreaterThan": 30 },
             "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
             "delete":        { "daysAfterModificationGreaterThan": 365 }
           }
         }
       }
     }]
   }
   ```
5. **Hands-on exercise:** Create a storage account, upload a blob, set lifecycle, generate a SAS, and access from a VM with managed identity.

#### Step 7: Database â€” Azure SQL, Cosmos DB, PostgreSQL, MySQL
1. **Azure SQL â€” managed SQL Server:**
   ```bash
   az sql server create -g rg-prod -n sql-prod \
     --admin-user sqladmin --admin-password "$SQL_PASS"
   az sql db create -g rg-prod -s sql-prod -n appdb \
     --service-objective S2 --backup-storage-redundancy Zone
   ```
2. **PostgreSQL Flexible Server â€” the modern managed Postgres:**
   ```bash
   az postgres flexible-server create -g rg-prod -n pg-prod \
     --admin-user pgadmin --admin-password "$PG_PASS" \
     --tier Burstable --sku-name Standard_B1ms \
     --storage-size 32 --high-availability ZoneRedundant
   ```
3. **Cosmos DB â€” the globally distributed NoSQL:**
   - Multi-region writes
   - 5 consistency levels
   - SQL API, MongoDB API, Cassandra API, Gremlin API
4. **The right database for the job:**
   | Need | Pick |
   |------|------|
   | SQL Server compat | Azure SQL |
   | Postgres, full open-source | Azure Postgres Flexible |
   | Globally distributed, low latency | Cosmos DB |
   | Caching | Redis Cache |
   | Search | Cognitive Search |
5. **Hands-on exercise:** Deploy Azure Postgres Flexible Server, connect from a VM in the same VNet, write a row.

#### Step 8: Serverless â€” Azure Functions, Logic Apps, Event Grid
1. **Azure Functions â€” the FaaS:**
   ```bash
   az functionapp create -g rg-prod -n myfunc \
     --runtime python --runtime-version 3.12 \
     --storage-account mystore --plan myplan
   ```
2. **The 4 hosting plans:**
   | Plan | Use |
   |------|-----|
   | Consumption | Pay per execution, scale to 0 |
   | Premium | Pre-warmed, no cold start, VNet |
   | Dedicated (App Service) | Predictable cost, long-running |
   | Flex Consumption | Newer, VNet, scale to 0 |
3. **Event Grid â€” the event router:**
   ```bash
   az eventgrid topic create -g rg-prod -n mytopic
   az eventgrid event-subscription create --name mysub \
     --source-resource-id /subscriptions/.../mytopic \
     --endpoint /subscriptions/.../myfunc
   ```
4. **Logic Apps â€” the integration workhorse:**
   - 400+ connectors
   - Low-code workflows
5. **Hands-on exercise:** Build a Function triggered by Event Grid on blob upload, writes to Cosmos DB.

#### Step 9: Containers â€” AKS, Container Apps, Container Instances
1. **AKS â€” the managed Kubernetes:**
   ```bash
   az aks create -g rg-prod -n aks-prod \
     --node-count 3 --enable-managed-identity \
     --network-plugin azure --network-plugin-mode overlay \
     --enable-oidc-issuer --enable-workload-identity \
     --enable-azure-rbac
   az aks get-credentials -g rg-prod -n aks-prod
   ```
2. **Container Apps â€” the serverless containers:**
   ```bash
   az containerapp create -g rg-prod -n myapp \
     --image ghcr.io/myorg/myapp:1.2.3 \
     --ingress external --target-port 8080 \
     --min-replicas 1 --max-replicas 10 \
     --env-vars LOG_LEVEL=info
   ```
3. **Container Instances â€” burst jobs, simple.**
4. **The choice matrix:**
   | Need | Pick |
   |------|------|
   | Full K8s, portability | AKS |
   | Microservices, KEDA, Dapr | Container Apps |
   | Burst, batch, simple | Container Instances |
   | Existing K8s skills | AKS |
5. **Azure Container Registry (ACR):**
   ```bash
   az acr create -g rg-prod -n myorg --sku Premium
   az acr login -n myorg
   docker push myorg.azurecr.io/myapp:1.2.3
   ```
6. **Hands-on exercise:** Deploy a Container App from ACR with auto-scaling, KEDA on a queue, ingress.

#### Step 10: Security Services
1. **Microsoft Defender for Cloud â€” the CSPM/CWPP:**
   - Recommendations, secure score
   - Defender plans: Servers, App Service, SQL, Storage, K8s, etc.
2. **Key Vault â€” secrets, keys, certs:**
   ```bash
   az keyvault create -g rg-prod -n myorg-kv \
     --enable-rbac-authorization --enable-purge-protection
   az keyvault secret set --vault-name myorg-kv --name db-pass --value "$DB_PASS"
   ```
3. **Sentinel â€” the cloud SIEM.**
4. **DDoS Protection â€” Standard for the public IP.**
5. **WAF â€” on App Gateway, Front Door, or CDN.**
6. **Hands-on exercise:** Enable Defender for Cloud, store a secret in Key Vault, retrieve from a VM with managed identity.

#### Step 11: Observability â€” Azure Monitor, Log Analytics, App Insights
1. **Azure Monitor â€” the central telemetry:**
   - **Metrics:** time-series, alerting
   - **Logs:** KQL queries in Log Analytics
   - **Traces:** App Insights
2. **A KQL query â€” the lingua franca:**
   ```kusto
   AppRequests
   | where TimeGenerated > ago(1h)
   | summarize count() by bin(TimeGenerated, 5m), ResultCode
   | render timechart
   ```
3. **App Insights â€” APM for any app:**
   ```bash
   az monitor app-insights component create -g rg-prod --app myapp \
     --application-type web --workspace-based
   ```
4. **Workbooks â€” the dashboard builder.**
5. **Alerts + Action Groups:**
   ```bash
   az monitor action-group create -g rg-prod -n alerts \
     --email-receiver name=ops email=ops@myorg.com
   az monitor metrics alert create -g rg-prod -n cpu-high \
     --scopes /subscriptions/.../vm \
     --condition "avg Percentage CPU > 80" \
     --window-size 5m --evaluation-frequency 1m \
     --action-groups alerts
   ```
6. **Hands-on exercise:** Wire App Insights to a web app; build a workbook with 4 panels; set an alert on response time.

#### Step 12: Cost & Governance
1. **Cost Management + Billing â€” budgets, alerts, exports:**
   ```bash
   az consumption budget create -g rg-prod -n monthly \
     --budget-amount 1000 --time-grain Monthly --start-date 2026-01-01
   ```
2. **Policy â€” the guardrails:**
   ```bash
   az policy assignment create -g rg-prod \
     --policy "Allowed resource types" \
     --params '{"listOfResourceTypesAllowed": {"value": ["Microsoft.Compute", "Microsoft.Storage"]}}'
   ```
3. **Blueprints â€” a packaged set of resources, policy, RBAC.**
4. **Locks â€” prevent delete or change at any scope.**
5. **Tags + Tag policies â€” for cost allocation.**
6. **Hands-on exercise:** Apply a policy that denies creation of resources without an `Env` tag; set a budget with an email alert.

#### Step 13: Multi-Region, DR, and the Enterprise Patterns
1. **Region pairs â€” Azure's automatic DR pairing.**
2. **Traffic Manager + Front Door â€” global L7 routing.**
3. **Geo-redundant storage (GRS) â€” 6 nines durability, async replication.**
4. **Azure Site Recovery â€” DR for VMs.**
5. **Backup Vault â€” centralized backup for VMs, SQL, Files, Blobs, Postgres.**
6. **Azure Arc â€” extend Azure to on-prem and other clouds:**
   ```bash
   az connectedk8s connect -g rg-arc -n on-prem --location eastus
   ```
7. **Hands-on exercise:** Enable GRS on a storage account, set up a Backup Vault, back up a VM.

#### Step 14: Production-Grade Azure Habits
- **Management groups + subscription strategy** â€” by env, by team
- **Entra ID + Managed Identities** â€” no long-lived secrets
- **Policy + Blueprints** â€” guardrails from day one
- **Private Endpoints** â€” not public endpoints
- **Azure Monitor + Workbooks** â€” observability by default
- **Cost alerts + budgets** â€” before the invoice arrives
- **Tags everywhere** â€” for cost + ownership
- **DR with paired regions** â€” geo-redundant storage, Site Recovery
- **DevOps with Azure DevOps or GitHub Actions** â€” pipelines as code
- **Practice incident response** â€” Game Days

### Practice Project â€” Production-Grade 3-Tier on Azure

> **Goal:** Build a production-shape Azure environment â€” multi-region, full observability, IaC, CI/CD, DR.
> **Time estimate:** 25-30 hours
> **Deliverables:** A Bicep/Terraform-managed infra, a deployed app, a working CI/CD, and a full runbook.

#### Phase 1 â€” Org & Identity (2h)
- [ ] **(I)** Create a Tenant, set up Management Groups (Sandbox, Dev, Prod)
- [ ] **(I)** Create 3 subscriptions; assign under MGs
- [ ] **(I)** Set up Entra ID with 3 users, 2 groups
- [ ] **(I)** Custom RBAC role: "DevOps Engineer" with limited Contributor at RG

#### Phase 2 â€” Network (2.5h)
- [ ] **(I)** Build a 3-AZ VNet via Bicep/Terraform
- [ ] **(I)** Add NSGs, Route Tables, Azure Firewall, NAT Gateway
- [ ] **(I)** Peer to a second VNet in another region
- [ ] **(A)** Add Private Endpoints for Storage, Key Vault, SQL

#### Phase 3 â€” Compute & App Gateway (2.5h)
- [ ] **(I)** Create a VMSS in private subnets
- [ ] **(I)** Add App Gateway with WAF + cert from Key Vault
- [ ] **(I)** Autoscale on CPU + LB requests
- [ ] **(A)** Add Spot instances for non-prod

#### Phase 4 â€” Database (2h)
- [ ] **(I)** Deploy Azure Postgres Flexible Server with HA
- [ ] **(I)** Store creds in Key Vault; reference from app
- [ ] **(I)** Enable Azure Backup for the DB
- [ ] **(A)** Add a read replica in a second region

#### Phase 5 â€” Storage (1.5h)
- [ ] **(I)** Storage account with GRS, lifecycle, private endpoint
- [ ] **(I)** Azure CDN (or Front Door) in front
- [ ] **(A)** Add immutable blob storage for compliance

#### Phase 6 â€” Containers (1.5h)
- [ ] **(I)** Deploy AKS with workload identity, OIDC, Azure RBAC
- [ ] **(I)** Deploy an app from ACR with KEDA scaling
- [ ] **(A)** Add Azure CNI Overlay for IPAM scale

#### Phase 7 â€” Serverless (1.5h)
- [ ] **(I)** Azure Function: HTTP trigger, write to Cosmos
- [ ] **(I)** Event Grid: blob upload â†’ Function
- [ ] **(A)** Premium plan with VNet integration

#### Phase 8 â€” Security (2h)
- [ ] **(I)** Enable Defender for Cloud + all plans
- [ ] **(I)** WAF on App Gateway + Front Door
- [ ] **(I)** Store all secrets in Key Vault with private endpoint
- [ ] **(A)** Add Microsoft Sentinel (or a third-party SIEM)

#### Phase 9 â€” Observability (1.5h)
- [ ] **(I)** Wire App Insights; build a workbook
- [ ] **(I)** Alerts: VMSS CPU, App Gateway 5xx, DB CPU
- [ ] **(A)** Distributed tracing with App Insights across services

#### Phase 10 â€” CI/CD (2h)
- [ ] **(I)** GitHub Actions with OIDC â†’ Azure
- [ ] **(I)** Build â†’ scan â†’ sign â†’ push to ACR
- [ ] **(I)** Deploy via Azure DevOps or GH Actions
- [ ] **(A)** Add blue/green with App Gateway slots

#### Phase 11 â€” DR & Cost (1.5h)
- [ ] **(I)** Backup Vault: nightly snapshots
- [ ] **(I)** Cost budget + email alert
- [ ] **(I)** Policy: deny resources without `Env` tag
- [ ] **(A)** Multi-region active-active with Traffic Manager + Cosmos

#### Phase 12 â€” Documentation (1h)
- [ ] **(I)** Architecture diagram
- [ ] **(I)** `RUNBOOK.md`: deploy, scale, fail, restore
- [ ] **(I)** `SECURITY.md`: Entra ID, Key Vault, Defender
- [ ] **(A)** `COST.md`: per-team, per-app

#### Bonus Challenges
- [ ] **(A)** Implement Azure Arc for on-prem K8s
- [ ] **(A)** Build a self-service platform with Azure Deployment Environments
- [ ] **(A)** Multi-region active-active with Cosmos DB + Front Door

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 30 tasks across 12 phases + 3 bonus challenges**

### Key Takeaways
- Azure is organized as Tenant â†’ MG â†’ Subscription â†’ RG â†’ Resources
- Entra ID is the identity layer; Managed Identities replace all secrets
- The VNet/NSG/App Gateway trio is the equivalent of AWS VPC/SG/ALB
- Azure Monitor + Log Analytics (KQL) is the observability stack
- Multi-region with Traffic Manager + Front Door + Cosmos is the HA pattern
- Hybrid (Arc, Stack) is Azure's unique strength

---

## ðŸŸ¢ Google Cloud Platform (GCP)

**Difficulty:** Advanced | **Time:** 45-55 hours

### Overview
GCP is Google's cloud and the home of services that leverage Google's own infrastructure â€” BigQuery (the data warehouse), Spanner (the globally distributed SQL), GKE (the original managed Kubernetes), and the global VPC. It is also the cloud with the strongest data + ML story. This tool covers every major GCP service, the project/folder/org hierarchy, IAM, networking, compute, data, serverless, and production patterns.

### Why Learn This
- ~10% market share, leader in data + ML
- GKE is the gold-standard managed Kubernetes
- The VPC is global (not regional) â€” unique among clouds
- BigQuery + Spanner are the reference implementations for cloud data
- The certifications (ACE, PCA, Professional Cloud Architect) are highly valued

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The hierarchy:**
   ```
   Organization
   â”œâ”€â”€ Folder
       â””â”€â”€ Project (the billable unit)
           â””â”€â”€ Resources
   ```
2. **The 4 services to know on day one:**
   - **Identity and Access Management (IAM):** the auth layer
   - **Cloud Resource Manager:** the API for orgs/folders/projects
   - **Cloud Monitoring + Logging:** the observability stack
   - **Cloud Billing:** the cost toolkit
3. **The 5-minute tour:**
   ```bash
   gcloud init
   gcloud auth login
   gcloud projects list
   gcloud config set project my-project
   gcloud compute regions list
   ```
4. **The GCP Architecture Framework â€” the 5 pillars:**
   | Pillar | Focus |
   |--------|-------|
   | Operational Excellence | Monitoring, automation, IaC |
   | Security | Identity, encryption, network |
   | Reliability | HA, DR, scaling |
   | Cost Optimization | Committed use, rightsizing |
   | Performance | Latency, throughput |
5. **Regions + Zones:**
   - 40+ regions
   - 3+ zones per region
   - **The VPC is global** â€” subnets are regional, resources are zonal
6. **Hands-on exercise:** Create a project, set up `gcloud`, enable 3 APIs (compute, container, monitoring), explore the console.

#### Step 2: IAM â€” The Foundation
1. **The 3 building blocks:**
   - **Principal:** user, group, service account
   - **Role:** a collection of permissions
   - **Policy:** binds a principal to a role on a resource
2. **The role hierarchy:**
   - **Basic:** Owner, Editor, Viewer (legacy â€” avoid)
   - **Predefined:** thousands, e.g., `roles/compute.admin`
   - **Custom:** your own, with specific permissions
3. **A service account â€” the workload identity:**
   ```bash
   gcloud iam service-accounts create myapp \
     --display-name="My App"
   gcloud projects add-iam-policy-binding my-project \
     --member="serviceAccount:myapp@my-project.iam.gserviceaccount.com" \
     --role="roles/storage.objectViewer"
   gcloud iam service-accounts keys create key.json \
     --iam-account=myapp@my-project.iam.gserviceaccount.com
   ```
4. **Workload Identity â€” the best practice for GKE:**
   - K8s SA â†’ GCP SA via IAM binding
   - No keys, no JSON
5. **A custom role with least privilege:**
   ```yaml
   # role.yaml
   title: "My App Storage User"
   stage: GA
   includedPermissions:
     - storage.buckets.get
     - storage.objects.get
     - storage.objects.create
   ```
   ```bash
   gcloud iam roles create myapp_storage_user \
     --project=my-project --file=role.yaml
   ```
6. **Access Context Manager + VPC Service Controls â€” for the secure perimeter:**
   - Perimeter around services, identity-aware, IP-aware
7. **Access Transparency â€” log every Google admin action.**
8. **Hands-on exercise:** Create a service account, bind a custom role to it, impersonate it from a VM with Workload Identity.

#### Step 3: The Network â€” VPC, Subnets, Firewall
1. **The primitives:**
   - **VPC:** global (the key differentiator)
   - **Subnet:** regional (in one region)
   - **Firewall rule:** stateful, allows or denies
   - **Cloud Router + Cloud NAT:** for private egress
   - **Cloud DNS:** managed DNS
2. **A production VPC:**
   ```bash
   gcloud compute networks create prod-vpc --subnet-mode=custom
   gcloud compute networks subnets create public-a \
     --network=prod-vpc --region=us-central1 --range=10.0.1.0/24
   gcloud compute networks subnets create private-a \
     --network=prod-vpc --region=us-central1 --range=10.0.10.0/24 \
     --enable-private-ip-google-access
   gcloud compute routers create cr-a --network=prod-vpc --region=us-central1
   gcloud compute routers nats create nat-a --router=cr-a \
     --region=us-central1 --nat-all-subnet-ip-ranges
   ```
3. **A firewall rule â€” the most important one (default-deny inbound):**
   ```bash
   gcloud compute firewall-rules create deny-all-in \
     --network=prod-vpc --direction=INGRESS --action=DENY \
     --rules=all --source-ranges=0.0.0.0/0 --priority=65000
   gcloud compute firewall-rules create allow-https \
     --network=prod-vpc --direction=INGRESS --action=ALLOW \
     --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https
   ```
4. **VPC Peering â€” 1-to-1, no transitive routing.**
5. **Cloud VPN â€” encrypted over internet:**
   - HA VPN: 99.99% SLA, two tunnels
6. **Cloud Interconnect â€” the dedicated line:**
   - 10, 100 Gbps
   - Partner or direct
7. **Shared VPC â€” share a VPC across projects:**
   - Host project owns the VPC
   - Service projects attach
8. **VPC Service Controls â€” the security perimeter for managed services.**
9. **Hands-on exercise:** Build a custom-mode VPC, 3 subnets in 1 region, Cloud NAT, firewall rules, peering to a second VPC.

#### Step 4: Compute â€” GCE, MIG, Sole-Tenant
1. **Create a VM:**
   ```bash
   gcloud compute instances create web \
     --machine-type=e2-standard-2 \
     --image-family=ubuntu-2204-lts --image-project=ubuntu-os-cloud \
     --zone=us-central1-a \
     --network=prod-vpc --subnet=private-a \
     --no-address \
     --service-account=myapp@my-project.iam.gserviceaccount.com \
     --scopes=cloud-platform \
     --tags=http-server,https-server \
     --metadata-from-file=startup-script=startup.sh
   ```
2. **Machine types:**
   - **General (E2, N2, N2D):** web, small DB
   - **Compute (C2, C2D):** CPU-bound
   - **Memory (M2, M3):** in-memory
   - **Accelerator (A2, A3, G2):** GPU/TPU
3. **Managed Instance Groups (MIG) â€” GCE's ASG:**
   ```bash
   gcloud compute instance-templates create web-tmpl \
     --machine-type=e2-standard-2 --image-family=ubuntu-2204-lts \
     --network=prod-vpc --subnet=private-a
   gcloud compute instance-groups managed create web-mig \
     --template=web-tmpl --size=3 --zone=us-central1-a
   gcloud compute instance-groups managed set-autoscaling web-mig \
     --min-num-replicas=3 --max-num-replicas=20 \
     --target-cpu-utilization=0.6
   ```
4. **Spot VMs + Committed Use Discounts + Sustained Use Discounts:**
   - Spot: 60-91% off, can be preempted
   - CUD: 1-3 year commit, up to 65% off
   - SUD: automatic for sustained usage
5. **Sole-tenant nodes â€” for compliance.**
6. **Hands-on exercise:** Create a MIG with autoscaling in private subnets, access via IAP SSH.

#### Step 5: Load Balancing â€” The 4 Types
1. **The 4 GCP load balancers:**
   | Type | Layer | Use |
   |------|-------|-----|
   | Network LB | L4 | TCP/UDP, ultra-low latency |
   | HTTP(S) LB | L7 | External, global, CDN |
   | Internal HTTP(S) | L7 | Internal, regional |
   | Internal TCP/UDP | L4 | Internal |
2. **The global HTTP(S) Load Balancer â€” the production default:**
   ```bash
   gcloud compute health-checks create http web-hc --port=8080 --request-path=/healthz
   gcloud compute backend-services create web-bes \
     --protocol=HTTP --health-checks=web-hc --global
   gcloud compute backend-services add-backend web-bes \
     --instance-group=web-mig --instance-group-zone=us-central1-a \
     --balancing-mode=UTILIZATION --max-utilization=0.8
   gcloud compute url-maps create web-lb --default-service=web-bes
   gcloud compute target-https-proxies create web-tp \
     --url-map=web-lb --ssl-certificates=web-cert
   gcloud compute forwarding-rules create web-fr \
     --address=web-ip --global --target-https-proxy=web-tp --ports=443
   ```
3. **Cloud CDN â€” built into the load balancer:**
   ```bash
   gcloud compute backend-services update web-bes --enable-cdn
   ```
4. **Cloud Armor â€” the WAF + DDoS protection:**
   ```bash
   gcloud compute security-policies create web-armor \
     --description="WAF for web"
   gcloud compute security-policies update web-armor \
     --type=CLOUD_ARMOR
   gcloud compute backend-services update web-bes --security-policy=web-armor
   ```
5. **Hands-on exercise:** Build a global HTTPS LB with a MIG backend, Cloud CDN, and a Cloud Armor WAF rule.

#### Step 6: Storage â€” GCS, Persistent Disk, Filestore
1. **Cloud Storage â€” the object store:**
   ```bash
   gsutil mb -l us-central1 -b on gs://myorg-logs
   gsutil cp file.txt gs://myorg-logs/
   gsutil iam ch allUsers:objectViewer gs://my-public  # only for public!
   ```
2. **Storage classes:**
   | Class | Use |
   |-------|-----|
   | Standard | Hot |
   | Nearline | 30+ days |
   | Coldline | 90+ days |
   | Archive | 180+ days |
3. **Object Lifecycle:**
   ```bash
   cat > lifecycle.json <<'EOF'
   {
     "lifecycle": {
       "rule": [
         { "action": { "type": "SetStorageClass", "storageClass": "NEARLINE" },
           "condition": { "age": 30 } },
         { "action": { "type": "SetStorageClass", "storageClass": "COLDLINE" },
           "condition": { "age": 90 } }
       ]
     }
   }
   EOF
   gsutil lifecycle set lifecycle.json gs://myorg-logs
   ```
4. **Versioning + Retention Policies + Object Holds:**
   ```bash
   gsutil versioning set on gs://myorg-logs
   gsutil retention set 90d gs://myorg-logs  # WORM-like
   ```
5. **Persistent Disk â€” block storage for VMs:**
   - Standard (HDD), Balanced (SSD), Performance (SSD), Extreme (SSD)
   - Snapshots, regional PD for HA
6. **Filestore â€” managed NFS.**
7. **Hands-on exercise:** Create a GCS bucket with versioning, lifecycle, uniform bucket-level access, and a signed URL.

#### Step 7: Database â€” Cloud SQL, Spanner, Firestore, Bigtable
1. **Cloud SQL â€” managed MySQL/Postgres/SQL Server:**
   ```bash
   gcloud sql instances create app \
     --database-version=POSTGRES_16 \
     --tier=db-custom-2-7680 --region=us-central1 \
     --availability-type=REGIONAL \
     --root-password="$DB_PASS" \
     --network=projects/my-project/global/networks/prod-vpc
   ```
2. **Cloud SQL features:**
   - HA (regional PD)
   - Read replicas (cross-region)
   - PITR (point-in-time recovery)
   - Automated backups
3. **Spanner â€” the globally distributed SQL:**
   - 5 9s availability, horizontal scaling
   - Strong consistency, ACID transactions
   - SQL interface, PostgreSQL dialect
4. **Firestore â€” the document DB:**
   - Native mode (the modern default) â€” strong consistency
   - Datastore mode â€” legacy
5. **Bigtable â€” the wide-column NoSQL:**
   - Petabyte scale, single-digit-ms reads/writes
   - The right tool for time-series, IoT, ad-tech
6. **Memorystore â€” Redis or Memcached.**
7. **The right database for the job:**
   | Need | Pick |
   |------|------|
   | Postgres, MySQL, single region | Cloud SQL |
   | Global, horizontally scaled SQL | Spanner |
   | Document, mobile/web | Firestore |
   | Time-series, wide-column | Bigtable |
   | Cache | Memorystore |
8. **Hands-on exercise:** Deploy Cloud SQL with HA, connect from a private VM, take a backup, restore.

#### Step 8: Serverless â€” Cloud Functions, Cloud Run, Workflows
1. **Cloud Functions â€” the FaaS:**
   ```bash
   gcloud functions deploy myfn \
     --runtime=python312 --trigger-http --allow-unauthenticated
   ```
2. **Cloud Run â€” the serverless containers:**
   ```bash
   gcloud run deploy myapp \
     --image=gcr.io/my-project/myapp:1.2.3 \
     --region=us-central1 \
     --min-instances=1 --max-instances=10 \
     --no-allow-unauthenticated
   ```
3. **Cloud Run features:**
   - Scale to 0 (or `--no-cpu-throttling`)
   - HTTPS endpoint, custom domain
   - Cloud SQL connector
   - Direct VPC egress (talk to private resources)
4. **Eventarc â€” the event router:**
   - Pub/Sub, GCS, Audit Logs â†’ Cloud Run
5. **Workflows â€” the orchestrator:**
   ```yaml
   main:
     params: [input]
     steps:
       - validate:
           call: http.post
           args:
             url: https://example.com/validate
             body: ${input}
       - process:
           call: http.post
           args:
             url: https://example.com/process
             body: ${input}
   ```
6. **Hands-on exercise:** Deploy a Cloud Run service from Artifact Registry, triggered by Pub/Sub, writes to Firestore.

#### Step 9: Containers on GCP â€” GKE, Cloud Run, Artifact Registry
1. **GKE â€” the gold-standard managed Kubernetes:**
   - **Autopilot:** fully managed nodes, pay per pod
   - **Standard:** you manage node pools
2. **GKE Autopilot â€” the modern default:**
   ```bash
   gcloud container clusters create-auto prod \
     --region=us-central1 --network=prod-vpc --subnetwork=private-a
   gcloud container clusters get-credentials prod --region=us-central1
   ```
3. **GKE Enterprise â€” the multi-cluster, multi-cloud Kubernetes:**
   - Fleet, Config Sync, Policy Controller, Config Connector
4. **Cloud Build â€” the build system:**
   ```yaml
   # cloudbuild.yaml
   steps:
     - name: gcr.io/cloud-builders/docker
       args: [build, -t, gcr.io/$PROJECT_ID/myapp:$SHORT_SHA, .]
     - name: gcr.io/cloud-builders/docker
       args: [push, gcr.io/$PROJECT_ID/myapp:$SHORT_SHA]
     - name: gcr.io/cloud-builders/gcloud
       args: [run, deploy, myapp, --image=gcr.io/$PROJECT_ID/myapp:$SHORT_SHA]
   ```
5. **Artifact Registry â€” the registry:**
   ```bash
   gcloud artifacts repositories create myapp --repository-format=docker --location=us-central1
   gcloud auth configure-docker us-central1-docker.pkg.dev
   ```
6. **Hands-on exercise:** Deploy a workload to GKE Autopilot with Workload Identity, HPA, and a LoadBalancer Service.

#### Step 10: Security Services
1. **Secret Manager â€” the secrets store:**
   ```bash
   gcloud secrets create db-pass --replication-policy=automatic
   echo -n "$DB_PASS" | gcloud secrets versions add db-pass --data-file=-
   ```
2. **Cloud KMS â€” encryption keys:**
   - HSM, software, EKM (external)
   - CMEK (customer-managed encryption keys) for all major services
3. **Security Command Center (SCC) â€” the CSPM:**
   - Standard (free) vs Premium (with threat detection)
4. **Cloud DLP â€” data loss prevention (PII scanner).**
5. **Identity-Aware Proxy (IAP) â€” zero-trust user access.**
6. **VPC Service Controls â€” the security perimeter.**
7. **Hands-on exercise:** Store a secret, access from a Cloud Run service with a dedicated SA, enable SCC standard.

#### Step 11: Observability â€” Cloud Monitoring, Logging, Trace
1. **Cloud Monitoring â€” metrics + alerting:**
   ```bash
   gcloud alpha monitoring policies create --policy-from-file=alert.yaml
   ```
2. **Cloud Logging â€” structured logs:**
   ```bash
   gcloud logging read 'resource.type=k8s_container severity=ERROR' --limit 50
   ```
3. **Log-based metrics â€” for SLI/SLO:**
   ```bash
   gcloud logging metrics create error_count \
     --description="Number of errors" \
     --log-filter='severity=ERROR'
   ```
4. **Cloud Trace â€” distributed tracing:**
   - OpenTelemetry auto-instrumentation
5. **Error Reporting â€” auto-groups exceptions.**
6. **SLOs + Burn Rate Alerts:**
   ```bash
   gcloud monitoring slos create myapp-availability \
     --service=myapp \
     --goal=0.999 --period=PERIOD_30_DAYS \
     --method=REQUEST_BASED \
     --distribution-cut=0.99
   ```
7. **Hands-on exercise:** Set up SLOs for a service, alert on burn rate, view the dashboard.

#### Step 12: Cost & Governance
1. **Cloud Billing â€” budgets, alerts, exports:**
   ```bash
   gcloud billing budgets create --billing-account=$BILLING --budget-from-file=budget.yaml
   ```
2. **Committed Use Discounts + Sustained Use + Spot.**
3. **Organization Policy Constraints â€” guardrails:**
   ```bash
   gcloud resource-manager org-policies set-policy policy.yaml
   ```
4. **IAM Conditions â€” context-aware access (time, IP, resource).**
5. **Tags + Tag bindings (the GCP equivalent of AWS tags):**
   ```bash
   gcloud resource-manager tags bindings create \
     --tag-value=tagValues/12345 --resource=//compute.googleapis.com/projects/...
   ```
6. **Hands-on exercise:** Apply a constraint to deny resource creation outside `us-central1`; set a budget alert.

#### Step 13: Multi-Region, DR, and the Enterprise Patterns
1. **Multi-region:**
   - **Global LB:** anycast IP, routes by health
   - **Spanner:** multi-region instance
   - **Cloud Storage:** multi-region or dual-region buckets
2. **DR patterns:**
   - **Backup/restore:** Coldline, snapshots
   - **Pilot Light:** minimal standby
   - **Warm Standby:** scaled-down running
   - **Multi-region active-active:** global LB + Spanner
3. **Anthos â€” the multi-cluster, multi-cloud Kubernetes:**
   - On-prem (Anthos on bare metal / VMware)
   - AWS (Anthos on AWS)
   - Azure (Anthos on Azure)
4. **Migration tools:**
   - **Migrate to Virtual Machines**
   - **Database Migration Service (DMS)**
   - **Transfer Appliance** (for petabytes)
5. **Hands-on exercise:** Set up a multi-region Cloud Storage bucket, replicate, set up global LB in front of two MIGs.

#### Step 14: Production-Grade GCP Habits
- **Projects by env, folders by team** â€” the billable unit is the project
- **IAM with predefined/custom roles** â€” never `roles/owner`
- **Workload Identity for GKE** â€” no JSON keys
- **Private IP everywhere** â€” Private Service Connect
- **Cloud Monitoring + Logging** â€” observability by default
- **Labels + Tags** â€” for cost + ownership
- **Org policies** â€” region/service restrictions
- **CUDs + Spot** â€” commit for baseline, spot for burst
- **DR with multi-region + Spanner** â€” for the critical
- **Practice incident response** â€” DiRT (Disaster Recovery Testing)

### Practice Project â€” Production-Grade 3-Tier on GCP

> **Goal:** Build a production-shape GCP environment â€” multi-region, full observability, IaC, CI/CD, DR.
> **Time estimate:** 25-30 hours
> **Deliverables:** A Terraform-managed infra, a deployed app, a working CI/CD, and a full runbook.

#### Phase 1 â€” Org & Identity (2h)
- [ ] **(I)** Set up Org, Folders (Common, Dev, Prod), Projects
- [ ] **(I)** Bind billing to a project
- [ ] **(I)** Create 3 service accounts (app, ci, ops)
- [ ] **(I)** Custom role with least-privilege for the app SA

#### Phase 2 â€” Network (2.5h)
- [ ] **(I)** Build a custom-mode VPC
- [ ] **(I)** Add 3 subnets (public, private, data)
- [ ] **(I)** Cloud NAT, Cloud Router
- [ ] **(I)** Firewall rules: default-deny, allow-https, allow-iap
- [ ] **(A)** VPC peering + Private Service Connect for Cloud SQL

#### Phase 3 â€” Compute & LB (2.5h)
- [ ] **(I)** Create a MIG in private subnets
- [ ] **(I)** Build a global HTTPS LB with CDN
- [ ] **(I)** Add Cloud Armor WAF with rate limiting
- [ ] **(A)** Add CUD for baseline + spot for burst

#### Phase 4 â€” Database (1.5h)
- [ ] **(I)** Cloud SQL Postgres with HA
- [ ] **(I)** Private IP, backup, PITR
- [ ] **(A)** Spanner for a second workload (demo multi-region)

#### Phase 5 â€” Storage (1h)
- [ ] **(I)** GCS bucket with versioning, lifecycle, uniform access
- [ ] **(I)** Multi-region bucket for the logs
- [ ] **(A)** Add retention policy (WORM)

#### Phase 6 â€” Containers (1.5h)
- [ ] **(I)** GKE Autopilot cluster
- [ ] **(I)** Workload Identity, HPA, Ingress
- [ ] **(A)** GKE Enterprise: Config Sync for GitOps

#### Phase 7 â€” Serverless (1h)
- [ ] **(I)** Cloud Run from Artifact Registry
- [ ] **(I)** Cloud Function triggered by Eventarc on GCS upload
- [ ] **(A)** Workflows: orchestrate a 3-step process

#### Phase 8 â€” Security (1.5h)
- [ ] **(I)** Secret Manager + Workload Identity
- [ ] **(I)** CMEK for GCS + Cloud SQL
- [ ] **(I)** Enable SCC Standard
- [ ] **(A)** VPC Service Controls perimeter

#### Phase 9 â€” Observability (1.5h)
- [ ] **(I)** SLO: 99.9% availability, burn rate alert
- [ ] **(I)** Custom dashboards
- [ ] **(A)** Cloud Trace across services

#### Phase 10 â€” CI/CD (2h)
- [ ] **(I)** Cloud Build â†’ Artifact Registry
- [ ] **(I)** Deploy to GKE / Cloud Run
- [ ] **(I)** GitHub Actions with Workload Identity Federation
- [ ] **(A)** Add canary deploy with Cloud Deploy

#### Phase 11 â€” DR & Cost (1.5h)
- [ ] **(I)** Multi-region GCS, Spanner or cross-region read replica
- [ ] **(I)** Budget alert
- [ ] **(I)** Org policy: region lock
- [ ] **(A)** DiRT drill: simulate a regional outage

#### Phase 12 â€” Documentation (1h)
- [ ] **(I)** Architecture diagram
- [ ] **(I)** `RUNBOOK.md`, `SECURITY.md`, `COST.md`
- [ ] **(A)** SLO dashboard for the app

#### Bonus Challenges
- [ ] **(A)** Migrate an app to GKE Enterprise (multi-cluster, Config Sync)
- [ ] **(A)** Build a BigQuery data lake + dashboard
- [ ] **(A)** Implement a multi-region active-active with Spanner + Global LB

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 30 tasks across 12 phases + 3 bonus challenges**

### Key Takeaways
- GCP is organized as Org â†’ Folder â†’ Project â†’ Resources
- The VPC is global â€” subnets are regional, resources are zonal
- Workload Identity replaces all service account keys in GKE
- GKE Autopilot is the easiest managed K8s, BigQuery is the data king
- Multi-region is built-in (Spanner, Global LB, multi-region GCS)
- GCP's data + ML story is the strongest of the three

---

## ðŸ¢ On-Prem Networking & Storage

**Difficulty:** Advanced | **Time:** 30-35 hours

### Overview
The cloud is not the whole story. Many enterprises run hybrid â€” a data center with bare metal, KVM/VMware, network fabrics, BGP, and storage arrays â€” connected to one or more clouds. The on-prem world has its own standards, vendors, and patterns: leaf-spine fabrics, VXLAN-EVPN overlays, BGP route servers, OSPF, MPLS, NetApp, Ceph, vSAN, iSCSI, NFS, and SDN controllers like NSX and Cilium. This tool covers the on-prem network architecture, the overlay/underlay model, the storage stack, and how to bridge it all to the cloud.

### Why Learn This
- 60%+ of enterprises run hybrid â€” on-prem is not dying
- The on-prem network is the substrate that holds the cloud connectivity together
- Ceph, NetApp, vSAN are the reference implementations for software-defined storage
- NSX, Cilium, Calico are the on-prem SDN equivalents
- Mastering on-prem is the difference between "I can run apps in the cloud" and "I can run any infrastructure anywhere"

### Learn by Doing â€” Step by Step

#### Step 1: The Data Center Architecture
1. **The classic three-tier:**
   ```
   Internet â†’ Firewall (Edge) â†’ Core (L3) â†’ Aggregation (L3) â†’ Access (L2) â†’ Servers
   ```
   - **Edge:** border, internet, peering
   - **Core:** fast L3, BGP/OSPF
   - **Aggregation:** services (firewall, load balancer)
   - **Access:** top-of-rack, L2
2. **The modern leaf-spine:**
   ```
   Server â†” Leaf (ToR) â†” Spine â†” Leaf (ToR) â†” Server
   ```
   - **Any server is exactly 2 hops from any other**
   - BGP or OSPF on the underlay
   - VXLAN-EVPN on the overlay
   - The standard for new builds (Facebook, Microsoft, Google)
3. **The super-spine for multi-pod:**
   ```
   Pod1 (Leaf+Spine) â†” Super-Spine â†” Pod2 (Leaf+Spine)
   ```
4. **Vendor examples:**
   - **Cisco Nexus 9000:** the industry default
   - **Arista 7000:** performance + EOS
   - **Juniper QFX:** Mist AI for ops
   - **Mellanox/NVIDIA Spectrum:** low-latency, AI/ML
   - **Dell EMC:** cost-optimized
   - **White-box (Cumulus, SONiC, FBOSS):** open NOS
5. **Hands-on exercise:** Draw the leaf-spine topology for a 4-rack, 2-pod DC; identify the underlay and overlay.

#### Step 2: The Underlay â€” BGP/OSPF, ECMP, MTU
1. **The underlay goal:** deterministic, lossless, low-latency L3 connectivity.
2. **BGP on the underlay (the modern choice):**
   - Each leaf is in its own AS
   - Each spine is in its own AS
   - eBGP between leaf and spine
   - ECMP for load balancing
3. **The config on a leaf (FRR / Cumulus / Arista EOS):**
   ```bgp
   router bgp 65001
     neighbor spine1 remote-as 65100
     neighbor spine2 remote-as 65100
     address-family ipv4 unicast
       network 10.0.0.0/24
       maximum-paths 64
   ```
4. **OSPF on the underlay (the alternative):**
   - Simpler to start
   - Doesn't scale as gracefully
   - Used in older Cisco shops
5. **MTU â€” the silent killer:**
   - **Jumbo frames:** 9000+ bytes end-to-end
   - Set MTU on every interface, every switch, every NIC
   - Path MTU discovery is a fallback, not a strategy
6. **ECMP â€” load balancing over equal paths:**
   - 64-way ECMP on modern switches
   - Hash: L3/L4, sometimes L2
7. **Hands-on exercise:** Set up FRR on two VMs in different AS, advertise /24s, verify ECMP and path selection.

#### Step 3: The Overlay â€” VXLAN-EVPN
1. **The overlay goal:** stretch L2 segments across L3 fabric.
2. **VXLAN â€” encapsulate L2 in L3:**
   - 24-bit VNI = 16M segments
   - UDP port 4789
   - Outer IP = the underlay
3. **EVPN â€” the BGP-based control plane:**
   - Type-2 routes: MAC/IP advertisement
   - Type-3 routes: VTEP discovery
   - Type-5 routes: IP prefix (L3 VNI)
4. **The roles:**
   - **VTEP:** the endpoint that encapsulates/decapsulates (leaf, server, hypervisor)
   - **BGP route reflector:** reduces full-mesh
5. **A leaf with FRR VXLAN-EVPN:**
   ```bash
   # /etc/frr/frr.conf
   router bgp 65001
     neighbor rr peer-group
     neighbor 10.0.0.1 remote-as 65001
     address-family l2vpn evpn
       neighbor rr activate
       advertise-all-vni
   ```
6. **Hands-on exercise:** Build a 2-leaf VXLAN-EVPN topology with FRR; ping across the L2-over-L3 tunnel.

#### Step 4: Kubernetes Networking On-Prem â€” CNI Deep Dive
1. **The CNI taxonomy:**
   | CNI | Underlay | Overlay | Policy | Service Mesh |
   |-----|----------|---------|--------|--------------|
   | Flannel | L2/L3 | VXLAN/host-gw | No | No |
   | Calico | L2/L3 | BGP / IPIP / VXLAN | Yes (L3-L4) | No |
   | Cilium | L2/L3 | VXLAN / native | Yes (L3-L7) | Hubble + eBPF |
   | Weave | L2/L3 | VXLAN | Yes | No |
   | Multus | â€” | â€” | â€” | Multi-NIC attachment |
2. **Calico â€” the BGP-native CNI:**
   - BGP for pod IP advertisement
   - Optional IPIP or VXLAN overlay
   - NetworkPolicy + GlobalNetworkPolicy
   - Calico Enterprise: eBPF, WireGuard encryption
3. **Cilium â€” the eBPF-native CNI:**
   - eBPF replaces kube-proxy
   - Hubble for observability
   - Cilium Network Policy (L3-L7, FQDN, DNS-aware)
   - Cluster Mesh for multi-cluster
4. **The on-prem K8s networking stack:**
   ```
   Pod â†” CNI (Calico/Cilium) â†” Leaf â†” Spine â†” Leaf â†” CNI â†” Pod
   ```
5. **The IPAM choice:**
   - **Calico IPAM:** local pools, BGP
   - **Cilium:** native routing (no overlay), cluster-pool
   - **Host-local:** each node allocates (Flannel-style)
6. **Hands-on exercise:** Install Calico with BGP; advertise pod CIDRs from a kind cluster to FRR; verify.

#### Step 5: Service Mesh On-Prem â€” Istio, Linkerd, Consul Connect
1. **The case:** mTLS between every service, traffic management, observability.
2. **Istio â€” the most popular:**
   - Envoy sidecar (or ambient mode, sidecar-less)
   - Control plane: Istiod
   - mTLS, traffic shifting, fault injection, telemetry
3. **Linkerd â€” the lightweight alternative:**
   - Linkerd2-proxy (Rust, micro-fewer resources)
   - Linkerd Viz for dashboards
4. **Consul Connect â€” HashiCorp's mesh:**
   - Integrates with Consul service discovery
   - Native or sidecar
5. **Cilium Service Mesh â€” the eBPF alternative:**
   - Sidecar-less, no proxy
   - L7 policy without sidecars
6. **Hands-on exercise:** Install Istio; deploy 2 services; enable strict mTLS; visualize the service graph in Kiali.

#### Step 6: Software-Defined Networking â€” NSX & the Vendors
1. **VMware NSX â€” the on-prem SDN leader:**
   - NSX-T: the modern version (KVM, ESXi, bare metal, public cloud)
   - Overlay: GENEVE
   - Distributed firewall, load balancer, VPN
2. **The NSX architecture:**
   ```
   NSX Manager (control plane)
   â”œâ”€â”€ NSX Edge (services: LB, VPN, NAT)
   â””â”€â”€ Hypervisor Transport Node (data plane)
   ```
3. **The alternatives:**
   - **Cisco ACI:** the original SDN, big-bang approach
   - **Juniper Contrail / Apstra:** intent-based networking
   - **Nutanix Flow:** microsegmentation
   - **Pluribus Networks:** adaptive fabric
4. **Hands-on exercise:** Stand up NSX-T in a lab (nested ESXi, or the free NSX-T evaluation); create a logical switch + segment.

#### Step 7: DNS, DHCP, IPAM (DDI) On-Prem
1. **The DDI stack:**
   - **Infoblox:** the enterprise default
   - **Microsoft DNS + DHCP + IPAM:** the Windows way
   - **BIND + ISC DHCP + NetBox:** the open-source
   - **BlueCat, EfficientIP:** the alternatives
2. **The NetBox way (modern, free, GitOps-friendly):**
   ```python
   # ipam.create_prefix
   prefix = Prefix(prefix="10.0.0.0/16", site=site, vrf=vrf, role=role)
   ```
3. **DNS for Kubernetes â€” CoreDNS on the underlay:**
   - Service DNS: `<svc>.<ns>.svc.cluster.local`
   - ExternalDNS â€” sync Services/Ingresses to the DDI
4. **Hands-on exercise:** Stand up NetBox, document a leaf-spine IPAM, sync K8s Services via ExternalDNS.

#### Step 8: Storage On-Prem â€” The Three Layers
1. **Block storage:**
   - **DAS:** local disks, JBOD, RAID
   - **SAN:** Fibre Channel, iSCSI â€” the enterprise default
   - **Software-defined:** vSAN, Ceph RBD, OpenEBS
2. **File storage:**
   - **NFS:** the Unix default, served by NetApp, Isilon, Linux NFS
   - **SMB:** the Windows default
   - **GlusterFS, CephFS:** the open-source scale-out
3. **Object storage:**
   - **MinIO:** the S3-compatible open-source
   - **Ceph Object Gateway (RGW):** S3 + Swift
   - **Scality, Cloudian:** the enterprise
4. **Hands-on exercise:** Deploy MinIO on 4 nodes with erasure coding; verify S3 API + multipart upload.

#### Step 9: Ceph â€” The Open Source Storage King
1. **The three services:**
   - **Ceph OSD:** object storage daemon (one per disk)
   - **Ceph MON:** monitors (consensus)
   - **Ceph MDS:** metadata server (for CephFS)
   - **Ceph RGW:** S3/Swift gateway
2. **The architecture:**
   ```
   Clients
   â”œâ”€â”€ librados (direct)
   â”œâ”€â”€ RBD (block)
   â”œâ”€â”€ CephFS (file)
   â””â”€â”€ RGW (object)
       â†“
   OSDs (CRUSH placement)
   ```
3. **CRUSH â€” the placement algorithm:**
   - No central lookup table
   - Maps objects to OSDs by rule
   - Rules encode failure domains (host, rack, row, room)
4. **Deploy with Rook on Kubernetes:**
   ```bash
   helm repo add rook-release https://charts.rook.io/release
   helm install rook-ceph rook-release/rook-ceph -n rook-ceph --create-namespace
   helm install rook-ceph-cluster rook-release/rook-ceph-cluster -n rook-ceph -f values.yaml
   ```
5. **Hands-on exercise:** Deploy a 3-node Ceph cluster; create an RBD pool, a CephFS, an RGW bucket.

#### Step 10: NetApp & the Enterprise Arrays
1. **NetApp ONTAP â€” the enterprise default:**
   - Unified storage (NAS + SAN)
   - Snapshot, replication, dedup, compression
   - ONTAP Select: software-defined on commodity hardware
   - Cloud Volumes ONTAP: ONTAP in AWS/Azure/GCP
2. **Pure Storage FlashArray â€” all-flash performance.**
3. **Dell EMC PowerStore / UnityXT.**
4. **HPE Nimble / Primera.**
5. **The protocols:**
   - NFS, SMB (file)
   - iSCSI, FC (block)
   - S3 (object, on modern arrays)
6. **The features you pay for:**
   - **Snapshots:** point-in-time, space-efficient
   - **Clones:** writable snapshots
   - **Replication:** sync (metro cluster) or async (SnapMirror)
   - **DR:** orchestrated failover
   - **QoS:** per-volume, per-tenant
7. **Hands-on exercise:** Set up ONTAP Select in a VM; create an NFS export + iSCSI LUN; snapshot + restore.

#### Step 11: Hyperconverged Infrastructure (HCI)
1. **The vendors:**
   - **Nutanix:** the leader, AHV + Prism
   - **VMware vSAN:** the incumbent
   - **Scale Computing, HPE SimpliVity, Cisco HyperFlex**
2. **The architecture:** compute + storage in one box, distributed.
3. **Nutanix â€” the most modern:**
   - **AHV:** the built-in KVM hypervisor
   - **Prism Central:** multi-cluster mgmt
   - **Flow:** microsegmentation
   - **Karbon:** managed Kubernetes
   - **Objects:** S3-compatible
4. **vSAN â€” the VMware path:**
   - Requires vCenter
   - Stretched cluster for HA
5. **Hands-on exercise:** Stand up Nutanix CE (Community Edition) on 3 nested VMs; create a VM, a Volume Group, a Kubernetes cluster.

#### Step 12: Connectivity to the Cloud
1. **Direct Connect (AWS) / ExpressRoute (Azure) / Interconnect (GCP):**
   - Dedicated line, 1-100 Gbps
   - Lower latency, higher throughput, more reliable
2. **VPN over internet:**
   - IPsec, IKEv2
   - Cheap, easy, less reliable
3. **The hybrid DNS pattern:**
   - On-prem DNS + cloud DNS (Route 53 Resolver, Azure DNS Private Resolver, Cloud DNS)
   - Forwarders in both directions
4. **The hybrid identity pattern:**
   - On-prem AD â†’ cloud (AD Connect, CloudSync)
   - The cloud identity is the source of truth for cloud
5. **Storage replication to the cloud:**
   - **AWS:** DataSync, Storage Gateway
   - **Azure:** Data Box, File Sync
   - **GCP:** Transfer Appliance, Storage Transfer Service
6. **Hands-on exercise:** Set up an IPsec VPN from on-prem to a cloud VPC; verify connectivity, DNS, and storage replication.

#### Step 13: Observability On-Prem
1. **The stack:**
   - **Metrics:** Prometheus + Grafana, or Telegraf + InfluxDB
   - **Logs:** ELK, Loki + Promtail, or Splunk
   - **Traces:** Jaeger, Tempo, or Datadog
2. **Network monitoring:**
   - **SNMP:** the legacy, ubiquitous
   - **Streaming telemetry:** gNMI, OpenConfig â€” the modern
   - **sFlow / NetFlow:** for traffic analysis
3. **The open-source observability stack:**
   ```bash
   # Prometheus + Grafana
   helm install kube-prom prometheus-community/kube-prometheus-stack
   # Loki for logs
   helm install loki grafana/loki-stack
   # Tempo for traces
   helm install tempo grafana/tempo
   ```
4. **Hands-on exercise:** Set up the full observability stack on a kind cluster; stream switch metrics via SNMP exporter.

#### Step 14: Production-Grade On-Prem Habits
- **Diagram every change** â€” DCIM, NetBox, Lumos
- **Version the network** â€” GitOps for switches (Cumulus, SONiC, Junos)
- **Test the underlay** â€” RFC 2544, 2889, Y.1564 (ethernet testing)
- **Encrypt in transit** â€” MACsec on the fabric, WireGuard between sites
- **Back up the control plane** â€” NSX Manager, vCenter, NetApp SMVI
- **Practice DR** â€” DC failure, switch failure, storage failure
- **Monitor power, cooling, space** â€” the data center's own health
- **Document the runbook** â€” every cable, every IP, every label

### Practice Project â€” Production-Grade On-Prem Lab

> **Goal:** Build a 2-pod, 4-leaf, 2-spine leaf-spine lab with VXLAN-EVPN, Calico/Cilium on K8s, Ceph, MinIO, NSX-T, and full observability.
> **Time estimate:** 25-30 hours
> **Deliverables:** A working lab, BGP + VXLAN-EVPN configs, Ceph cluster, and a runbook.

#### Phase 1 â€” Lab (3h)
- [ ] **(I)** Stand up 4 nested VMs (leaves) + 2 VMs (spines) with FRR
- [ ] **(I)** Configure eBGP on the underlay, verify ECMP
- [ ] **(I)** Add a third pod with a super-spine
- [ ] **(A)** Use Cumulus VX (free) for a real switch NOS

#### Phase 2 â€” Overlay (2.5h)
- [ ] **(I)** Enable VXLAN-EVPN between leaves
- [ ] **(I)** Add a route reflector; reduce full-mesh
- [ ] **(I)** Stretch a VLAN across all 4 leaves
- [ ] **(A)** Add EVPN Type-5 (L3 VNI) for inter-VXLAN routing

#### Phase 3 â€” K8s + CNI (3h)
- [ ] **(I)** Deploy a 3-node K8s cluster (kubeadm or K3s)
- [ ] **(I)** Install Calico with BGP; advertise pod CIDRs
- [ ] **(I)** Switch to Cilium; verify Hubble observability
- [ ] **(A)** Implement multi-cluster with Cilium Cluster Mesh

#### Phase 4 â€” Storage (3h)
- [ ] **(I)** Deploy a 3-node Ceph cluster (Cephadm or Rook)
- [ ] **(I)** Create an RBD pool, mount in a K8s pod
- [ ] **(I)** Create a CephFS, mount in 2 pods
- [ ] **(I)** Deploy MinIO on top of Ceph RGW (or separately)
- [ ] **(A)** Set up snapshots + restore

#### Phase 5 â€” Service Mesh (2h)
- [ ] **(I)** Install Istio; deploy 2 services
- [ ] **(I)** Enable strict mTLS; verify with a curl
- [ ] **(I)** Add Kiali for the service graph
- [ ] **(A)** Implement traffic shifting for a canary

#### Phase 6 â€” NSX-T (2h)
- [ ] **(A)** Stand up NSX-T in the lab (nested ESXi or KVM)
- [ ] **(A)** Create a logical switch, a segment, a tier-1 gateway
- [ ] **(A)** Enable distributed firewall
- [ ] **(A)** Connect NSX to the K8s cluster via NCP

#### Phase 7 â€” DDI (1.5h)
- [ ] **(I)** Stand up NetBox; document the IPAM
- [ ] **(I)** Configure BIND + ISC DHCP for the lab
- [ ] **(I)** Install ExternalDNS; sync K8s Services to BIND
- [ ] **(A)** Use the Infoblox API to allocate IPs

#### Phase 8 â€” Observability (1.5h)
- [ ] **(I)** Deploy Prometheus + Grafana
- [ ] **(I)** Add SNMP exporter for the leaves
- [ ] **(I)** Loki for logs, Tempo for traces
- [ ] **(A)** Add streaming telemetry (gNMI) from a Cumulus switch

#### Phase 9 â€” Cloud Connectivity (2h)
- [ ] **(I)** Spin up a cloud VPC, set up IPsec VPN
- [ ] **(I)** Verify pod-to-VM connectivity
- [ ] **(I)** Sync DNS both ways
- [ ] **(A)** Set up a Direct Connect/ExpressRoute equivalent in the lab (with another pair of VMs)

#### Phase 10 â€” Documentation (1h)
- [ ] **(I)** Architecture diagram (underlay + overlay)
- [ ] **(I)** IPAM in NetBox
- [ ] **(I)** `RUNBOOK.md`: rebuild each layer
- [ ] **(A)** DR drill runbook: switch failure, Ceph failure, cluster failure

#### Bonus Challenges
- [ ] **(A)** Implement GitOps for the switches (Cumulus + Ansible or Junos + PyEZ)
- [ ] **(A)** Add a Nutanix CE cluster; migrate one workload to it
- [ ] **(A)** Build an Active-Active stretched Ceph across 2 sites

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 26 tasks across 10 phases + 3 bonus challenges**

### Key Takeaways
- The on-prem data center is a layered stack: underlay, overlay, K8s, storage, mesh
- Leaf-spine + VXLAN-EVPN is the modern fabric
- Calico/Cilium are the K8s networking standards
- Ceph is the open-source storage king; NetApp is the enterprise default
- Hybrid is real â€” every enterprise will have on-prem + cloud for the foreseeable future

---

## ðŸ“Š Observability Stack

**Difficulty:** Intermediate | **Time:** 25-30 hours

### Overview
Observability is the ability to ask arbitrary questions about a system without shipping new code. It rests on the three signals â€” metrics, logs, traces â€” plus the derived signals: profiles, events, audit logs, and SLI/SLO budgets. The open-source stack (Prometheus, Grafana, Loki, Tempo, OpenTelemetry) is the de facto standard; the SaaS stacks (Datadog, New Relic, Honeycomb, Dynatrace) are the paid alternatives. Mastering observability is the difference between a system you operate by guesswork and a system you operate by data.

### Why Learn This
- The 3 signals are the universal language of on-call
- OpenTelemetry is the vendor-neutral standard; every modern tool emits or accepts it
- SLI/SLO + error budgets are how you balance reliability and velocity
- Cost observability is the new frontier (FinOps)
- The SaaS vendors (Datadog, etc.) are excellent but expensive; the open-source stack is the default

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The three signals:**
   | Signal | Question | Example |
   |--------|----------|---------|
   | Metrics | How much / how many? | CPU%, request rate, queue depth |
   | Logs | What happened? | "User X failed login" |
   | Traces | Where did time go? | The path of one request through 7 services |
2. **The two newer signals:**
   - **Profiles:** continuous profiling (Pyroscope, Parca, Datadog Continuous Profiler)
   - **Events:** deploys, alerts, config changes (event metadata)
3. **The cardinality trap:**
   - **Low cardinality:** status, region, method (good for metrics)
   - **High cardinality:** user_id, request_id, order_id (good for logs, bad for metrics)
4. **RED + USE + Golden Signals â€” the four frameworks:**
   - **RED (services):** Rate, Errors, Duration
   - **USE (resources):** Utilization, Saturation, Errors
   - **Golden Signals:** Latency, Traffic, Errors, Saturation
   - **Four Golden Signals** (SRE book): add Saturation to RED
5. **The stack components:**
   - **Instrumentation:** OpenTelemetry SDK
   - **Collection:** OpenTelemetry Collector
   - **Storage:** Prometheus, Loki, Tempo, S3
   - **Query:** PromQL, LogQL, TraceQL
   - **Visualization:** Grafana
6. **Hands-on exercise:** Set up a minimal stack: OpenTelemetry Collector â†’ Prometheus + Loki + Tempo â†’ Grafana.

#### Step 2: OpenTelemetry â€” The Vendor-Neutral Standard
1. **The two parts:**
   - **API:** the language-level calls (Tracer, Meter, Logger)
   - **SDK:** the implementation that exports
2. **The signals:**
   - **Traces:** `tracer.start_as_current_span("name")`
   - **Metrics:** `meter.create_counter("name")`
   - **Logs:** `logger.info("message")`
3. **The exporters:**
   - **OTLP:** the standard protocol
   - **Prometheus, Jaeger, Zipkin:** legacy
4. **A minimal Python example:**
   ```python
   from opentelemetry import trace, metrics
   from opentelemetry.sdk.trace import TracerProvider
   from opentelemetry.sdk.metrics import MeterProvider
   from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
   from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter
   from opentelemetry.sdk.trace.export import BatchSpanProcessor
   from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
   from opentelemetry.sdk.resources import Resource

   resource = Resource.create({"service.name": "myapp"})
   tp = TracerProvider(resource=resource)
   tp.add_span_processor(BatchSpanProcessor(OTLPSpanExporter(endpoint="otel-collector:4317", insecure=True)))
   trace.set_tracer_provider(tp)
   tracer = trace.get_tracer(__name__)

   with tracer.start_as_current_span("deploy") as span:
       span.set_attribute("env", "prod")
       # work
   ```
5. **The OpenTelemetry Collector â€” the central hub:**
   ```yaml
   # otel-collector-config.yaml
   receivers:
     otlp:
       protocols: { grpc: {}, http: {} }
   exporters:
     prometheusremotewrite:
       endpoint: http://prometheus:9090/api/v1/write
     loki:
       endpoint: http://loki:3100/loki/api/v1/push
     otlp/tempo:
       endpoint: tempo:4317
       tls: { insecure: true }
   service:
     pipelines:
       traces:
         receivers: [otlp]
         exporters: [otlp/tempo]
       metrics:
         receivers: [otlp]
         exporters: [prometheusremotewrite]
       logs:
         receivers: [otlp]
         exporters: [loki]
   ```
6. **Auto-instrumentation:** OpenTelemetry has zero-code agents for Java, Python, Node, Go, .NET.
7. **Hands-on exercise:** Instrument a Python app with OTel; export to a local collector; view in Jaeger or Tempo.

#### Step 3: Prometheus â€” The Metrics Backbone
1. **The data model:**
   - **Metric name:** `http_requests_total`
   - **Labels:** `{method="GET", status="200"}`
   - **Value:** float64
   - **Timestamp:** ms since epoch
2. **The metric types:**
   | Type | Use |
   |------|-----|
   | Counter | Monotonically increasing |
   | Gauge | Up and down |
   | Histogram | Bucketed observations (latency, size) |
   | Summary | Quantiles, but not aggregatable |
3. **PromQL â€” the query language:**
   ```promql
   # Request rate per second
   rate(http_requests_total[5m])
   # Error rate
   sum(rate(http_requests_total{status=~"5.."}[5m])) /
   sum(rate(http_requests_total[5m]))
   # P99 latency
   histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
   ```
4. **Scrape config:**
   ```yaml
   scrape_configs:
     - job_name: myapp
       kubernetes_sd_configs:
         - role: pod
       relabel_configs:
         - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
           action: keep
           regex: true
       metrics_path: /metrics
   ```
5. **Alerting rules:**
   ```yaml
   groups:
     - name: myapp
       rules:
         - alert: HighErrorRate
           expr: |
             sum(rate(http_requests_total{status=~"5.."}[5m])) /
             sum(rate(http_requests_total[5m])) > 0.05
           for: 5m
           labels: { severity: critical }
           annotations:
             summary: "Error rate > 5% for 5m"
             runbook: https://wiki/runbooks/high-error-rate
   ```
6. **Prometheus storage:**
   - Default: local TSDB
   - **Thanos:** unlimited retention, S3-backed
   - **Cortex / Mimir:** multi-tenant
   - **Victoria Metrics:** drop-in, smaller
7. **The 10 best practices:**
   - Use recording rules for expensive queries
   - Drop high-cardinality labels
   - Set scrape intervals deliberately (15s default)
   - Use exemplars to link metrics â†’ traces
   - Set retention deliberately (15d default, longer via remote)
   - Use federation for multi-cluster
8. **Hands-on exercise:** Install Prometheus; scrape a /metrics endpoint; write 5 PromQL queries; set 2 alerts.

#### Step 4: Grafana â€” The Visualization Layer
1. **The data sources:** Prometheus, Loki, Tempo, MySQL, Postgres, CloudWatch, BigQuery, Elasticsearch, Datadog, etc.
2. **The dashboard JSON:**
   ```json
   {
     "title": "My App Golden Signals",
     "panels": [
       { "type": "timeseries", "title": "Request Rate", "targets": [
         { "expr": "sum(rate(http_requests_total[1m]))" } ] },
       { "type": "stat", "title": "P99 Latency", "targets": [
         { "expr": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))" } ] }
     ]
   }
   ```
3. **Grafana as code â€” Grafonnet, Jsonnet, or Helm:**
   ```bash
   helm install grafana grafana/grafana -f values.yaml
   # Provision dashboards via ConfigMap
   ```
4. **Grafana alerting (v9+):**
   - Define alert rules in the UI or YAML
   - Route via contact points (Slack, PagerDuty, Opsgenie, webhook)
5. **The 4 dashboards every service must have:**
   - **Golden signals** (rate, errors, duration, saturation)
   - **Saturation** (CPU, memory, queue depth)
   - **Errors** (top errors, error trend)
   - **Dependencies** (DB, cache, downstream services)
6. **Hands-on exercise:** Import the official Prometheus stats dashboard; build a 4-panel golden signals dashboard.

#### Step 5: Loki â€” The Logs Stack
1. **The design:** like Prometheus, but for logs â€” labels, not full-text indexes.
2. **The labels:** like Prometheus labels â€” low cardinality only.
3. **LogQL â€” the query language:**
   ```logql
   {namespace="prod", app="myapp"} |= "error" | json | latency > 1000
   {namespace="prod"} |~ "panic|exception"
   ```
4. **The architecture:**
   ```
   Promtail / Vector / Fluent Bit
       â†“
   Loki distributor â†’ ingester â†’ chunk store (S3, GCS, filesystem)
                                â†’ querier â†’ Grafana
   ```
5. **Promtail â€” the agent (like Prometheus, but for logs):**
   ```yaml
   server: { http_listen_port: 9080 }
   positions: { filename: /tmp/positions.yaml }
   clients: [{ url: http://loki:3100/loki/api/v1/push }]
   scrape_configs:
     - job_name: kubernetes
       kubernetes_sd_configs: [{ role: pod }]
       relabel_configs: [...]
   ```
6. **The alternatives to Promtail:**
   - **Fluent Bit:** the CNCF choice
   - **Vector:** Rust, fast, transform-heavy
   - **Alloy:** Grafana's new agent (Grafana Agent â†’ Alloy)
7. **Hands-on exercise:** Install Loki + Promtail; stream container logs; query in Grafana.

#### Step 6: Tempo â€” The Traces Stack
1. **The design:** cheap, object-storage-backed, OpenTelemetry-compatible.
2. **The data model:** trace = a tree of spans, each with attributes.
3. **TraceQL â€” the query language:**
   ```
   { resource.service.name = "myapp" } | { span.http.status_code = 500 }
   ```
4. **The architecture:**
   ```
   OTel SDK â†’ OTel Collector â†’ Tempo distributor â†’ ingester â†’ S3/GCS
                                                            â†’ querier â†’ Grafana
   ```
5. **The linkage to other signals:**
   - **Exemplars in Prometheus:** click a metric, see the trace
   - **Logs with trace_id:** click a log, see the trace
   - **Grafana unified:** query all 3 signals from one panel
6. **Jaeger â€” the alternative:** the original OSS tracing system, also excellent.
7. **Hands-on exercise:** Install Tempo + OTel Collector; instrument an app; view traces in Grafana.

#### Step 7: The Full Grafana Stack
1. **The canonical install â€” Grafana Cloud or Grafana + OSS:**
   ```bash
   helm install grafana grafana/grafana -n monitoring
   helm install loki grafana/loki-stack -n monitoring
   helm install tempo grafana/tempo -n monitoring
   helm install prometheus prometheus-community/prometheus -n monitoring
   ```
2. **Or the bundled Helm chart:**
   ```bash
   helm install grafana grafana/grafana -f values.yaml
   # values.yaml configures datasources
   ```
3. **The correlation pattern â€” the "single pane of glass":**
   - A dashboard with: requests/sec (Prometheus), error rate (Prometheus), P99 latency (Prometheus), top errors (Loki), slowest trace (Tempo)
4. **Hands-on exercise:** Deploy the full stack; instrument a 3-service app; build a unified dashboard.

#### Step 8: The Open-Source Alternative â€” ELK / EFK
1. **The stack:**
   - **Elasticsearch:** the search/index store
   - **Logstash / Fluentd / Fluent Bit:** the ingest pipeline
   - **Kibana:** the visualization
2. **When to pick ELK over Loki:**
   - Full-text search is critical
   - You need complex aggregations on log content
   - You're already on Elastic for other reasons
3. **The lighter alternative â€” OpenSearch:**
   - The open-source fork after Elastic's license change
4. **Hands-on exercise:** Install the EFK stack; stream container logs; create a Kibana visualization.

#### Step 9: The SaaS Alternatives â€” Datadog, New Relic, Honeycomb, Dynatrace
1. **Datadog â€” the leading all-in-one:**
   - APM, infra, logs, RUM, synthetics, security, network, cost
   - 600+ integrations
   - The downside: $host/month stacks up fast
2. **New Relic â€” the user-based pricing model:**
   - All-in-one, free tier generous
   - NRQL query language
3. **Honeycomb â€” the observability 2.0:**
   - Event-based, high-cardinality-first
   - BubbleUp for debugging
4. **Dynatrace â€” the AI-driven one:**
   - Auto-instrumentation via OneAgent
   - Davis AI for anomaly detection
5. **Grafana Cloud â€” the open-source SaaS:**
   - Free tier: 10K metrics series, 50GB logs
   - Pro: usage-based
6. **The choice:**
   - **Budget-sensitive, K8s-heavy:** Grafana + Loki + Tempo
   - **All-in-one, $$$ no object:** Datadog or Dynatrace
   - **Event-based, debugging-first:** Honeycomb
7. **Hands-on exercise:** Sign up for Grafana Cloud free tier; ship a 3-service app's metrics/logs/traces there.

#### Step 10: SLIs, SLOs, Error Budgets â€” The Reliability Model
1. **The three definitions:**
   - **SLI (Service Level Indicator):** a measured value (e.g., 99.95% of requests return 200)
   - **SLO (Service Level Objective):** a target for the SLI (e.g., 99.9% over 30 days)
   - **Error budget:** 1 - SLO (e.g., 0.1% = 43.2 minutes/month of downtime)
2. **A typical SLO hierarchy:**
   - **User-facing:** 99.9% of API requests < 500ms
   - **Per-service:** 99.95% of order-service requests succeed
   - **Per-dependency:** 99.99% of DB queries < 50ms
3. **Burn rate alerting â€” the modern approach:**
   - **Page:** 1% of budget in 1h (very bad)
   - **Ticket:** 5% of budget in 24h
   - **Review:** 10% of budget in 7d
4. **The Google SRE workbook formulas:**
   - Page when burn > 14.4x for 1h (1% of 30d budget in 1h)
   - Page when burn > 6x for 6h
5. **SLO implementation in Grafana or Sloth:**
   ```yaml
   # sloth.sloth.yaml
   service: myapp
   labels: { team: payments }
   slos:
     - name: requests-success
       objective: 99.9
       description: "Successful HTTP requests"
       sli:
         events:
           error_query: sum(rate(http_requests_total{status=~"5.."}[{{.window}}]))
           total_query: sum(rate(http_requests_total[{{.window}}]))
   ```
6. **Hands-on exercise:** Define 3 SLOs for your app; create a burn-rate alert; create an SLO dashboard.

#### Step 11: Profiling, Events, and the New Signals
1. **Continuous profiling (Pyroscope, Parca, Datadog):**
   - Always-on CPU, memory, lock, allocation profiles
   - The 4th signal: "what code is running?"
2. **The Pyroscope + OTel combo:**
   ```python
   from pyroscope import configure
   configure(app_name="myapp", server_address="http://pyroscope:4040")
   ```
3. **Events â€” the metadata layer:**
   - Deploys, alerts, config changes, feature flags
   - Use as Grafana annotations: `POST /api/annotations`
4. **The unified observability picture:**
   - Metrics (Prometheus) â€” what
   - Logs (Loki) â€” why
   - Traces (Tempo) â€” where
   - Profiles (Pyroscope) â€” how (code-level)
   - Events â€” when
5. **Hands-on exercise:** Add Pyroscope to your app; see the function-level breakdown.

#### Step 12: AIOps & Anomaly Detection
1. **The options:**
   - **Grafana ML:** forecasting, anomaly detection in Grafana
   - **Prometheus + alertmanager + AlertBot:** simple
   - **Datadog Watchdog / New Relic AI / Dynatrace Davis:** SaaS AI
   - **Moogsoft, BigPanda, PagerDuty Event Intelligence:** aggregator
2. **Noise reduction â€” the 3 levers:**
   - **Alert routing:** right team, right severity
   - **Deduplication:** same root cause â†’ one ticket
   - **Auto-resolution:** close when fixed
3. **On-call hygiene:**
   - One page per incident
   - Sympathy pages: wait for human ack
   - Runbook URL in every alert
4. **Hands-on exercise:** Add Grafana ML anomaly detection on a noisy metric; set up auto-close.

#### Step 13: Observability for Cost (FinOps)
1. **The 3 pillars of FinOps:**
   - **Inform:** show teams what they spend
   - **Optimize:** rightsizing, spot, schedules
   - **Operate:** continuous, automated
2. **The open-source cost stack:**
   - **OpenCost:** the CNCF cost standard, Prometheus-native
   - **Kubecost:** the polished version
   - **Cloudability, Vantage:** SaaS
3. **The metrics to expose:**
   - `node_cpu_hourly_cost`
   - `pod_cpu_hourly_cost`
   - `namespace_daily_cost`
4. **Hands-on exercise:** Install OpenCost; build a per-namespace cost dashboard; identify the top 3 wastes.

#### Step 14: Production-Grade Observability Habits
- **The 3 signals (minimum):** metrics, logs, traces for every service
- **OpenTelemetry instrumentation:** don't vendor-lock-in
- **Low-cardinality labels only** in metrics and Loki
- **Structured JSON logs** with trace_id
- **Exemplars** to link metrics â†” traces
- **SLI/SLO + burn rate alerts** for every user-facing service
- **Grafana as the single pane of glass** â€” no dashboards in 5 places
- **Runbook URL in every alert**
- **Alert on symptoms, not causes** (page on error rate, not on CPU)
- **Profile in production** â€” continuous, low overhead
- **Cost observability** is part of observability
- **Practice incidents** â€” Game Days / DiRT

### Practice Project â€” Full Observability Platform

> **Goal:** Build a production-grade observability stack for a 3-service app, with SLOs, burn-rate alerts, profiles, cost, and incident drills.
> **Time estimate:** 15-20 hours
> **Deliverables:** A deployed stack, instrumented app, dashboards, alerts, and a Game Day runbook.

#### Phase 1 â€” Stack (2.5h)
- [ ] **(I)** Deploy Prometheus, Grafana, Loki, Tempo on a kind cluster
- [ ] **(I)** Install OTel Collector; configure OTLP receivers
- [ ] **(I)** Install Promtail (or Vector) for log shipping
- [ ] **(A)** Add Pyroscope for continuous profiling

#### Phase 2 â€” App Instrumentation (2h)
- [ ] **(I)** Instrument a 3-service app with OTel (traces, metrics, logs)
- [ ] **(I)** Auto-instrument the framework (Flask, Express, Spring)
- [ ] **(I)** Add structured JSON logs with trace_id
- [ ] **(A)** Add exemplars to Prometheus metrics

#### Phase 3 â€” Dashboards (2h)
- [ ] **(I)** Build a Golden Signals dashboard per service
- [ ] **(I)** Build a cluster overview dashboard
- [ ] **(I)** Add a service map from Tempo
- [ ] **(A)** Add Pyroscope panels for CPU + allocation

#### Phase 4 â€” SLOs (2h)
- [ ] **(I)** Define 3 SLOs: availability, latency, freshness
- [ ] **(I)** Set up Sloth or Grafana SLOs
- [ ] **(I)** Create burn-rate alerts (1h, 6h, 24h, 7d)
- [ ] **(A)** Implement the error budget freeze policy

#### Phase 5 â€” Alerts (1.5h)
- [ ] **(I)** Write 10 alert rules (4 symptoms, 3 saturation, 3 health)
- [ ] **(I)** Configure alert routes by severity
- [ ] **(I)** Integrate with Slack + PagerDuty (or email)
- [ ] **(A)** Add an alert noise dashboard (alerts/wk, MTTA, MTTR)

#### Phase 6 â€” Logs (1h)
- [ ] **(I)** Build 3 saved LogQL queries for top errors
- [ ] **(I)** Add a Loki dashboard for the service
- [ ] **(A)** Add log-based metrics for SLI derivation

#### Phase 7 â€” Cost (1.5h)
- [ ] **(I)** Install OpenCost; build a per-namespace dashboard
- [ ] **(I)** Identify the top 3 cost wastes
- [ ] **(A)** Add a cost anomaly alert

#### Phase 8 â€” Incident Drill (2h)
- [ ] **(I)** Write a Game Day scenario: "DB goes read-only"
- [ ] **(I)** Run it; time MTTD, MTTA, MTTR
- [ ] **(I)** Write the postmortem
- [ ] **(A)** Add an automated chaos experiment (Litmus or Chaos Mesh)

#### Phase 9 â€” Production Hardening (1.5h)
- [ ] **(I)** Configure long-term storage (Thanos/Mimir) for Prometheus
- [ ] **(I)** S3-backed Loki + Tempo
- [ ] **(I)** Set retention policies
- [ ] **(A)** Multi-cluster federation

#### Phase 10 â€” Documentation (1h)
- [ ] **(I)** `RUNBOOK.md` per service
- [ ] **(I)** `SLO.md` per service
- [ ] **(I)** `GAMEDAY.md` library
- [ ] **(A)** `OBSERVABILITY.md` architecture decision record

#### Bonus Challenges
- [ ] **(A)** Migrate from open-source to Grafana Cloud
- [ ] **(A)** Implement Datadog alongside OSS for comparison
- [ ] **(A)** Build a real-time log-to-trace link in Grafana

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 24 tasks across 10 phases + 3 bonus challenges**

### Key Takeaways
- The 3 signals (metrics, logs, traces) are the minimum; profiles and events are the future
- OpenTelemetry is the standard; instrument once, export anywhere
- Low-cardinality labels only â€” high-cardinality data goes in logs
- SLO + error budget + burn rate is the modern reliability model
- The open-source stack is mature; the SaaS is convenient; pick by team size + budget

---

## ðŸ” GitOps â€” ArgoCD, Flux & the Pull Model

**Difficulty:** Intermediate | **Time:** 15-20 hours

### Overview
GitOps is the practice of using Git as the single source of truth for declarative infrastructure and applications, with automated agents continuously reconciling the live state to the declared state. ArgoCD and Flux are the two leading implementations in Kubernetes. This tool covers the principles, both tools in depth, the patterns (App of Apps, Kustomize, Helm, secrets, multi-cluster), and the production-grade behaviors (RBAC, SSO, DR, Image Updater).

### Why Learn This
- GitOps is the deployment model for every modern K8s platform
- The "pull" model removes CI â†’ cluster credentials; safer by design
- ArgoCD and Flux are the two standards â€” knowing both makes you platform-flexible
- The patterns (multi-cluster, multi-tenant, multi-env) transfer across tools
- Progressive delivery (Argo Rollouts, Flagger) integrates naturally

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The 4 GitOps principles (OpenGitOps):**
   1. **Declarative** â€” the whole system is described declaratively
   2. **Versioned and immutable** â€” stored in Git, never edited live
   3. **Pulled automatically** â€” agents pull, not pushed
   4. **Continuously reconciled** â€” drift detected and corrected
2. **Push vs pull deployment:**
   ```
   Push (CI): CI â†’ kubectl apply   (CI has cluster credentials)
   Pull (GitOps): in-cluster agent reconciles from Git   (no cluster creds in CI)
   ```
3. **The repo layouts:**
   - **Application repo:** the app source code
   - **Config repo:** the K8s manifests, Helm values, Kustomize overlays
   - **Mono-repo:** both
4. **The tools:**
   | Tool | Strength |
   |------|----------|
   | **ArgoCD** | Best UI, multi-cluster, App of Apps, Rollouts |
   | **Flux** | GitOps Toolkit, modular, lighter, Helm Operator |
   | **Jenkins X** | Opinionated, wraps Tekton + Prow |
   | **Spinnaker** | Multi-cloud, mature, but heavy |
5. **Hands-on exercise:** Push a 3-service manifest to a Git repo; install ArgoCD; watch it deploy.

#### Step 2: ArgoCD â€” Install & First App
1. **Install:**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   # Or Helm
   helm repo add argo https://argoproj.github.io/argo-helm
   helm install argocd argo/argo-cd -n argocd --create-namespace
   ```
2. **Access the UI:**
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   # Get the initial password
   argocd admin initial-password -n argocd
   ```
3. **The CLI:**
   ```bash
   argocd login argocd.example.com --grpc
   argocd app create myapp \
     --repo https://github.com/myorg/config \
     --path apps/myapp/overlays/prod \
     --dest-server https://kubernetes.default.svc \
     --dest-namespace prod \
     --sync-policy automated \
     --self-heal \
     --prune
   argocd app list
   argocd app get myapp
   argocd app sync myapp
   argocd app history myapp
   ```
4. **The Application manifest:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata: { name: myapp, namespace: argocd }
   spec:
     project: default
     source:
       repoURL: https://github.com/myorg/config
       targetRevision: main
       path: apps/myapp/overlays/prod
     destination:
       server: https://kubernetes.default.svc
       namespace: prod
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
         - CreateNamespace=true
   ```
5. **Hands-on exercise:** Install ArgoCD; create your first Application from a public Git repo; sync; view in UI.

#### Step 3: ArgoCD â€” The Kustomize + Helm + Plain YAML Trinity
1. **Kustomize:**
   ```yaml
   spec:
     source:
       repoURL: https://github.com/myorg/config
       path: apps/myapp/overlays/prod
       # No plugin needed â€” Kustomize is built-in
   ```
2. **Helm:**
   ```yaml
   spec:
     source:
       repoURL: https://charts.bitnami.com/bitnami
       chart: nginx
       targetRevision: 15.x
       helm:
         valueFiles:
           - values-prod.yaml
         parameters:
           - name: service.type
             value: LoadBalancer
   ```
3. **Plain YAML (directory of manifests):**
   ```yaml
   spec:
     source:
       repoURL: https://github.com/myorg/config
       path: apps/myapp/base
   ```
4. **Hands-on exercise:** Create 3 Applications: Kustomize, Helm, plain YAML. Compare.

#### Step 4: ArgoCD â€” App of Apps & The Multi-Tenant Pattern
1. **The App of Apps pattern:**
   ```yaml
   # apps/platform.yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata: { name: platform }
   spec:
     source:
       repoURL: https://github.com/myorg/config
       path: apps/
       directory:
         recurse: true
         include: '{app1,app2,app3}.yaml'
   ```
2. **The ApplicationSet controller â€” for fan-out:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: ApplicationSet
   metadata: { name: cluster-apps }
   spec:
     generators:
       - list: { elements:
         - cluster: prod-east
           url: https://prod-east.example.com
         - cluster: prod-west
           url: https://prod-west.example.com }
     template:
       metadata: { name: '{{cluster}}-apps' }
       spec:
         project: default
         source:
           repoURL: https://github.com/myorg/config
           path: apps/overlays/{{cluster}}
         destination:
           server: '{{url}}'
           namespace: argocd
   ```
3. **The multi-tenant pattern:**
   - **AppProject** = boundary (RBAC, allowed repos, allowed clusters)
   - One project per team, with their own cluster or namespace
4. **A project:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: AppProject
   metadata: { name: payments, namespace: argocd }
   spec:
     description: Payments team
     sourceRepos:
       - https://github.com/myorg/payments-*
     destinations:
       - namespace: 'payments-*'
         server: https://kubernetes.default.svc
     clusterResourceWhitelist: []
     namespaceResourceWhitelist:
       - group: '*'
         kind: '*'
   ```
5. **Hands-on exercise:** Set up the App of Apps pattern; create 2 AppProjects; onboard a new app to each.

#### Step 5: ArgoCD â€” SSO, RBAC, and Security
1. **SSO with OIDC (Okta, Azure AD, Google):**
   ```yaml
   # argocd-cm.yaml
   url: https://argocd.example.com
   oidc.config: |
     name: Okta
     issuer: https://myorg.okta.com
     clientId: $oidc.clientId
     clientSecret: $oidc.clientSecret
     requestedScopes: ["openid", "profile", "email", "groups"]
   ```
2. **RBAC:**
   ```yaml
   # argocd-rbac-cm.yaml
   policy.csv: |
     p, role:devops, applications, *, */*, allow
     p, role:devops, clusters, get, https://kubernetes.default.svc, allow
     g, myorg:devops, role:devops
   ```
3. **Secrets management:**
   - **Bitnami Sealed Secrets** â€” encrypted secrets in Git
   - **External Secrets Operator** â€” sync from Vault / SM / KV
   - **SOPS + ArgoCD** â€” Mozilla SOPS for encrypted YAML
4. **Sync windows â€” when syncs are allowed:**
   ```yaml
   syncWindows:
     - kind: deny
       schedule: '0 0 * * 6'
       duration: 24h
       applications: ['*']
   ```
5. **Resource exclusions â€” what ArgoCD won't manage:**
   ```yaml
   ignoredResources:
     - groups: [""]
       kinds: ["ConfigMap"]
   ```
6. **Hands-on exercise:** Configure OIDC with Dex; set up 2 roles (admin, viewer); set up a sync window.

#### Step 6: ArgoCD â€” Notifications, Image Updater, Rollouts
1. **Notifications:**
   ```yaml
   # argocd-notifications-cm.yaml
   triggers:
     - name: on-deployed
       condition: app.status.operationState.phase in ['Succeeded']
       template: app-deployed
   subscriptions:
     - recipients: [slack:argocd]
       triggers: [on-deployed]
   ```
2. **Image Updater â€” auto-bump image tags from registry:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: ImageUpdater
   metadata: { name: myapp }
   spec:
     apps:
       - namePattern: myapp
         imageList:
           - ghcr.io/myorg/myapp
         updateStrategy: { strategy: Latest }
         git:
           repo: https://github.com/myorg/config
           branch: main
           writeBackMethod: argocd
   ```
3. **Argo Rollouts â€” progressive delivery:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Rollout
   metadata: { name: myapp }
   spec:
     replicas: 5
     strategy:
       canary:
         steps:
           - setWeight: 10
           - pause: { duration: 5m }
           - setWeight: 50
           - pause: { duration: 5m }
         canaryService: myapp
         stableService: myapp-stable
   ```
4. **Hands-on exercise:** Configure Slack notifications; set up Image Updater; deploy via Argo Rollouts.

#### Step 7: Flux â€” The Alternative
1. **Install the GitOps Toolkit:**
   ```bash
   curl -s https://fluxcd.io/install.sh | sudo bash
   flux check
   flux install
   ```
2. **The four controllers:**
   - **Source Controller:** Git, Helm, OCI, Bucket
   - **Kustomize Controller:** applies Kustomize
   - **Helm Controller:** reconciles Helm releases
   - **Notification Controller:** alerts
3. **A GitRepository + Kustomization:**
   ```yaml
   apiVersion: source.toolkit.fluxcd.io/v1
   kind: GitRepository
   metadata: { name: myorg, namespace: flux-system }
   spec:
     interval: 1m
     url: https://github.com/myorg/config
     ref:
       branch: main
   ---
   apiVersion: kustomize.toolkit.fluxcd.io/v1
   kind: Kustomization
   metadata: { name: myapp, namespace: flux-system }
   spec:
     interval: 10m
     sourceRef: { name: myorg }
     path: ./apps/myapp/overlays/prod
     prune: true
     wait: true
   ```
4. **The CLI:**
   ```bash
   flux get kustomizations
   flux reconcile kustomization myapp
   flux logs
   flux create source git myorg --url=https://github.com/myorg/config
   flux create kustomization myapp --source=myorg --path=./apps/myapp/overlays/prod
   ```
5. **The advantages of Flux:**
   - **Modular:** each controller is independent
   - **Light:** less RAM than ArgoCD
   - **CNCF Graduated:** more vendor-neutral
   - **Multi-tenancy:** better isolation story
6. **Hands-on exercise:** Install Flux; create a GitRepository + Kustomization; reconcile.

#### Step 8: Flux â€” Advanced Patterns
1. **HelmRelease:**
   ```yaml
   apiVersion: helm.toolkit.fluxcd.io/v2
   kind: HelmRelease
   metadata: { name: myapp, namespace: prod }
   spec:
     interval: 5m
     chart:
       spec:
         chart: myapp
         version: '1.x'
         sourceRef: { kind: HelmRepository, name: bitnami }
     values:
       replicaCount: 3
   ```
2. **Multi-tenancy with Tenants:**
   ```yaml
   apiVersion: tenancy.fluxcd.io/v1
   kind: Tenant
   metadata: { name: payments }
   spec:
     roleBindings:
       - clusterRoles: [edit]
         subjects:
           - kind: User
             name: payments-team
   ```
3. **Notifications (Alert â†’ Slack, GitHub status, etc.):**
   ```yaml
   apiVersion: notification.toolkit.fluxcd.io/v1beta1
   kind: Alert
   metadata: { name: on-call }
   spec:
     eventSeverity: error
     eventSources:
       - kind: Kustomization
         name: '*'
     providerRef: { name: slack }
   ```
4. **Image automation â€” Flux's Image Reflector + Policy + Update:**
   ```yaml
   apiVersion: image.toolkit.fluxcd.io/v1beta2
   kind: ImagePolicy
   metadata: { name: myapp }
   spec:
     imageRepositoryRef: { name: myapp }
     policy: { semver: { range: '1.2.x' } }
   ```
5. **Hands-on exercise:** Set up a HelmRelease, an ImagePolicy, a Slack alert.

#### Step 9: ArgoCD vs Flux â€” When to Use Which
1. **ArgoCD â€” the better choice when:**
   - You want a UI as the primary interface
   - You need multi-cluster management (ApplicationSet)
   - You want Argo Rollouts built-in
   - You have a single team / small platform team
2. **Flux â€” the better choice when:**
   - You want modular, composable controllers
   - You prefer pure GitOps (no UI)
   - You have strict multi-tenancy needs
   - You want minimal resource usage
   - You want CNCF-graduated neutrality
3. **The 2026 reality:** ArgoCD leads in market share, but both are excellent. Most teams pick one and standardize.
4. **Hands-on exercise:** Set up the same workload in both; compare UX, resource usage, multi-cluster.

#### Step 10: Production-Grade GitOps Habits
- **Git is the source of truth** â€” never `kubectl apply` from a laptop
- **One repo per cluster, or a monorepo with cluster directories** â€” pick deliberately
- **Apps as code, not as UI clicks** â€” version the Application manifests
- **AppProject (ArgoCD) or Tenant (Flux)** for multi-tenancy
- **Sealed Secrets / SOPS / ESO** for secrets in Git
- **Image Updater / Image Policy** for auto-bumps
- **Notifications** for every sync, fail, and drift
- **Sync windows** to prevent surprise deploys on Friday night
- **DR: backup the Git repo + the GitOps controller state** (Velero)
- **Audit log** â€” who deployed what when
- **Drift detection** â€” set `selfHeal: true` (ArgoCD) or `prune: true` (Flux)
- **Practice failures** â€” delete a manifest, see it get recreated; edit a live resource, see it corrected

### Practice Project â€” Production-Grade GitOps Platform

> **Goal:** Build a multi-cluster, multi-tenant GitOps platform with ArgoCD + Flux, Image Updater, secrets management, and SSO.
> **Time estimate:** 12-15 hours
> **Deliverables:** A working GitOps platform, 2 clusters, a config repo, and a runbook.

#### Phase 1 â€” Foundations (1.5h)
- [ ] **(I)** Stand up 2 kind clusters (dev, prod)
- [ ] **(I)** Create a config repo with overlays/dev and overlays/prod
- [ ] **(B)** Install ArgoCD on both clusters
- [ ] **(A)** Install Flux on one cluster and compare

#### Phase 2 â€” First Apps (2h)
- [ ] **(I)** Deploy 3 services via ArgoCD Application
- [ ] **(I)** Use Kustomize overlays
- [ ] **(I)** Use one Helm chart
- [ ] **(I)** Use App of Apps for the bundle

#### Phase 3 â€” Multi-Cluster (2h)
- [ ] **(I)** Add prod cluster to ArgoCD
- [ ] **(I)** Use ApplicationSet to fan out
- [ ] **(I)** Promote from dev to prod via PR
- [ ] **(A)** Add a third region

#### Phase 4 â€” Multi-Tenancy (1.5h)
- [ ] **(I)** Create 2 AppProjects (payments, search)
- [ ] **(I)** Restrict source repos + destinations per project
- [ ] **(A)** Use Flux Tenants for stricter isolation

#### Phase 5 â€” Security (2h)
- [ ] **(I)** Configure SSO via Dex
- [ ] **(I)** RBAC: 2 roles (admin, developer)
- [ ] **(I)** Sealed Secrets: encrypt a DB password
- [ ] **(A)** Use External Secrets Operator with Vault

#### Phase 6 â€” Automation (1.5h)
- [ ] **(I)** Image Updater: auto-bump on new image push
- [ ] **(I)** Notifications: Slack on sync, fail
- [ ] **(A)** Argo Rollouts: canary deploy with auto-rollback

#### Phase 7 â€” Production Hardening (1.5h)
- [ ] **(I)** Sync window: no prod deploys on weekends
- [ ] **(I)** Resource exclusions for ConfigMaps
- [ ] **(I)** DR: backup ArgoCD with Velero
- [ ] **(A)** HA ArgoCD (3 replicas, Redis HA)

#### Phase 8 â€” Documentation (1h)
- [ ] **(I)** README: how to onboard a new app
- [ ] **(I)** RUNBOOK: drift, failed sync, RBAC issue
- [ ] **(I)** `ARCHITECTURE.md`: clusters, repos, RBAC, secrets
- [ ] **(A)** `DISASTER.md`: lose Git, lose ArgoCD, lose cluster

#### Bonus Challenges
- [ ] **(A)** Add Flagger for automated canary + metrics rollback
- [ ] **(A)** Implement a self-service PR-based onboarding portal
- [ ] **(A)** Mirror to a second Git provider for DR

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 22 tasks across 8 phases + 3 bonus challenges**

### Key Takeaways
- GitOps = Git as the source of truth + automated reconciliation
- ArgoCD = UI + multi-cluster + Rollouts. Flux = modular + CNCF + multi-tenancy
- Pull model removes cluster credentials from CI; safer by default
- AppProject / Tenant for multi-tenancy, Sealed Secrets / ESO for secrets
- Drift detection, sync windows, and notifications are the production bar

---

## ðŸ•¸ï¸ Service Mesh â€” Istio, Linkerd, Consul Connect, Cilium

**Difficulty:** Advanced | **Time:** 20-25 hours

### Overview
A service mesh is a dedicated infrastructure layer for service-to-service communication. It handles mTLS, traffic management (canary, A/B, retries, circuit breaking), policy (authz, rate limiting), and observability (the 7 golden signals) without changing app code. Istio is the most popular (Envoy-based, Kubernetes-first). Linkerd is the lightweight alternative (Rust proxy, CNCF Graduated). Consul Connect is the multi-platform choice (HashiCorp). Cilium mesh is the eBPF sidecar-less future. This tool covers all four, the patterns they share, and the production trade-offs.

### Why Learn This
- Service mesh is the standard for zero-trust, mTLS, traffic control
- 50%+ of large K8s deployments run a mesh
- Knowing Istio + Linkerd covers 90% of the market
- The trade-offs (latency, complexity, control plane size) are real and worth understanding
- eBPF mesh (Cilium, Linkerd 2.x with eBPF) is the new direction

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The data plane vs the control plane:**
   - **Data plane:** the proxies (Envoy, linkerd2-proxy) that intercept every request
   - **Control plane:** the orchestrator (Istiod, linkerd-controller) that configures them
2. **The sidecar pattern â€” what mesh means:**
   ```
   Pod: [App container] [Envoy sidecar]
   Traffic: App â†’ localhost:15001 (Envoy) â†’ Network â†’ Pod: Envoy â†’ App
   ```
3. **What the mesh gives you:**
   - **mTLS** between every service, automatically
   - **Traffic splitting:** canary, A/B, blue/green
   - **Retries + circuit breaking** â€” without app code
   - **AuthZ policy:** L4 + L7
   - **Telemetry:** request rate, latency, errors
   - **Fault injection:** for testing
4. **The trade-offs:**
   - **Latency:** ~1-3ms per hop (with sidecars)
   - **Memory:** ~50-100MB per sidecar
   - **Complexity:** another control plane to operate
5. **The mesh vendors:**
   | Mesh | Proxy | Language | CNCF |
   |------|-------|----------|------|
   | Istio | Envoy | C++ | Graduated |
   | Linkerd | linkerd2-proxy | Rust | Graduated |
   | Consul Connect | Envoy or built-in | C++ | â€” |
   | Cilium | None (eBPF) | C | Graduated |
6. **Hands-on exercise:** Install Istio on a kind cluster; deploy 2 services; verify mTLS.

#### Step 2: Istio â€” Install & Tour
1. **Install with `istioctl`:**
   ```bash
   curl -L https://istio.io/downloadIstio | sh -
   cd istio-*
   export PATH=$PWD/bin:$PATH
   istioctl install --set profile=demo -y
   kubectl label namespace default istio-injection=enabled
   ```
2. **The deployment profiles:**
   | Profile | Use |
   |---------|-----|
   | `demo` | Lots of addons, learning |
   | `default` | Production-leaning |
   | `minimal` | Just the control plane |
   | `ambient` | Sidecar-less, eBPF |
3. **The addons:**
   ```bash
   kubectl apply -f samples/addons/prometheus.yaml
   kubectl apply -f samples/addons/grafana.yaml
   kubectl apply -f samples/addons/kiali.yaml
   kubectl apply -f samples/addons/jaeger.yaml
   ```
4. **Deploy an app:**
   ```bash
   kubectl apply -f samples/httpbin/httpbin.yaml
   kubectl apply -f samples/sleep/sleep.yaml
   kubectl exec -it sleep -- curl httpbin:8000
   ```
5. **Hands-on exercise:** Install Istio demo; deploy httpbin + sleep; view the service graph in Kiali.

#### Step 3: Istio â€” Traffic Management
1. **The 4 traffic primitives:**
   - **VirtualService:** routing rules (canary, mirror, fault)
   - **DestinationRule:** subset + load balancing + connection pool
   - **Gateway:** ingress/egress traffic
   - **ServiceEntry:** add external services to the mesh
2. **A canary deploy (90/10):**
   ```yaml
   apiVersion: networking.istio.io/v1
   kind: VirtualService
   metadata: { name: myapp }
   spec:
     hosts: [myapp]
     http:
       - route:
           - destination: { host: myapp, subset: v1 }
             weight: 90
           - destination: { host: myapp, subset: v2 }
             weight: 10
   ---
   apiVersion: networking.istio.io/v1
   kind: DestinationRule
   metadata: { name: myapp }
   spec:
     host: myapp
     subsets:
       - name: v1
         labels: { version: v1 }
       - name: v2
         labels: { version: v2 }
   ```
3. **Retries + circuit breaking:**
   ```yaml
   spec:
     trafficPolicy:
       connectionPool:
         tcp: { maxConnections: 100 }
         http: { h2UpgradePolicy: UPGRADE, maxRequestsPerConnection: 10 }
       outlierDetection:
         consecutive5xxErrors: 5
         interval: 30s
         baseEjectionTime: 30s
   ```
4. **Fault injection:**
   ```yaml
   http:
     - fault:
         delay: { percentage: { value: 10 }, fixedDelay: 5s }
         abort: { percentage: { value: 1 }, httpStatus: 500 }
       route: [...]
   ```
5. **The Gateway API (the modern way):**
   ```yaml
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   ```
6. **Hands-on exercise:** Deploy v1 and v2; do a 10% canary; inject 5s delay to v2; verify behavior.

#### Step 4: Istio â€” Security (mTLS, AuthZ)
1. **Strict mTLS cluster-wide:**
   ```yaml
   apiVersion: security.istio.io/v1
   kind: PeerAuthentication
   metadata: { name: default, namespace: istio-system }
   spec:
     mtls: { mode: STRICT }
   ```
2. **Per-namespace mTLS:**
   ```yaml
   spec:
     mtls: { mode: PERMISSIVE }   # allow upgrade
   ```
3. **AuthorizationPolicy â€” L4:**
   ```yaml
   apiVersion: security.istio.io/v1
   kind: AuthorizationPolicy
   metadata: { name: myapp, namespace: prod }
   spec:
     action: ALLOW
     rules:
       - from:
           - source: { principals: ["cluster.local/ns/prod/sa/api"] }
         to:
           - operation: { methods: ["GET"], paths: ["/api/*"] }
   ```
4. **L7 authz (with JWT):**
   ```yaml
   rules:
     - from:
         - source:
             requestPrincipals: ["*@myorg.com"]
   ```
5. **RequestAuthentication â€” JWT validation:**
   ```yaml
   apiVersion: security.istio.io/v1
   kind: RequestAuthentication
   metadata: { name: jwt }
   spec:
     jwtRules:
       - issuer: https://myorg.okta.com
         jwksUri: https://myorg.okta.com/.well-known/jwks.json
   ```
6. **Hands-on exercise:** Enable strict mTLS; add a JWT requirement; verify with a token and without.

#### Step 5: Istio â€” Observability
1. **Built-in metrics â€” the 4 golden signals per service:**
   - `istio_requests_total`
   - `istio_request_duration_milliseconds_bucket`
   - `istio_request_bytes_bucket`
   - `istio_response_bytes_bucket`
2. **Kiali â€” the service graph:**
   - Real-time visualization of services, traffic, errors
3. **Jaeger â€” distributed tracing:**
   - Trace propagation out of the box
4. **Access logs:**
   ```yaml
   apiVersion: telemetry.istio.io/v1
   kind: Telemetry
   metadata: { name: access-log }
   spec:
     accessLogging:
       - providers:
         - name: envoy
         filter:
           expression: "response.code >= 400"
   ```
5. **Hands-on exercise:** Add the access log filter; view the trace in Jaeger; create a Grafana dashboard.

#### Step 6: Istio â€” Ambient Mode (Sidecar-Less)
1. **The architecture:**
   ```
   Pod: [App] [ztunnel]   â† node-level proxy
   ```
   - **ztunnel:** per-node, L4, transparent
   - **Waypoint proxy:** per-namespace, L7, optional
2. **The benefits:**
   - No sidecar resource overhead
   - Faster startup, faster rollout
   - Simpler debugging
3. **Install:**
   ```bash
   istioctl install --set profile=ambient -y
   kubectl label namespace prod istio.io/dataplane-mode=ambient
   ```
4. **The maturity:** ambient is GA in Istio 1.22+; widely used in 2026.
5. **Hands-on exercise:** Migrate from sidecar to ambient; compare resource usage and startup time.

#### Step 7: Linkerd â€” The Lightweight Alternative
1. **Install:**
   ```bash
   curl -fsL https://run.linkerd.io/install | sh
   export PATH=$HOME/.linkerd2/bin:$PATH
   linkerd install --crds | kubectl apply -f -
   linkerd install | kubectl apply -f -
   linkerd viz install | kubectl apply -f -
   kubectl get -n linkerd deploy
   ```
2. **Inject the proxy:**
   ```bash
   kubectl get -n prod deploy -o yaml | linkerd inject - | kubectl apply -f -
   ```
3. **The Linkerd dashboard:**
   ```bash
   linkerd viz dashboard
   ```
4. **Service profiles â€” the Linkerd power feature:**
   ```yaml
   apiVersion: linkerd.io/v1alpha2
   kind: ServiceProfile
   metadata: { name: myapp.prod.svc.cluster.local, namespace: prod }
   spec:
     routes:
       - name: GetUsers
         condition: { method: GET, pathRegex: /users/.* }
         responseClasses:
           - condition: { status: { min: 500, max: 599 } }
             isFailure: true
   ```
5. **Retries + timeouts:**
   ```yaml
   spec:
     retryBudget:
       retryRatio: 0.2
       minRetriesPerSecond: 10
       ttl: 10s
   ```
6. **Traffic split:**
   ```yaml
   apiVersion: split.smi-spec.io/v1alpha1
   kind: TrafficSplit
   metadata: { name: myapp }
   spec:
     service: myapp
     backends:
       - service: myapp-v1
         weight: 900
       - service: myapp-v2
         weight: 100
   ```
7. **The pros vs Istio:**
   - Lighter (~10MB per proxy vs 50MB+ for Envoy)
   - Rust = safer, fewer CVEs
   - Simpler config (no YAML forest)
   - Smaller surface area = less to learn
8. **Hands-on exercise:** Install Linkerd; inject a deployment; view the live traffic in the dashboard.

#### Step 8: Consul Connect â€” The Multi-Platform Choice
1. **Install Consul on K8s:**
   ```bash
   helm repo add hashicorp https://helm.releases.hashicorp.com
   helm install consul hashicorp/consul --set global.enabled=true \
     --set connectInject.enabled=true --set server.replicas=3
   ```
2. **The two modes:**
   - **Transparent proxy:** sidecar intercepts
   - **Built-in:** built into Consul agent, no sidecar
3. **Service intentions â€” the authz:**
   ```bash
   consul config write - <<EOF
   Kind = "service-intentions"
   Name = "myapp"
   Sources = [
     { Name = "api", Permissions = [{ Action = "allow" }] }
   ]
   EOF
   ```
4. **The Federation â€” multi-cluster, multi-cloud:**
   - Primary cluster + secondary clusters
   - Mesh gateway connects them
5. **The pros:**
   - Works on K8s, VMs, bare metal â€” uniform
   - Service discovery is the primary purpose
   - HashiCorp stack integration (Vault, Nomad, Boundary)
6. **Hands-on exercise:** Install Consul; set up 2 intentions; verify a service can and cannot call another.

#### Step 9: Cilium Service Mesh â€” The eBPF Future
1. **The architecture:**
   - **Cilium agent:** per-node, eBPF programs
   - **No sidecar** â€” intercepts at the kernel
2. **L7 policy without Envoy:**
   ```yaml
   apiVersion: cilium.io/v2
   kind: CiliumNetworkPolicy
   metadata: { name: l7 }
   spec:
     endpointSelector: { matchLabels: { app: myapp } }
     ingress:
       - fromEndpoints: [{ matchLabels: { app: api } }]
         toPorts:
           - ports: [{ port: "80", protocol: TCP }]
             rules:
               http: [{ method: "GET", path: "/api/.*" }]
   ```
3. **The benefits:**
   - **No sidecar = no latency, no memory overhead**
   - Same eBPF for networking + policy
   - Hubble for observability
4. **The maturity:** L7 features grew significantly in 2024-2025; production-ready for many use cases.
5. **Hands-on exercise:** Install Cilium with the L7 proxy; write an L7 policy; verify with curl.

#### Step 10: The Mesh Decision & Production Patterns
1. **The decision matrix:**
   | Need | Pick |
   |------|------|
   | Most K8s, want ecosystem | Istio |
   | Light, simple, K8s-only | Linkerd |
   | Multi-platform, HashiCorp stack | Consul Connect |
   | Sidecar-less, eBPF-first | Cilium |
2. **Multi-mesh: don't run two in the same cluster.**
3. **The production hardening:**
   - **HA control plane:** 3 replicas, multiple AZs
   - **Resource limits on the proxies**
   - **Outbound traffic policy:** Registry only (deny by default)
   - **Strict mTLS cluster-wide**
   - **Authorization policies default-deny**
   - **Sidecar resource limits:** CPU + memory per pod
4. **The gotchas:**
   - **Sidecar ordering:** init containers must complete
   - **Pod lifecycle:** preStop hook to drain
   - **Debugging:** `istioctl proxy-config` / `linkerd diagnostics`
   - **Upgrades:** control plane + data plane together
5. **Hands-on exercise:** Pick the mesh that fits your use case; deploy a 3-service app with strict mTLS, canary, L7 authz, and dashboard.

### Practice Project â€” Production-Grade Service Mesh

> **Goal:** Build a multi-mesh evaluation (Istio + Linkerd + Cilium), deploy a 3-service app behind each, compare resource usage, latency, and features, then pick one and production-harden it.
> **Time estimate:** 15-20 hours
> **Deliverables:** Working meshes, comparison report, and production-hardened deploy.

#### Phase 1 â€” Setup (1.5h)
- [ ] **(I)** Stand up 3 kind clusters (mesh-istio, mesh-linkerd, mesh-cilium)
- [ ] **(I)** Deploy a 3-service app (web, api, db) on each
- [ ] **(B)** Run baseline benchmarks (latency, resource usage)

#### Phase 2 â€” Istio (3h)
- [ ] **(I)** Install Istio (sidecar mode)
- [ ] **(I)** Strict mTLS cluster-wide
- [ ] **(I)** Canary deploy with VirtualService
- [ ] **(I)** L7 AuthorizationPolicy + JWT
- [ ] **(A)** Migrate to ambient mode; compare resources

#### Phase 3 â€” Linkerd (2h)
- [ ] **(I)** Install Linkerd + viz
- [ ] **(I)** Inject proxies; view live traffic
- [ ] **(I)** Service profile + retries
- [ ] **(I)** Traffic split (canary)

#### Phase 4 â€” Cilium (2h)
- [ ] **(I)** Install Cilium; replace kube-proxy
- [ ] **(I)** L7 policy (no sidecar)
- [ ] **(I)** Hubble observability
- [ ] **(I)** Compare resource usage vs sidecar

#### Phase 5 â€” Comparison (1.5h)
- [ ] **(I)** Document latency, memory, CPU per mesh
- [ ] **(I)** Document feature coverage
- [ ] **(I)** Pick the best fit; justify
- [ ] **(A)** Run a chaos experiment in each

#### Phase 6 â€” Production Hardening (3h)
- [ ] **(I)** Outbound traffic policy: deny by default
- [ ] **(I)** Authorization policies: default deny
- [ ] **(I)** Resource limits on every sidecar
- [ ] **(I)** HA control plane (3 replicas, multi-AZ)
- [ ] **(A)** Multi-cluster mesh (Istio multi-primary or Linkerd multi-cluster)

#### Phase 7 â€” Observability (1h)
- [ ] **(I)** Dashboards: request rate, error rate, P99 latency
- [ ] **(I)** Distributed tracing from end to end
- [ ] **(I)** Alerts on mesh health

#### Phase 8 â€” DR (1h)
- [ ] **(I)** Back up the mesh state
- [ ] **(I)** Practice a control plane failure
- [ ] **(A)** Document the upgrade path

#### Phase 9 â€” Documentation (1h)
- [ ] **(I)** `MESH_DECISION.md` â€” the comparison
- [ ] **(I)** `RUNBOOK.md` â€” common issues
- [ ] **(A)** `ARCHITECTURE.md` â€” full mesh diagram

#### Bonus Challenges
- [ ] **(A)** Implement a custom Envoy filter in Istio
- [ ] **(A)** Build a multi-mesh gateway (Istio east-west)
- [ ] **(A)** Wire the mesh to Vault for cert management (or SPIFFE/SPIRE)

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 22 tasks across 9 phases + 3 bonus challenges**

### Key Takeaways
- A service mesh gives you mTLS, traffic control, authz, telemetry â€” without app code
- Istio = ecosystem. Linkerd = simplicity. Consul = multi-platform. Cilium = eBPF
- The trade-offs are real: sidecars add latency + memory; ambient and eBPF are the future
- Strict mTLS + default-deny AuthZ is the production bar
- Multi-cluster + multi-AZ is the HA pattern

---

## ðŸ” Secrets Management â€” Vault, ESO, Sealed Secrets, SOPS, SPIFFE

**Difficulty:** Intermediate | **Time:** 20-25 hours

### Overview
Secrets management is the discipline of storing, distributing, rotating, and auditing credentials, API keys, certificates, and tokens. In 2026, "store the secret in a YAML file in Git" is not acceptable â€” even with encryption. The production bar is: a dedicated secrets manager, dynamic secrets where possible, automatic rotation, short-lived credentials, and full audit. This tool covers HashiCorp Vault (the gold standard), External Secrets Operator (the K8s integration), Sealed Secrets (GitOps-friendly encryption), SOPS (file encryption), and SPIFFE/SPIRE (workload identity).

### Why Learn This
- Secrets in Git = a security incident waiting to happen
- Vault, ESO, and Sealed Secrets are the most common production tools
- SPIFFE/SPIRE is the future â€” workload identity replaces all static creds
- Dynamic secrets (Vault Database, AWS, GCP) eliminate the rotation problem
- Every compliance framework (SOC 2, PCI, HIPAA) requires secret management

### Learn by Doing â€” Step by Step

#### Step 1: The Mental Model
1. **The 4 categories of secrets:**
   - **Application secrets:** DB passwords, API keys
   - **Infrastructure secrets:** cloud IAM keys, SSH keys
   - **Workload identity:** SPIFFE IDs, JWTs for service-to-service
   - **PKI:** TLS certs, mTLS certs
2. **The lifecycle:**
   ```
   Generate â†’ Store â†’ Distribute â†’ Rotate â†’ Revoke â†’ Audit
   ```
3. **The 5 anti-patterns:**
   - **Secrets in Git** (even encrypted at rest)
   - **Long-lived API keys** (use OIDC / short-lived tokens)
   - **Same secret across envs** (use per-env credentials)
   - **Secrets in environment variables** (visible in process list)
   - **No rotation policy** (creds age forever)
4. **The 5 tools you'll use:**
   | Tool | Purpose |
   |------|---------|
   | HashiCorp Vault | The secrets manager (KV, DB, AWS, PKI, Transit) |
   | External Secrets Operator | Sync from Vault/SM/KV to K8s |
   | Sealed Secrets | Bitnami, encrypted in Git |
   | SOPS | Mozilla, file-level encryption |
   | SPIFFE/SPIRE | Workload identity |
5. **Hands-on exercise:** Identify the secrets in your current project; map them to a tool.

#### Step 2: HashiCorp Vault â€” The Gold Standard
1. **Install Vault dev mode:**
   ```bash
   vault server -dev -dev-root-token-id=root
   export VAULT_ADDR=http://127.0.0.1:8200
   export VAULT_TOKEN=root
   ```
2. **The 4 secrets engines you'll use most:**
   - **KV v2:** key-value, versioned
   - **Database:** dynamic Postgres/MySQL creds
   - **AWS:** dynamic IAM keys
   - **PKI:** dynamic TLS certs
3. **The 4 auth methods you'll use most:**
   - **Token:** the simple one
   - **Kubernetes:** Pods auth via SA
   - **AWS / GCP / Azure:** cloud IAM
   - **OIDC:** for users and CI
4. **KV v2 â€” write, read, version:**
   ```bash
   vault secrets enable -path=secret kv-v2
   vault kv put secret/myapp/db password="s3cret"
   vault kv get -version=2 secret/myapp/db
   vault kv metadata get secret/myapp/db
   ```
5. **Dynamic Postgres credentials:**
   ```bash
   vault secrets enable database
   vault write database/config/myapp \
     plugin_name=postgresql-database-plugin \
     allowed_roles="readonly" \
     connection_url="postgresql://{{username}}:{{password}}@db:5432/myapp?sslmode=disable" \
     username="vault" password="vault-root"
   vault write database/roles/readonly \
     db_name=myapp \
     creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
     default_ttl="1h" max_ttl="24h"
   vault read database/creds/readonly
   ```
6. **PKI â€” dynamic TLS:**
   ```bash
   vault secrets enable pki
   vault secrets tune -max-lease-ttl=87600h pki
   vault write -field=certificate pki/root/generate/internal \
     common_name=myorg.com ttl=87600h > ca_cert.crt
   vault write pki/config/urls issuing_certificates="$VAULT_ADDR/v1/pki/ca" \
     crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
   vault secrets enable -path=pki_int pki
   vault write -format=json pki_int/intermediate/generate/internal \
     common_name="myapp.myorg.com" ttl="43800h" | jq -r '.data.csr' > pki_intermediate.csr
   vault write pki/root/sign-intermediate csr=@pki_intermediate.csr \
     format=pem_bundle ttl="43800h"
   ```
7. **Hands-on exercise:** Run Vault dev; enable KV + Database; create a dynamic DB role; read creds; verify the DB user exists.

#### Step 3: Vault â€” Production Hardening
1. **HA â€” Integrated Raft storage:**
   ```bash
   vault operator init -key-shares=5 -key-threshold=3
   vault operator unseal <key1> <key2> <key3>
   ```
2. **Auto-unseal with AWS KMS / GCP CKMS / Azure Key Vault / Transit:**
   ```hcl
   seal "awskms" {
     region     = "us-east-1"
     kms_key_id = "alias/myorg-vault"
   }
   ```
3. **The policy â€” least privilege:**
   ```hcl
   path "secret/data/myapp/*" {
     capabilities = ["read", "list"]
   }
   path "database/creds/readonly" {
     capabilities = ["read"]
   }
   ```
4. **Audit logs:**
   ```bash
   vault audit enable file file_path=/var/log/vault_audit.log
   # Or to syslog
   vault audit enable syslog tag="vault" facility="AUTH"
   ```
5. **Namespaces â€” multi-tenancy in Vault Enterprise or OSS 1.16+:**
   ```bash
   vault namespace create payments
   ```
6. **Hands-on exercise:** Stand up a 3-node HA Vault with Raft; enable audit log; write a policy.

#### Step 4: Vault â€” Kubernetes Auth
1. **Enable K8s auth:**
   ```bash
   vault auth enable kubernetes
   vault write auth/kubernetes/config \
     kubernetes_host="https://kubernetes.default.svc"
   ```
2. **A role for a service account:**
   ```bash
   vault write auth/kubernetes/role/myapp \
     bound_service_account_names=myapp \
     bound_service_account_namespaces=prod \
     policies=myapp-read \
     audience=vault \
     ttl=1h
   ```
3. **The app authenticates:**
   ```bash
   vault write auth/kubernetes/login role=myapp jwt=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
   # Returns a Vault token
   vault kv get secret/myapp/db
   ```
4. **With Vault Agent Injector:**
   - Annotate the Pod
   - Vault Agent sidecar fetches and renews
5. **The injector annotation:**
   ```yaml
   metadata:
     annotations:
       vault.hashicorp.com/agent-inject: "true"
       vault.hashicorp.com/role: "myapp"
       vault.hashicorp.com/agent-inject-secret-DB_PASS: "secret/data/myapp/db#password"
   ```
6. **Hands-on exercise:** Enable K8s auth; create a role; deploy an app with the injector; verify secret is mounted.

#### Step 5: External Secrets Operator (ESO) â€” K8s-Native Integration
1. **Install ESO:**
   ```bash
   helm repo add external-secrets https://charts.external-secrets.io
   helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
   ```
2. **The 3 CRDs:**
   - **SecretStore:** where the secrets live (Vault, AWS SM, GSM, Azure KV)
   - **ExternalSecret:** which secret to fetch
   - **ClusterSecretStore:** cluster-wide SecretStore
3. **A SecretStore for Vault:**
   ```yaml
   apiVersion: external-secrets.io/v1beta1
   kind: SecretStore
   metadata: { name: vault, namespace: prod }
   spec:
     provider:
       vault:
         server: https://vault.internal:8200
         path: secret
         version: v2
         auth:
           kubernetes:
             mountPath: kubernetes
             role: myapp
             serviceAccountRef:
               name: myapp
   ```
4. **An ExternalSecret:**
   ```yaml
   apiVersion: external-secrets.io/v1beta1
   kind: ExternalSecret
   metadata: { name: myapp, namespace: prod }
   spec:
     refreshInterval: 1h
     secretStoreRef: { name: vault, kind: SecretStore }
     target:
       name: myapp
       creationPolicy: Owner
     data:
       - secretKey: db-password
         remoteRef: { key: secret/data/myapp/db, property: password }
   ```
5. **The result:** a K8s `Secret` named `myapp` with key `db-password`, refreshed every hour.
6. **The push vs pull refresh:**
   - Pull (default): ESO polls
   - Push (Vault + webhook): Vault pushes on change
7. **The alternatives to Vault:**
   - AWS Secrets Manager, GCP Secret Manager, Azure Key Vault
   - 1Password, Doppler, Akeyless (SaaS)
8. **Hands-on exercise:** Install ESO; create a SecretStore for Vault; sync a secret; watch the K8s Secret get created.

#### Step 6: Sealed Secrets â€” GitOps-Friendly Encryption
1. **The idea:** encrypt a Secret in such a way that only the Sealed Secrets controller in the target cluster can decrypt.
2. **Install the controller:**
   ```bash
   helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
   helm install sealed-secrets sealed-secrets/sealed-secrets -n kube-system
   ```
3. **Install `kubeseal`:**
   ```bash
   wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.27.0/kubeseal-0.27.0-linux-amd64.tar.gz
   tar -xzf kubeseal-*.tar.gz
   sudo install -m 755 kubeseal /usr/local/bin/
   ```
4. **Encrypt a Secret:**
   ```bash
   echo -n s3cret > password.txt
   kubectl create secret generic myapp --dry-run=client \
     --from-file=password=password.txt -o yaml > myapp-secret.yaml
   kubeseal --controller-name=sealed-secrets --controller-namespace=kube-system \
     -f myapp-secret.yaml -o yaml > myapp-sealed.yaml
   rm password.txt
   # myapp-sealed.yaml is safe to commit
   ```
5. **The controller decrypts on apply:**
   ```bash
   kubectl apply -f myapp-sealed.yaml
   kubectl get secret myapp -o yaml
   ```
6. **The 3 gotchas:**
   - **Key rotation:** old sealed secrets can break
   - **Scope:** cluster-wide or per-namespace
   - **Recovery:** back up the controller's private key
7. **Hands-on exercise:** Install Sealed Secrets; encrypt a Secret; commit to Git; apply from a fresh machine.

#### Step 7: SOPS â€” File-Level Encryption
1. **The idea:** encrypt specific values in a YAML/JSON/env file. The keys, structure stay in plaintext; only the values are encrypted.
2. **Install:**
   ```bash
   brew install sops
   # or
   sudo curl -L https://github.com/getsops/sops/releases/download/v3.9.0/sops-v3.9.0.linux.amd64 -o /usr/local/bin/sops
   sudo chmod +x /usr/local/bin/sops
   ```
3. **With Age (the modern key):**
   ```bash
   age-keygen -o key.age
   export SOPS_AGE_RECIPIENTS=$(age-keygen -y key.age)
   sops --encrypt --in-place secrets.yaml
   ```
4. **With AWS KMS:**
   ```bash
   export SOPS_KMS_ARN="arn:aws:kms:us-east-1:123:key/abc"
   sops --encrypt --kms $SOPS_KMS_ARN secrets.yaml
   ```
5. **A SOPS-encrypted file:**
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata: { name: myapp }
   stringData:
     password: ENC[AES256_GCM,data:abc...,tag:...,type:str]
   sops:
     kms: []
     age:
       - recipient: age1abc...
         enc: |
           -----BEGIN AGE ENCRYPTED FILE-----
           ...
   ```
6. **With Flux + SOPS â€” the production combo:**
   - Decryption happens at apply time
   - Use `sops-age` or `sops-kms` keys
7. **Hands-on exercise:** SOPS-encrypt a Secret with Age; commit to Git; decrypt with the right key.

#### Step 8: SPIFFE & SPIRE â€” Workload Identity
1. **The problem:** every workload needs an identity, and it can't be a static key.
2. **SPIFFE â€” the spec:**
   - **SVID:** SPIFFE Verifiable Identity Document (an X.509 cert or JWT)
   - **SPIFFE ID:** `spiffe://trust-domain/ns/foo/sa/bar`
3. **SPIRE â€” the implementation:**
   - **SPIRE Server:** the CA + control plane
   - **SPIRE Agent:** per-node, attests workloads, issues SVIDs
4. **The architecture:**
   ```
   SPIRE Server (control plane)
       â”œâ”€â”€ issues SVIDs to SPIRE Agents
       â””â”€â”€ SPIRE Agents (per node)
           â”œâ”€â”€ attest workloads (K8s SA, unix UID, etc.)
           â””â”€â”€ issue SVIDs to attested workloads
   ```
5. **The attestation:**
   - K8s: by ServiceAccount
   - Linux: by UID/GID
   - AWS: by IAM role
6. **The result:** every workload has a verifiable identity, automatic mTLS, no static keys.
7. **SPIRE on K8s:**
   ```bash
   helm repo add spiffe https://spiffe.github.io/helm-charts
   helm install spire spiffe/spire -n spire --create-namespace
   ```
8. **Hands-on exercise:** Install SPIRE on a kind cluster; deploy a workload with the SPIRE Agent; verify mTLS with SVIDs.

#### Step 9: The Certificate Stack â€” cert-manager, Let's Encrypt, PKI
1. **cert-manager â€” the K8s cert controller:**
   ```bash
   helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
   ```
2. **An Issuer (Let's Encrypt):**
   ```yaml
   apiVersion: cert-manager.io/v1
   kind: ClusterIssuer
   metadata: { name: letsencrypt-prod }
   spec:
     acme:
       server: https://acme-v02.api.letsencrypt.org/directory
       email: ops@myorg.com
       privateKeySecretRef: { name: letsencrypt-prod }
       solvers:
         - http01: { ingress: { class: nginx } }
   ```
3. **A Certificate:**
   ```yaml
   apiVersion: cert-manager.io/v1
   kind: Certificate
   metadata: { name: myapp }
   spec:
     secretName: myapp-tls
     issuerRef: { name: letsencrypt-prod, kind: ClusterIssuer }
     dnsNames: [app.example.com]
   ```
4. **The cert sources:**
   - **ACME:** Let's Encrypt, ZeroSSL
   - **Vault PKI:** internal CA
   - **Self-signed:** for dev
   - **External CA:** Venafi, DigiCert, EJBCA
5. **Hands-on exercise:** Install cert-manager; issue a cert from Let's Encrypt; use in an Ingress.

#### Step 10: Production-Grade Secrets Management Habits
- **Never put secrets in Git** â€” even encrypted
- **Use Vault / ESO / Sealed Secrets / SOPS** as appropriate
- **Dynamic secrets** for databases and cloud â€” auto-rotate
- **Short-lived credentials** â€” TTL of 1h, not 1y
- **Audit everything** â€” who read what when
- **Rotate on suspicion** â€” and on schedule
- **Back up the secrets manager** â€” Raft snapshots, K8s Secret backups
- **Test the restore** â€” quarterly
- **DR drill** â€” lose Vault, recover from backup, unseal with KMS
- **Compliance mapping** â€” SOC 2, PCI, HIPAA, ISO 27001
- **No SSH keys, no API keys** in 2026 â€” use workload identity (SPIFFE) or OIDC

### Practice Project â€” Production-Grade Secrets Platform

> **Goal:** Build a full secrets platform with Vault (HA, audit), ESO, Sealed Secrets, SOPS, cert-manager, and a workload identity story.
> **Time estimate:** 15-20 hours
> **Deliverables:** A working platform, an app using dynamic creds, signed certs, and a runbook.

#### Phase 1 â€” Vault Foundation (2.5h)
- [ ] **(I)** Stand up a 3-node Vault with Integrated Raft
- [ ] **(I)** Enable auto-unseal with AWS KMS
- [ ] **(I)** Enable audit logs (file + syslog)
- [ ] **(I)** Enable K8s auth
- [ ] **(A)** Set up a 2-cluster active-active with replication

#### Phase 2 â€” Dynamic Secrets (2.5h)
- [ ] **(I)** Enable Database secrets engine for Postgres
- [ ] **(I)** Create a role with 1h TTL
- [ ] **(I)** Read creds, verify the DB user, watch rotation
- [ ] **(I)** Enable AWS secrets engine for IAM keys
- [ ] **(A)** Enable PKI for internal mTLS

#### Phase 3 â€” K8s Integration (2h)
- [ ] **(I)** Install ESO; create a SecretStore for Vault
- [ ] **(I)** Create ExternalSecret for an app
- [ ] **(I)** Use the injector annotation
- [ ] **(A)** Use Vault Agent Auto-Auth with cloud IAM

#### Phase 4 â€” GitOps (1.5h)
- [ ] **(I)** Install Sealed Secrets; encrypt a Secret
- [ ] **(I)** Commit to Git; apply from fresh machine
- [ ] **(I)** SOPS-encrypt a YAML with Age
- [ ] **(I)** Flux with SOPS â€” decrypt at apply
- [ ] **(A)** ArgoCD with Sealed Secrets + ESO hybrid

#### Phase 5 â€” PKI (1.5h)
- [ ] **(I)** Install cert-manager
- [ ] **(I)** Issue a cert from Let's Encrypt
- [ ] **(I)** Use it in an Ingress
- [ ] **(A)** Set up Vault PKI as an internal CA

#### Phase 6 â€” Workload Identity (2h)
- [ ] **(I)** Install SPIRE on a kind cluster
- [ ] **(I)** Deploy a workload; verify SVID issuance
- [ ] **(I)** mTLS between two workloads with SVIDs
- [ ] **(A)** Integrate with Istio (SPIRE as the SDS)

#### Phase 7 â€” Operations (1.5h)
- [ ] **(I)** Audit log analysis: who read what
- [ ] **(I)** Rotate a secret, verify app picks up
- [ ] **(I)** DR: restore Vault from a Raft snapshot
- [ ] **(A)** Compromise drill: revoke all tokens, verify apps fail safely

#### Phase 8 â€” Documentation (1h)
- [ ] **(I)** `SECRETS.md` â€” what tool for what
- [ ] **(I)** `RUNBOOK.md` â€” rotate, restore, compromise
- [ ] **(A)** `COMPLIANCE.md` â€” SOC 2, PCI, HIPAA mapping
- [ ] **(A)** `DR.md` â€” quarterly drill results

#### Bonus Challenges
- [ ] **(A)** Build a self-service secrets portal on top of Vault
- [ ] **(A)** Integrate HashiCorp Boundary for just-in-time access
- [ ] **(A)** Multi-region Vault with performance replication

**Legend:** (B) = Beginner, (I) = Intermediate, (A) = Advanced
**Total: 24 tasks across 8 phases + 3 bonus challenges**

### Key Takeaways
- Secrets management is non-negotiable in 2026 â€” Vault, ESO, Sealed Secrets, SOPS, SPIFFE
- Dynamic secrets > static. Short-lived > long-lived. Workload identity > static keys
- Vault is the gold standard; ESO is the K8s glue; Sealed Secrets / SOPS for Git
- SPIFFE/SPIRE is the future â€” every workload gets an identity
- Audit + rotation + DR drill are the production bar

---
