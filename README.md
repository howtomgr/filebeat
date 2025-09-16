# filebeat Installation Guide

filebeat is a free and open-source log shipper. Filebeat provides lightweight shipper for forwarding logs

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 256MB minimum
  - Storage: 1GB for data
  - Network: Various outputs
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5066 (default filebeat port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install filebeat
sudo dnf install -y filebeat

# Enable and start service
sudo systemctl enable --now filebeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
sudo firewall-cmd --reload

# Verify installation
filebeat --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install filebeat
sudo apt install -y filebeat

# Enable and start service
sudo systemctl enable --now filebeat

# Configure firewall
sudo ufw allow 5066

# Verify installation
filebeat --version
```

### Arch Linux

```bash
# Install filebeat
sudo pacman -S filebeat

# Enable and start service
sudo systemctl enable --now filebeat

# Verify installation
filebeat --version
```

### Alpine Linux

```bash
# Install filebeat
apk add --no-cache filebeat

# Enable and start service
rc-update add filebeat default
rc-service filebeat start

# Verify installation
filebeat --version
```

### openSUSE/SLES

```bash
# Install filebeat
sudo zypper install -y filebeat

# Enable and start service
sudo systemctl enable --now filebeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
sudo firewall-cmd --reload

# Verify installation
filebeat --version
```

### macOS

```bash
# Using Homebrew
brew install filebeat

# Start service
brew services start filebeat

# Verify installation
filebeat --version
```

### FreeBSD

```bash
# Using pkg
pkg install filebeat

# Enable in rc.conf
echo 'filebeat_enable="YES"' >> /etc/rc.conf

# Start service
service filebeat start

# Verify installation
filebeat --version
```

### Windows

```bash
# Using Chocolatey
choco install filebeat

# Or using Scoop
scoop install filebeat

# Verify installation
filebeat --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/filebeat

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
filebeat --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable filebeat

# Start service
sudo systemctl start filebeat

# Stop service
sudo systemctl stop filebeat

# Restart service
sudo systemctl restart filebeat

# Check status
sudo systemctl status filebeat

# View logs
sudo journalctl -u filebeat -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add filebeat default

# Start service
rc-service filebeat start

# Stop service
rc-service filebeat stop

# Restart service
rc-service filebeat restart

# Check status
rc-service filebeat status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'filebeat_enable="YES"' >> /etc/rc.conf

# Start service
service filebeat start

# Stop service
service filebeat stop

# Restart service
service filebeat restart

# Check status
service filebeat status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start filebeat
brew services stop filebeat
brew services restart filebeat

# Check status
brew services list | grep filebeat
```

### Windows Service Manager

```powershell
# Start service
net start filebeat

# Stop service
net stop filebeat

# Using PowerShell
Start-Service filebeat
Stop-Service filebeat
Restart-Service filebeat

# Check status
Get-Service filebeat
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream filebeat_backend {
    server 127.0.0.1:5066;
}

server {
    listen 80;
    server_name filebeat.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name filebeat.example.com;

    ssl_certificate /etc/ssl/certs/filebeat.example.com.crt;
    ssl_certificate_key /etc/ssl/private/filebeat.example.com.key;

    location / {
        proxy_pass http://filebeat_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName filebeat.example.com
    Redirect permanent / https://filebeat.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName filebeat.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/filebeat.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/filebeat.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5066/
    ProxyPassReverse / http://127.0.0.1:5066/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend filebeat_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/filebeat.pem
    redirect scheme https if !{ ssl_fc }
    default_backend filebeat_backend

backend filebeat_backend
    balance roundrobin
    server filebeat1 127.0.0.1:5066 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R filebeat:filebeat /etc/filebeat
sudo chmod 750 /etc/filebeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status filebeat

# View logs
sudo journalctl -u filebeat -f

# Monitor resource usage
top -p $(pgrep filebeat)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/filebeat"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/filebeat-backup-$DATE.tar.gz" /etc/filebeat /var/lib/filebeat

echo "Backup completed: $BACKUP_DIR/filebeat-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop filebeat

# Restore from backup
tar -xzf /backup/filebeat/filebeat-backup-*.tar.gz -C /

# Start service
sudo systemctl start filebeat
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u filebeat -n 100
sudo tail -f /var/log/filebeat/filebeat.log

# Check configuration
filebeat --version

# Check permissions
ls -la /etc/filebeat
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5066

# Test connectivity
telnet localhost 5066

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep filebeat)

# Check disk I/O
iotop -p $(pgrep filebeat)

# Check connections
ss -an | grep 5066
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  filebeat:
    image: filebeat:latest
    ports:
      - "5066:5066"
    volumes:
      - ./config:/etc/filebeat
      - ./data:/var/lib/filebeat
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update filebeat

# Debian/Ubuntu
sudo apt update && sudo apt upgrade filebeat

# Arch Linux
sudo pacman -Syu filebeat

# Alpine Linux
apk update && apk upgrade filebeat

# openSUSE
sudo zypper update filebeat

# FreeBSD
pkg update && pkg upgrade filebeat

# Always backup before updates
tar -czf /backup/filebeat-pre-update-$(date +%Y%m%d).tar.gz /etc/filebeat

# Restart after updates
sudo systemctl restart filebeat
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/filebeat

# Clean old logs
find /var/log/filebeat -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/filebeat
```

## Additional Resources

- Official Documentation: https://docs.filebeat.org/
- GitHub Repository: https://github.com/filebeat/filebeat
- Community Forum: https://forum.filebeat.org/
- Best Practices Guide: https://docs.filebeat.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
