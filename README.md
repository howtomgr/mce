# MCE (Media Center Enterprise) Installation Guide

MCE (Media Center Enterprise) is a comprehensive media server solution that combines multiple FOSS tools to create a complete home entertainment system. It typically includes components like Plex or Jellyfin for media streaming, Sonarr/Radarr for content management, and various download clients. This guide focuses on setting up a complete media center stack as a FOSS alternative to commercial solutions like Kaleidescape or Control4.

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

### System Requirements:
- CPU: Minimum 2 cores (4+ recommended for transcoding)
- RAM: 4GB minimum (8GB+ recommended)
- Storage: 100GB+ for media library
- Network: Gigabit Ethernet recommended

### Dependencies:
- Python 3.x
- Docker or native packages
- Media codecs (ffmpeg)

### Required Packages by OS:
- **RHEL/CentOS/Rocky/AlmaLinux**: epel-release, python3, ffmpeg
- **Debian/Ubuntu**: python3, python3-pip, ffmpeg, mediainfo
- **Arch Linux**: python, ffmpeg, mediainfo
- **Alpine Linux**: python3, ffmpeg, mediainfo
- **openSUSE/SLES**: python3, ffmpeg, mediainfo
- **macOS**: python3, ffmpeg (via Homebrew)
- **FreeBSD**: python3, ffmpeg, mediainfo
- **Windows**: Python 3.x, ffmpeg

## 2. Supported Operating Systems

MCE media center stack is supported on:
- RHEL 8/9 and derivatives (CentOS, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later)
- FreeBSD 13+
- Windows 10/11/Server 2019+

## 3. Installation

### RHEL/CentOS/Rocky/AlmaLinux

```bash
# Install EPEL repository
sudo dnf install -y epel-release

# Install dependencies
sudo dnf install -y python3 python3-pip ffmpeg mediainfo git

# Create media directories
sudo mkdir -p /opt/mce/{config,data,media,downloads}

# Install Jellyfin (FOSS alternative to Plex)
sudo dnf install -y https://repo.jellyfin.org/releases/server/centos/stable/server/jellyfin-server-*.rpm
sudo dnf install -y jellyfin-web

# Install Sonarr
cat << EOF | sudo tee /etc/yum.repos.d/sonarr.repo
[sonarr]
name=sonarr
baseurl=https://download.sonarr.tv/v3/main/3.0.10.1567/
gpgcheck=1
gpgkey=https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2009837CBFFD68F45BC180471F4F90DE2A9B4BF8
enabled=1
EOF

sudo dnf install -y sonarr

# Install Radarr
# Download latest release
wget -O /tmp/Radarr.linux.tar.gz https://github.com/Radarr/Radarr/releases/download/v4.5.2.7318/Radarr.linux-core-x64.tar.gz
sudo tar -xf /tmp/Radarr.linux.tar.gz -C /opt/
sudo mv /opt/Radarr /opt/radarr

# Install Prowlarr (indexer manager)
wget -O /tmp/Prowlarr.linux.tar.gz https://github.com/Prowlarr/Prowlarr/releases/download/v1.8.6.3946/Prowlarr.linux-core-x64.tar.gz
sudo tar -xf /tmp/Prowlarr.linux.tar.gz -C /opt/
sudo mv /opt/Prowlarr /opt/prowlarr

# Install qBittorrent (FOSS torrent client)
sudo dnf install -y qbittorrent-nox
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install dependencies
sudo apt install -y python3 python3-pip ffmpeg mediainfo git curl

# Create media directories
sudo mkdir -p /opt/mce/{config,data,media,downloads}

# Install Jellyfin
sudo apt install -y apt-transport-https
curl -fsSL https://repo.jellyfin.org/ubuntu/jellyfin_team.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/jellyfin.gpg
echo "deb [arch=$( dpkg --print-architecture ) signed-by=/etc/apt/keyrings/jellyfin.gpg] https://repo.jellyfin.org/$( awk -F'=' '/^ID=/{ print $NF }' /etc/os-release ) $( awk -F'=' '/^VERSION_CODENAME=/{ print $NF }' /etc/os-release ) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list
sudo apt update
sudo apt install -y jellyfin

# Install Sonarr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 2009837CBFFD68F45BC180471F4F90DE2A9B4BF8
echo "deb https://apt.sonarr.tv/debian buster main" | sudo tee /etc/apt/sources.list.d/sonarr.list
sudo apt update
sudo apt install -y sonarr

# Install Radarr
wget -O /tmp/Radarr.linux.tar.gz https://github.com/Radarr/Radarr/releases/download/v4.5.2.7318/Radarr.linux-core-x64.tar.gz
sudo tar -xf /tmp/Radarr.linux.tar.gz -C /opt/
sudo mv /opt/Radarr /opt/radarr

# Install Prowlarr
wget -O /tmp/Prowlarr.linux.tar.gz https://github.com/Prowlarr/Prowlarr/releases/download/v1.8.6.3946/Prowlarr.linux-core-x64.tar.gz
sudo tar -xf /tmp/Prowlarr.linux.tar.gz -C /opt/
sudo mv /opt/Prowlarr /opt/prowlarr

# Install qBittorrent
sudo apt install -y qbittorrent-nox
```

