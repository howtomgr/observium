# observium Installation Guide

observium is a free and open-source network monitoring platform. Observium provides autodiscovering network monitoring platform

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
  - RAM: 2GB minimum
  - Storage: 10GB for data
  - Network: SNMP/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default observium port)
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

# Install observium
sudo dnf install -y observium

# Enable and start service
sudo systemctl enable --now observium

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
observium --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install observium
sudo apt install -y observium

# Enable and start service
sudo systemctl enable --now observium

# Configure firewall
sudo ufw allow 80

# Verify installation
observium --version
```

### Arch Linux

```bash
# Install observium
sudo pacman -S observium

# Enable and start service
sudo systemctl enable --now observium

# Verify installation
observium --version
```

### Alpine Linux

```bash
# Install observium
apk add --no-cache observium

# Enable and start service
rc-update add observium default
rc-service observium start

# Verify installation
observium --version
```

### openSUSE/SLES

```bash
# Install observium
sudo zypper install -y observium

# Enable and start service
sudo systemctl enable --now observium

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
observium --version
```

### macOS

```bash
# Using Homebrew
brew install observium

# Start service
brew services start observium

# Verify installation
observium --version
```

### FreeBSD

```bash
# Using pkg
pkg install observium

# Enable in rc.conf
echo 'observium_enable="YES"' >> /etc/rc.conf

# Start service
service observium start

# Verify installation
observium --version
```

### Windows

```bash
# Using Chocolatey
choco install observium

# Or using Scoop
scoop install observium

# Verify installation
observium --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/observium

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
observium --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable observium

# Start service
sudo systemctl start observium

# Stop service
sudo systemctl stop observium

# Restart service
sudo systemctl restart observium

# Check status
sudo systemctl status observium

# View logs
sudo journalctl -u observium -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add observium default

# Start service
rc-service observium start

# Stop service
rc-service observium stop

# Restart service
rc-service observium restart

# Check status
rc-service observium status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'observium_enable="YES"' >> /etc/rc.conf

# Start service
service observium start

# Stop service
service observium stop

# Restart service
service observium restart

# Check status
service observium status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start observium
brew services stop observium
brew services restart observium

# Check status
brew services list | grep observium
```

### Windows Service Manager

```powershell
# Start service
net start observium

# Stop service
net stop observium

# Using PowerShell
Start-Service observium
Stop-Service observium
Restart-Service observium

# Check status
Get-Service observium
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream observium_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name observium.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name observium.example.com;

    ssl_certificate /etc/ssl/certs/observium.example.com.crt;
    ssl_certificate_key /etc/ssl/private/observium.example.com.key;

    location / {
        proxy_pass http://observium_backend;
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
    ServerName observium.example.com
    Redirect permanent / https://observium.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName observium.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/observium.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/observium.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend observium_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/observium.pem
    redirect scheme https if !{ ssl_fc }
    default_backend observium_backend

backend observium_backend
    balance roundrobin
    server observium1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R observium:observium /etc/observium
sudo chmod 750 /etc/observium

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status observium

# View logs
sudo journalctl -u observium -f

# Monitor resource usage
top -p $(pgrep observium)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/observium"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/observium-backup-$DATE.tar.gz" /etc/observium /var/lib/observium

echo "Backup completed: $BACKUP_DIR/observium-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop observium

# Restore from backup
tar -xzf /backup/observium/observium-backup-*.tar.gz -C /

# Start service
sudo systemctl start observium
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u observium -n 100
sudo tail -f /var/log/observium/observium.log

# Check configuration
observium --version

# Check permissions
ls -la /etc/observium
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep observium)

# Check disk I/O
iotop -p $(pgrep observium)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  observium:
    image: observium:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/observium
      - ./data:/var/lib/observium
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update observium

# Debian/Ubuntu
sudo apt update && sudo apt upgrade observium

# Arch Linux
sudo pacman -Syu observium

# Alpine Linux
apk update && apk upgrade observium

# openSUSE
sudo zypper update observium

# FreeBSD
pkg update && pkg upgrade observium

# Always backup before updates
tar -czf /backup/observium-pre-update-$(date +%Y%m%d).tar.gz /etc/observium

# Restart after updates
sudo systemctl restart observium
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/observium

# Clean old logs
find /var/log/observium -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/observium
```

## Additional Resources

- Official Documentation: https://docs.observium.org/
- GitHub Repository: https://github.com/observium/observium
- Community Forum: https://forum.observium.org/
- Best Practices Guide: https://docs.observium.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
