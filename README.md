# promtail Installation Guide

promtail is a free and open-source log collector. Promtail collects logs and sends them to Loki

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
  - Storage: 1GB for positions
  - Network: Push to Loki
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9080 (default promtail port)
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

# Install promtail
sudo dnf install -y promtail

# Enable and start service
sudo systemctl enable --now promtail

# Configure firewall
sudo firewall-cmd --permanent --add-port=9080/tcp
sudo firewall-cmd --reload

# Verify installation
promtail --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install promtail
sudo apt install -y promtail

# Enable and start service
sudo systemctl enable --now promtail

# Configure firewall
sudo ufw allow 9080

# Verify installation
promtail --version
```

### Arch Linux

```bash
# Install promtail
sudo pacman -S promtail

# Enable and start service
sudo systemctl enable --now promtail

# Verify installation
promtail --version
```

### Alpine Linux

```bash
# Install promtail
apk add --no-cache promtail

# Enable and start service
rc-update add promtail default
rc-service promtail start

# Verify installation
promtail --version
```

### openSUSE/SLES

```bash
# Install promtail
sudo zypper install -y promtail

# Enable and start service
sudo systemctl enable --now promtail

# Configure firewall
sudo firewall-cmd --permanent --add-port=9080/tcp
sudo firewall-cmd --reload

# Verify installation
promtail --version
```

### macOS

```bash
# Using Homebrew
brew install promtail

# Start service
brew services start promtail

# Verify installation
promtail --version
```

### FreeBSD

```bash
# Using pkg
pkg install promtail

# Enable in rc.conf
echo 'promtail_enable="YES"' >> /etc/rc.conf

# Start service
service promtail start

# Verify installation
promtail --version
```

### Windows

```bash
# Using Chocolatey
choco install promtail

# Or using Scoop
scoop install promtail

# Verify installation
promtail --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/promtail

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
promtail --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable promtail

# Start service
sudo systemctl start promtail

# Stop service
sudo systemctl stop promtail

# Restart service
sudo systemctl restart promtail

# Check status
sudo systemctl status promtail

# View logs
sudo journalctl -u promtail -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add promtail default

# Start service
rc-service promtail start

# Stop service
rc-service promtail stop

# Restart service
rc-service promtail restart

# Check status
rc-service promtail status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'promtail_enable="YES"' >> /etc/rc.conf

# Start service
service promtail start

# Stop service
service promtail stop

# Restart service
service promtail restart

# Check status
service promtail status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start promtail
brew services stop promtail
brew services restart promtail

# Check status
brew services list | grep promtail
```

### Windows Service Manager

```powershell
# Start service
net start promtail

# Stop service
net stop promtail

# Using PowerShell
Start-Service promtail
Stop-Service promtail
Restart-Service promtail

# Check status
Get-Service promtail
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream promtail_backend {
    server 127.0.0.1:9080;
}

server {
    listen 80;
    server_name promtail.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name promtail.example.com;

    ssl_certificate /etc/ssl/certs/promtail.example.com.crt;
    ssl_certificate_key /etc/ssl/private/promtail.example.com.key;

    location / {
        proxy_pass http://promtail_backend;
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
    ServerName promtail.example.com
    Redirect permanent / https://promtail.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName promtail.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/promtail.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/promtail.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9080/
    ProxyPassReverse / http://127.0.0.1:9080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend promtail_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/promtail.pem
    redirect scheme https if !{ ssl_fc }
    default_backend promtail_backend

backend promtail_backend
    balance roundrobin
    server promtail1 127.0.0.1:9080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R promtail:promtail /etc/promtail
sudo chmod 750 /etc/promtail

# Configure firewall
sudo firewall-cmd --permanent --add-port=9080/tcp
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
sudo systemctl status promtail

# View logs
sudo journalctl -u promtail -f

# Monitor resource usage
top -p $(pgrep promtail)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/promtail"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/promtail-backup-$DATE.tar.gz" /etc/promtail /var/lib/promtail

echo "Backup completed: $BACKUP_DIR/promtail-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop promtail

# Restore from backup
tar -xzf /backup/promtail/promtail-backup-*.tar.gz -C /

# Start service
sudo systemctl start promtail
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u promtail -n 100
sudo tail -f /var/log/promtail/promtail.log

# Check configuration
promtail --version

# Check permissions
ls -la /etc/promtail
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9080

# Test connectivity
telnet localhost 9080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep promtail)

# Check disk I/O
iotop -p $(pgrep promtail)

# Check connections
ss -an | grep 9080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  promtail:
    image: promtail:latest
    ports:
      - "9080:9080"
    volumes:
      - ./config:/etc/promtail
      - ./data:/var/lib/promtail
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update promtail

# Debian/Ubuntu
sudo apt update && sudo apt upgrade promtail

# Arch Linux
sudo pacman -Syu promtail

# Alpine Linux
apk update && apk upgrade promtail

# openSUSE
sudo zypper update promtail

# FreeBSD
pkg update && pkg upgrade promtail

# Always backup before updates
tar -czf /backup/promtail-pre-update-$(date +%Y%m%d).tar.gz /etc/promtail

# Restart after updates
sudo systemctl restart promtail
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/promtail

# Clean old logs
find /var/log/promtail -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/promtail
```

## Additional Resources

- Official Documentation: https://docs.promtail.org/
- GitHub Repository: https://github.com/promtail/promtail
- Community Forum: https://forum.promtail.org/
- Best Practices Guide: https://docs.promtail.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