### Arch Linux

```bash
# Install dependencies
sudo pacman -S --needed python ffmpeg mediainfo git

# Create media directories
sudo mkdir -p /opt/mce/{config,data,media,downloads}

# Install from AUR
yay -S jellyfin-bin sonarr radarr prowlarr qbittorrent-nox
```

### Alpine Linux

```bash
# Install dependencies
apk add --no-cache python3 py3-pip ffmpeg mediainfo git

# Create media directories
mkdir -p /opt/mce/{config,data,media,downloads}

# Install Jellyfin
apk add --no-cache jellyfin jellyfin-web

# Manual installation for other components
# (Download and extract as shown in RHEL section)
```

### openSUSE/SLES

```bash
# Install dependencies
sudo zypper install -y python3 python3-pip ffmpeg mediainfo git

# Create media directories
sudo mkdir -p /opt/mce/{config,data,media,downloads}

# Add Packman repository for multimedia codecs
sudo zypper ar -cfp 90 https://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Tumbleweed/ packman
sudo zypper refresh

# Install components
sudo zypper install -y jellyfin qbittorrent-nox

# Manual installation for Sonarr/Radarr/Prowlarr
# (Follow RHEL manual installation steps)
```

### macOS

```bash
# Install Homebrew if not present
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install python@3 ffmpeg mediainfo

# Create directories
mkdir -p ~/MCE/{config,data,media,downloads}

# Install services
brew install --cask jellyfin
brew install sonarr
brew install radarr

# qBittorrent
brew install --cask qbittorrent
```

### FreeBSD

```bash
# Install dependencies
pkg install -y python3 ffmpeg mediainfo git

# Create media directories
mkdir -p /usr/local/etc/mce/{config,data,media,downloads}

# Install from ports or packages
pkg install -y jellyfin sonarr radarr qbittorrent-nox
```

### Windows

```powershell
# Install Chocolatey package manager
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install dependencies
choco install -y python3 ffmpeg mediainfo git

# Create directories
New-Item -ItemType Directory -Force -Path C:\MCE\config
New-Item -ItemType Directory -Force -Path C:\MCE\data
New-Item -ItemType Directory -Force -Path C:\MCE\media
New-Item -ItemType Directory -Force -Path C:\MCE\downloads

# Install services
choco install -y jellyfin sonarr radarr prowlarr qbittorrent
```

## 4. Configuration

### Initial Setup

1. **Jellyfin Configuration** (Media Server):
   ```bash
   # Access web interface
   http://localhost:8096
   
   # Configure:
   # - Admin user
   # - Media libraries (point to /opt/mce/media)
   # - Transcoding settings
   # - Remote access
   ```

2. **Sonarr Configuration** (TV Show Manager):
   ```bash
   # Access web interface
   http://localhost:8989
   
   # Configure:
   # - Indexers (via Prowlarr)
   # - Download clients (qBittorrent)
   # - Media management
   # - Quality profiles
   ```

3. **Radarr Configuration** (Movie Manager):
   ```bash
   # Access web interface
   http://localhost:7878
   
   # Configure:
   # - Indexers (via Prowlarr)
   # - Download clients
   # - Media management
   # - Quality profiles
   ```

4. **Prowlarr Configuration** (Indexer Manager):
   ```bash
   # Access web interface
   http://localhost:9696
   
   # Configure:
   # - Indexers
   # - Apps (Sonarr/Radarr sync)
   # - Download clients
   ```

5. **qBittorrent Configuration**:
   ```bash
   # Default web interface
   http://localhost:8080
   # Default credentials: admin/adminadmin
   
   # Configure:
   # - Download paths
   # - Connection limits
   # - Web UI settings
   ```

### Environment Variables

Create `/etc/mce/mce.conf`:
```bash
# Media directories
MCE_MEDIA_ROOT=/opt/mce/media
MCE_DOWNLOADS=/opt/mce/downloads
MCE_CONFIG=/opt/mce/config

# Service ports
JELLYFIN_PORT=8096
SONARR_PORT=8989
RADARR_PORT=7878
PROWLARR_PORT=9696
QBITTORRENT_PORT=8080

# API Keys (generated during setup)
SONARR_API_KEY=your_key_here
RADARR_API_KEY=your_key_here
PROWLARR_API_KEY=your_key_here
```

