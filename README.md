# stellar-core Installation Guide

stellar-core is a free and open-source Stellar node. Stellar Core provides backbone node for Stellar network

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
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 100GB for history
  - Network: Stellar protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 11625 (default stellar-core port)
  - HTTP on 11626
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

# Install stellar-core
sudo dnf install -y stellar_core

# Enable and start service
sudo systemctl enable --now stellar-core

# Configure firewall
sudo firewall-cmd --permanent --add-port=11625/tcp
sudo firewall-cmd --reload

# Verify installation
stellar-core --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install stellar-core
sudo apt install -y stellar_core

# Enable and start service
sudo systemctl enable --now stellar-core

# Configure firewall
sudo ufw allow 11625

# Verify installation
stellar-core --version
```

### Arch Linux

```bash
# Install stellar-core
sudo pacman -S stellar_core

# Enable and start service
sudo systemctl enable --now stellar-core

# Verify installation
stellar-core --version
```

### Alpine Linux

```bash
# Install stellar-core
apk add --no-cache stellar_core

# Enable and start service
rc-update add stellar-core default
rc-service stellar-core start

# Verify installation
stellar-core --version
```

### openSUSE/SLES

```bash
# Install stellar-core
sudo zypper install -y stellar_core

# Enable and start service
sudo systemctl enable --now stellar-core

# Configure firewall
sudo firewall-cmd --permanent --add-port=11625/tcp
sudo firewall-cmd --reload

# Verify installation
stellar-core --version
```

### macOS

```bash
# Using Homebrew
brew install stellar_core

# Start service
brew services start stellar_core

# Verify installation
stellar-core --version
```

### FreeBSD

```bash
# Using pkg
pkg install stellar_core

# Enable in rc.conf
echo 'stellar-core_enable="YES"' >> /etc/rc.conf

# Start service
service stellar-core start

# Verify installation
stellar-core --version
```

### Windows

```bash
# Using Chocolatey
choco install stellar_core

# Or using Scoop
scoop install stellar_core

# Verify installation
stellar-core --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/stellar_core

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
stellar-core --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable stellar-core

# Start service
sudo systemctl start stellar-core

# Stop service
sudo systemctl stop stellar-core

# Restart service
sudo systemctl restart stellar-core

# Check status
sudo systemctl status stellar-core

# View logs
sudo journalctl -u stellar-core -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add stellar-core default

# Start service
rc-service stellar-core start

# Stop service
rc-service stellar-core stop

# Restart service
rc-service stellar-core restart

# Check status
rc-service stellar-core status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'stellar-core_enable="YES"' >> /etc/rc.conf

# Start service
service stellar-core start

# Stop service
service stellar-core stop

# Restart service
service stellar-core restart

# Check status
service stellar-core status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start stellar_core
brew services stop stellar_core
brew services restart stellar_core

# Check status
brew services list | grep stellar_core
```

### Windows Service Manager

```powershell
# Start service
net start stellar-core

# Stop service
net stop stellar-core

# Using PowerShell
Start-Service stellar-core
Stop-Service stellar-core
Restart-Service stellar-core

# Check status
Get-Service stellar-core
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream stellar_core_backend {
    server 127.0.0.1:11625;
}

server {
    listen 80;
    server_name stellar_core.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name stellar_core.example.com;

    ssl_certificate /etc/ssl/certs/stellar_core.example.com.crt;
    ssl_certificate_key /etc/ssl/private/stellar_core.example.com.key;

    location / {
        proxy_pass http://stellar_core_backend;
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
    ServerName stellar_core.example.com
    Redirect permanent / https://stellar_core.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName stellar_core.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/stellar_core.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/stellar_core.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:11625/
    ProxyPassReverse / http://127.0.0.1:11625/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend stellar_core_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/stellar_core.pem
    redirect scheme https if !{ ssl_fc }
    default_backend stellar_core_backend

backend stellar_core_backend
    balance roundrobin
    server stellar_core1 127.0.0.1:11625 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R stellar_core:stellar_core /etc/stellar_core
sudo chmod 750 /etc/stellar_core

# Configure firewall
sudo firewall-cmd --permanent --add-port=11625/tcp
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
sudo systemctl status stellar-core

# View logs
sudo journalctl -u stellar-core -f

# Monitor resource usage
top -p $(pgrep stellar_core)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/stellar_core"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/stellar_core-backup-$DATE.tar.gz" /etc/stellar_core /var/lib/stellar_core

echo "Backup completed: $BACKUP_DIR/stellar_core-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop stellar-core

# Restore from backup
tar -xzf /backup/stellar_core/stellar_core-backup-*.tar.gz -C /

# Start service
sudo systemctl start stellar-core
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u stellar-core -n 100
sudo tail -f /var/log/stellar_core/stellar_core.log

# Check configuration
stellar-core --version

# Check permissions
ls -la /etc/stellar_core
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 11625

# Test connectivity
telnet localhost 11625

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep stellar_core)

# Check disk I/O
iotop -p $(pgrep stellar_core)

# Check connections
ss -an | grep 11625
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  stellar_core:
    image: stellar_core:latest
    ports:
      - "11625:11625"
    volumes:
      - ./config:/etc/stellar_core
      - ./data:/var/lib/stellar_core
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update stellar_core

# Debian/Ubuntu
sudo apt update && sudo apt upgrade stellar_core

# Arch Linux
sudo pacman -Syu stellar_core

# Alpine Linux
apk update && apk upgrade stellar_core

# openSUSE
sudo zypper update stellar_core

# FreeBSD
pkg update && pkg upgrade stellar_core

# Always backup before updates
tar -czf /backup/stellar_core-pre-update-$(date +%Y%m%d).tar.gz /etc/stellar_core

# Restart after updates
sudo systemctl restart stellar-core
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/stellar_core

# Clean old logs
find /var/log/stellar_core -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/stellar_core
```

## Additional Resources

- Official Documentation: https://docs.stellar_core.org/
- GitHub Repository: https://github.com/stellar_core/stellar_core
- Community Forum: https://forum.stellar_core.org/
- Best Practices Guide: https://docs.stellar_core.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
