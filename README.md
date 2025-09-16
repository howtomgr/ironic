# ironic Installation Guide

ironic is a free and open-source bare metal provisioning. Ironic provides bare metal provisioning for OpenStack

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
  - Storage: 50GB for images
  - Network: HTTP/IPMI
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 6385 (default ironic port)
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

# Install ironic
sudo dnf install -y ironic

# Enable and start service
sudo systemctl enable --now ironic

# Configure firewall
sudo firewall-cmd --permanent --add-port=6385/tcp
sudo firewall-cmd --reload

# Verify installation
ironic --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ironic
sudo apt install -y ironic

# Enable and start service
sudo systemctl enable --now ironic

# Configure firewall
sudo ufw allow 6385

# Verify installation
ironic --version
```

### Arch Linux

```bash
# Install ironic
sudo pacman -S ironic

# Enable and start service
sudo systemctl enable --now ironic

# Verify installation
ironic --version
```

### Alpine Linux

```bash
# Install ironic
apk add --no-cache ironic

# Enable and start service
rc-update add ironic default
rc-service ironic start

# Verify installation
ironic --version
```

### openSUSE/SLES

```bash
# Install ironic
sudo zypper install -y ironic

# Enable and start service
sudo systemctl enable --now ironic

# Configure firewall
sudo firewall-cmd --permanent --add-port=6385/tcp
sudo firewall-cmd --reload

# Verify installation
ironic --version
```

### macOS

```bash
# Using Homebrew
brew install ironic

# Start service
brew services start ironic

# Verify installation
ironic --version
```

### FreeBSD

```bash
# Using pkg
pkg install ironic

# Enable in rc.conf
echo 'ironic_enable="YES"' >> /etc/rc.conf

# Start service
service ironic start

# Verify installation
ironic --version
```

### Windows

```bash
# Using Chocolatey
choco install ironic

# Or using Scoop
scoop install ironic

# Verify installation
ironic --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ironic

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ironic --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ironic

# Start service
sudo systemctl start ironic

# Stop service
sudo systemctl stop ironic

# Restart service
sudo systemctl restart ironic

# Check status
sudo systemctl status ironic

# View logs
sudo journalctl -u ironic -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ironic default

# Start service
rc-service ironic start

# Stop service
rc-service ironic stop

# Restart service
rc-service ironic restart

# Check status
rc-service ironic status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ironic_enable="YES"' >> /etc/rc.conf

# Start service
service ironic start

# Stop service
service ironic stop

# Restart service
service ironic restart

# Check status
service ironic status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ironic
brew services stop ironic
brew services restart ironic

# Check status
brew services list | grep ironic
```

### Windows Service Manager

```powershell
# Start service
net start ironic

# Stop service
net stop ironic

# Using PowerShell
Start-Service ironic
Stop-Service ironic
Restart-Service ironic

# Check status
Get-Service ironic
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ironic_backend {
    server 127.0.0.1:6385;
}

server {
    listen 80;
    server_name ironic.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ironic.example.com;

    ssl_certificate /etc/ssl/certs/ironic.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ironic.example.com.key;

    location / {
        proxy_pass http://ironic_backend;
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
    ServerName ironic.example.com
    Redirect permanent / https://ironic.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ironic.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ironic.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ironic.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:6385/
    ProxyPassReverse / http://127.0.0.1:6385/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ironic_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ironic.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ironic_backend

backend ironic_backend
    balance roundrobin
    server ironic1 127.0.0.1:6385 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ironic:ironic /etc/ironic
sudo chmod 750 /etc/ironic

# Configure firewall
sudo firewall-cmd --permanent --add-port=6385/tcp
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
sudo systemctl status ironic

# View logs
sudo journalctl -u ironic -f

# Monitor resource usage
top -p $(pgrep ironic)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ironic"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ironic-backup-$DATE.tar.gz" /etc/ironic /var/lib/ironic

echo "Backup completed: $BACKUP_DIR/ironic-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ironic

# Restore from backup
tar -xzf /backup/ironic/ironic-backup-*.tar.gz -C /

# Start service
sudo systemctl start ironic
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ironic -n 100
sudo tail -f /var/log/ironic/ironic.log

# Check configuration
ironic --version

# Check permissions
ls -la /etc/ironic
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 6385

# Test connectivity
telnet localhost 6385

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ironic)

# Check disk I/O
iotop -p $(pgrep ironic)

# Check connections
ss -an | grep 6385
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ironic:
    image: ironic:latest
    ports:
      - "6385:6385"
    volumes:
      - ./config:/etc/ironic
      - ./data:/var/lib/ironic
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ironic

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ironic

# Arch Linux
sudo pacman -Syu ironic

# Alpine Linux
apk update && apk upgrade ironic

# openSUSE
sudo zypper update ironic

# FreeBSD
pkg update && pkg upgrade ironic

# Always backup before updates
tar -czf /backup/ironic-pre-update-$(date +%Y%m%d).tar.gz /etc/ironic

# Restart after updates
sudo systemctl restart ironic
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ironic

# Clean old logs
find /var/log/ironic -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ironic
```

## Additional Resources

- Official Documentation: https://docs.ironic.org/
- GitHub Repository: https://github.com/ironic/ironic
- Community Forum: https://forum.ironic.org/
- Best Practices Guide: https://docs.ironic.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