## 5. Service Management

### systemd (Linux)

Create service files for each component:

**Jellyfin** (usually auto-created):
```bash
sudo systemctl enable --now jellyfin
sudo systemctl status jellyfin
```

**Sonarr Service** (`/etc/systemd/system/sonarr.service`):
```ini
[Unit]
Description=Sonarr Daemon
After=network.target

[Service]
Type=simple
User=sonarr
Group=media
ExecStart=/usr/bin/mono --debug /opt/sonarr/Sonarr.exe -nobrowser
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Radarr Service** (`/etc/systemd/system/radarr.service`):
```ini
[Unit]
Description=Radarr Daemon
After=network.target

[Service]
Type=simple
User=radarr
Group=media
ExecStart=/opt/radarr/Radarr -nobrowser
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable services:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now sonarr radarr prowlarr qbittorrent-nox
```

### OpenRC (Alpine Linux)

Create init scripts in `/etc/init.d/` for each service.

### rc.d (FreeBSD)

Enable in `/etc/rc.conf`:
```bash
jellyfin_enable="YES"
sonarr_enable="YES"
radarr_enable="YES"
```

### launchd (macOS)

Services are managed automatically when installed via Homebrew.

### Windows Services

Services are automatically created during installation.

## 6. Troubleshooting

### Common Issues

1. **Permission Errors**:
   ```bash
   # Fix ownership
   sudo chown -R media:media /opt/mce
   sudo chmod -R 755 /opt/mce
   ```

2. **Port Conflicts**:
   ```bash
   # Check port usage
   sudo netstat -tlnp | grep -E "(8096|8989|7878|9696|8080)"
   ```

3. **Database Issues**:
   ```bash
   # Backup and recreate database
   cp /opt/mce/config/sonarr/sonarr.db /opt/mce/config/sonarr/sonarr.db.bak
   ```

### Log Files

- Jellyfin: `/var/log/jellyfin/`
- Sonarr: `/opt/mce/config/sonarr/logs/`
- Radarr: `/opt/mce/config/radarr/logs/`
- qBittorrent: `~/.local/share/qBittorrent/logs/`

## 7. Security Considerations

### Network Security

1. **Firewall Rules**:
   ```bash
   # Allow only necessary ports
   sudo firewall-cmd --permanent --add-port=8096/tcp  # Jellyfin
   sudo firewall-cmd --permanent --add-port=8989/tcp  # Sonarr (local only)
   sudo firewall-cmd --permanent --add-port=7878/tcp  # Radarr (local only)
   sudo firewall-cmd --reload
   ```

2. **Reverse Proxy** (nginx example):
   ```nginx
   server {
       listen 443 ssl;
       server_name media.example.com;
       
       ssl_certificate /etc/ssl/certs/media.crt;
       ssl_certificate_key /etc/ssl/private/media.key;
       
       location / {
           proxy_pass http://localhost:8096;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```

### Authentication

1. Enable authentication on all services
2. Use strong passwords
3. Enable 2FA where available
4. Restrict API access

### File Permissions

```bash
# Create dedicated user
sudo useradd -r -s /bin/false media

# Set permissions
sudo chown -R media:media /opt/mce
sudo chmod -R 750 /opt/mce/config
sudo chmod -R 755 /opt/mce/media
```

## 8. Performance Tuning

### Jellyfin Optimization

1. **Hardware Acceleration**:
   ```bash
   # Intel Quick Sync
   sudo usermod -aG video jellyfin
   
   # NVIDIA
   sudo apt install -y nvidia-docker2
   ```

2. **Transcoding Settings**:
   - Enable hardware transcoding
   - Set transcode directory to fast storage (SSD)
   - Limit simultaneous streams

### Storage Optimization

1. **File System**:
   ```bash
   # Use XFS or ext4 for media storage
   sudo mkfs.xfs /dev/sdb1
   
   # Mount with optimizations
   echo "/dev/sdb1 /opt/mce/media xfs defaults,noatime 0 2" | sudo tee -a /etc/fstab
   ```

2. **Cache Settings**:
   - Configure read-ahead cache
   - Use SSD for metadata

### Network Optimization

```bash
# Increase network buffers
echo "net.core.rmem_max = 134217728" | sudo tee -a /etc/sysctl.conf
echo "net.core.wmem_max = 134217728" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 9. Backup and Restore

### Backup Strategy

1. **Configuration Backup**:
   ```bash
   #!/bin/bash
   # backup-mce.sh
   BACKUP_DIR="/backup/mce/$(date +%Y%m%d)"
   mkdir -p "$BACKUP_DIR"
   
   # Backup configurations
   tar -czf "$BACKUP_DIR/configs.tar.gz" /opt/mce/config/
   
   # Backup databases
   for service in jellyfin sonarr radarr; do
       cp /opt/mce/config/$service/*.db "$BACKUP_DIR/"
   done
   ```

2. **Media Library Backup**:
   ```bash
   # Use rsync for incremental backups
   rsync -av --progress /opt/mce/media/ /backup/media/
   ```

### Restore Procedure

```bash
# Stop services
sudo systemctl stop jellyfin sonarr radarr prowlarr

# Restore configurations
tar -xzf /backup/mce/20240115/configs.tar.gz -C /

# Restore databases
cp /backup/mce/20240115/*.db /opt/mce/config/

# Start services
sudo systemctl start jellyfin sonarr radarr prowlarr
```

## 10. System Requirements

### Minimum Requirements
- **CPU**: 2 cores @ 2.0GHz
- **RAM**: 4GB
- **Storage**: 100GB for OS and applications
- **Network**: 100Mbps

### Recommended Requirements
- **CPU**: 4+ cores @ 3.0GHz (with Quick Sync or GPU for transcoding)
- **RAM**: 8GB+
- **Storage**: 
  - 250GB SSD for OS and applications
  - 4TB+ HDD for media storage
- **Network**: 1Gbps

### Storage Calculation
- TV Shows: ~2-5GB per hour (1080p)
- Movies: ~5-15GB per movie (1080p)
- 4K Content: ~25-50GB per movie

## 11. Support

- **Documentation**: https://github.com/howtomgr/mce
- **Jellyfin**: https://jellyfin.org/docs/
- **Sonarr**: https://wiki.servarr.com/sonarr
- **Radarr**: https://wiki.servarr.com/radarr
- **Community Forums**: 
  - r/jellyfin
  - r/sonarr
  - r/radarr

## 12. Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 13. License

This guide is provided under the MIT License. Individual components have their own licenses:
- Jellyfin: GPL-2.0
- Sonarr: GPL-3.0
- Radarr: GPL-3.0
- qBittorrent: GPL-2.0+

## 14. Acknowledgments

- The Jellyfin team for the excellent media server
- The Servarr team for Sonarr/Radarr/Prowlarr
- The open-source community

## 15. Version History

- v1.0.0 (2024-01-15): Initial SPEC 2.0 compliant guide
- Migrated from multi-tool installation to comprehensive MCE stack

## 16. Appendices

### A. API Integration

Example Python script for Sonarr API:
```python
#!/usr/bin/env python3
import requests

SONARR_URL = "http://localhost:8989"
API_KEY = "your_api_key_here"

headers = {"X-Api-Key": API_KEY}

# Get all series
response = requests.get(f"{SONARR_URL}/api/v3/series", headers=headers)
series = response.json()

for show in series:
    print(f"{show['title']} - {show['status']}")
```

### B. Mobile Access

Configure external access:
1. Use a reverse proxy (nginx/Caddy)
2. Set up SSL certificates (Let's Encrypt)
3. Configure dynamic DNS if needed
4. Install mobile apps:
   - Jellyfin mobile app
   - nzb360 (Android) for Sonarr/Radarr
   - Lunasea (iOS/Android)

### C. Hardware Recommendations

**Budget Build** (~$500):
- Intel Pentium Gold G6400
- 8GB DDR4 RAM
- 120GB SSD + 2TB HDD
- Supports Quick Sync for hardware transcoding

**Performance Build** (~$1000):
- Intel Core i5-12400
- 16GB DDR4 RAM
- 500GB NVMe SSD + 8TB HDD
- Excellent Quick Sync performance

**Power User Build** (~$2000+):
- Intel Core i7-12700K or AMD Ryzen 7 5700X
- 32GB DDR4 RAM
- 1TB NVMe SSD + 4x8TB HDD (RAID)
- NVIDIA GPU for transcoding
- 10Gbe networking

### D. Automation Scripts

**Auto-organize script** (`/opt/mce/scripts/organize.sh`):
```bash
#!/bin/bash
# Automatically organize downloaded media

DOWNLOADS="/opt/mce/downloads/complete"
TV_DEST="/opt/mce/media/tv"
MOVIE_DEST="/opt/mce/media/movies"

# Move TV shows
find "$DOWNLOADS" -name "*.S[0-9][0-9]E[0-9][0-9].*" -exec mv {} "$TV_DEST/" \;

# Move movies
find "$DOWNLOADS" -name "*.mkv" -o -name "*.mp4" -size +1G -exec mv {} "$MOVIE_DEST/" \;
```

---

For more detailed information and updates, visit https://github.com/howtomgr/mce