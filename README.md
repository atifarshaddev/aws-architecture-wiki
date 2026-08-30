# aws-cloud-runbook
# ☁️ Enterprise Cloud Infrastructure & DevOps Runbook

This repository serves as a production-grade operational runbook and engineering reference manual. It documents standard operating procedures for Linux systems administration, SSH cryptographic connections, network engineering triage, and multi-service AWS CLI orchestration.

---

## 💻 1. SSH Cryptographic Connections & Session Management
*Protocols executed on local control machines (macOS/Linux) to establish secure remote terminal sessions with cloud infrastructure.*

### 🔑 Private Key Security Policy
Before initializing an SSH handshake, POSIX file permissions on your downloaded private key (`.pem`) must be restricted. If the file is readable by other local system users, the SSH client will reject the connection by default.
```bash
# Restrict permissions: Only the root owner can read the key file (Secure)
chmod 400 your-key-name.pem

# Verify file permissions flags match (-r--------)
ls -l your-key-name.pem
```

### 🌉 Boundary Crossings (Handshake Mechanics)
```bash
# Target standard Amazon Linux 2 / Amazon Linux 2023 environments
ssh -i your-key-name.pem ec2-user@<YOUR-EC2-PUBLIC-IP>

# Target Ubuntu Server environments (Default non-root superuser changes)
ssh -i your-key-name.pem ubuntu@<YOUR-EC2-PUBLIC-IP>

# Pass custom port if SSH daemon has been hard-hardened away from port 22
ssh -i your-key-name.pem ec2-user@<YOUR-EC2-PUBLIC-IP> -p 2222
```

---

## 🐧 2. Comprehensive Linux Systems Administration
*Core operating system engineering commands executed directly INSIDE the EC2 instance shell via SSH.*

### 📦 System Maintenance & Package Updates
```bash
# Update local package manager indices and patch OS security updates
sudo yum update -y                      # RHEL / Amazon Linux Enterprise
sudo apt update && sudo apt upgrade -y  # Debian / Ubuntu Engine Architecture
```

### 📂 Advanced Filesystem & Navigation Operations
```bash
# Print current absolute working directory pathway
pwd

# List directory contents with extended details (permissions, sizes, timestamps, hidden files)
ls -laht

# Create nested directory trees recursively in one block
mkdir -p /var/www/html/assets/images

# Copy files and directories recursively across pathways
cp -r /source/directory/ /destination/pathway/

# Forcefully remove directory architectures recursively (Use with extreme caution)
rm -rf /tmp/stale-cache-directory
```

### ✍️ File Editing & Stream Manipulation
```bash
# Open or create a file in the minimalist command-line text editor
nano /etc/nginx/nginx.conf

# Output entire contents of a file directly to the terminal stdout stream
cat /var/log/nginx/access.log

# Stream the last 50 entries of a log file and follow updates in real-time
tail -n 50 -f /var/log/httpd/error_log

# Filter and extract specific error strings from dense log outputs
grep -i "500 Internal Server Error" /var/log/nginx/error.log
```

### 🛠️ Linux Permission Matrix & Identity Control
```bash
# View your active shell runtime identity context
whoami

# Change owner and group of a target file or folder architecture
sudo chown -R nginx:nginx /var/www/html/

# Modify file read/write/execute binary bitmasks
chmod 755 /var/www/html/index.html   # Owner: RWX, Group: RX, Public: RX
```

### 📊 Real-Time Performance & Resource Triage
```bash
# View a comprehensive interactive dashboard of CPU, Memory, and Active Process Threads
top

# Audit physical storage allocation across all mounted hard drive storage arrays
df -h

# Check volatile system RAM usage in human-readable Megabytes/Gigabytes
free -m

# Systematically terminate an unresponsive or runaway background application process
sudo kill -9 <PROCESS-ID-NUMBER>
```

---

## 🏎️ 3. AWS CLI (Command Line Interface) Infrastructure Orchestration
*Global API control commands executed from an authenticated shell terminal to manipulate raw AWS cloud infrastructure layer objects around instances.*

### 🔐 Multi-Profile Context Engine
```bash
# Trigger interactive primary authentication setup loop
aws configure

# Validate current active IAM caller identity context and account numbers
aws sts get-caller-identity
```

### 🖥️ Elastic Compute Cloud (EC2) Control Plane
```bash
# Inspect all instances running across the target region using custom filtering JSON flags
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{ID:InstanceId,State:State.Name,IP:PublicIpAddress}' --output table

# Gracefully request hypervisor to cycle power states
aws ec2 start-instances --instance-ids <INSTANCE-ID-STRING>
aws ec2 stop-instances --instance-ids <INSTANCE-ID-STRING>

# Permanently purge, de-allocate, and destroy an infrastructure asset boundary
aws ec2 terminate-instances --instance-ids <INSTANCE-ID-STRING>
```

### 🪣 Simple Storage Service (S3) Objects Engineering
```bash
# Audit account workspace to list all globally unique cloud buckets
aws s3 ls

# Allocate and lock down a new global object storage namespace bucket
aws s3 mb s3://enterprise-data-repository-lake

# Upload local static objects to the cloud bucket using parallel processing streams
aws s3 cp local-dataset.tar.gz s3://enterprise-data-repository-lake/backups/

# Sync entire directories to an S3 backup path while deleting vanished source items
aws s3 sync /var/www/html/ s3://enterprise-data-repository-lake/production-site/ --delete
```

---

## 🔍 4. Hardened Networking Diagnostics & Triage
*Network core utility operations used inside terminal environments to trace communication loops, resolve DNS path faults, and test security firewalls.*

```bash
# Send ICMP echo packets to test raw network layer layer connectivity limits
ping -c 4 8.8.8.8

# Run absolute lookup traces on domain DNS names against global name server maps
nslookup your-digital-domain.com

# Trace network hop route pathways down physical network routers across the internet backbones
traceroute <YOUR-EC2-PUBLIC-IP>

# Query the network sockets layer to audit active listening ports inside your server
sudo netstat -tunlp

# Test application firewall rules by requesting headers directly from specific web ports
# (Validates if AWS Security Group is successfully passing traffic on Port 80 / 443)
curl -I http://<YOUR-EC2-PUBLIC-IP>:80
```
