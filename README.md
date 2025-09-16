# che Installation Guide

che is a free and open-source developer workspace. Eclipse Che provides Kubernetes-native IDE and developer workspace

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
  - CPU: 4+ cores
  - RAM: 8GB minimum
  - Storage: 20GB for workspaces
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default che port)
  - Various services
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

# Install che
sudo dnf install -y che

# Enable and start service
sudo systemctl enable --now che

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
che --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install che
sudo apt install -y che

# Enable and start service
sudo systemctl enable --now che

# Configure firewall
sudo ufw allow 8080

# Verify installation
che --version
```

### Arch Linux

```bash
# Install che
sudo pacman -S che

# Enable and start service
sudo systemctl enable --now che

# Verify installation
che --version
```

### Alpine Linux

```bash
# Install che
apk add --no-cache che

# Enable and start service
rc-update add che default
rc-service che start

# Verify installation
che --version
```

### openSUSE/SLES

```bash
# Install che
sudo zypper install -y che

# Enable and start service
sudo systemctl enable --now che

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
che --version
```

### macOS

```bash
# Using Homebrew
brew install che

# Start service
brew services start che

# Verify installation
che --version
```

### FreeBSD

```bash
# Using pkg
pkg install che

# Enable in rc.conf
echo 'che_enable="YES"' >> /etc/rc.conf

# Start service
service che start

# Verify installation
che --version
```

### Windows

```bash
# Using Chocolatey
choco install che

# Or using Scoop
scoop install che

# Verify installation
che --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/che

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
che --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable che

# Start service
sudo systemctl start che

# Stop service
sudo systemctl stop che

# Restart service
sudo systemctl restart che

# Check status
sudo systemctl status che

# View logs
sudo journalctl -u che -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add che default

# Start service
rc-service che start

# Stop service
rc-service che stop

# Restart service
rc-service che restart

# Check status
rc-service che status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'che_enable="YES"' >> /etc/rc.conf

# Start service
service che start

# Stop service
service che stop

# Restart service
service che restart

# Check status
service che status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start che
brew services stop che
brew services restart che

# Check status
brew services list | grep che
```

### Windows Service Manager

```powershell
# Start service
net start che

# Stop service
net stop che

# Using PowerShell
Start-Service che
Stop-Service che
Restart-Service che

# Check status
Get-Service che
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream che_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name che.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name che.example.com;

    ssl_certificate /etc/ssl/certs/che.example.com.crt;
    ssl_certificate_key /etc/ssl/private/che.example.com.key;

    location / {
        proxy_pass http://che_backend;
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
    ServerName che.example.com
    Redirect permanent / https://che.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName che.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/che.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/che.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend che_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/che.pem
    redirect scheme https if !{ ssl_fc }
    default_backend che_backend

backend che_backend
    balance roundrobin
    server che1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R che:che /etc/che
sudo chmod 750 /etc/che

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status che

# View logs
sudo journalctl -u che -f

# Monitor resource usage
top -p $(pgrep che)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/che"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/che-backup-$DATE.tar.gz" /etc/che /var/lib/che

echo "Backup completed: $BACKUP_DIR/che-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop che

# Restore from backup
tar -xzf /backup/che/che-backup-*.tar.gz -C /

# Start service
sudo systemctl start che
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u che -n 100
sudo tail -f /var/log/che/che.log

# Check configuration
che --version

# Check permissions
ls -la /etc/che
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep che)

# Check disk I/O
iotop -p $(pgrep che)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  che:
    image: che:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/che
      - ./data:/var/lib/che
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update che

# Debian/Ubuntu
sudo apt update && sudo apt upgrade che

# Arch Linux
sudo pacman -Syu che

# Alpine Linux
apk update && apk upgrade che

# openSUSE
sudo zypper update che

# FreeBSD
pkg update && pkg upgrade che

# Always backup before updates
tar -czf /backup/che-pre-update-$(date +%Y%m%d).tar.gz /etc/che

# Restart after updates
sudo systemctl restart che
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/che

# Clean old logs
find /var/log/che -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/che
```

## Additional Resources

- Official Documentation: https://docs.che.org/
- GitHub Repository: https://github.com/che/che
- Community Forum: https://forum.che.org/
- Best Practices Guide: https://docs.che.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
