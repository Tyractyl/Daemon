# Daemon

Talon, The Tyractyl Daemon

Tyractyl Daemon is Tyractyl's server control plane, built for the rapidly changing gaming industry and designed to be
highly performant and secure. The daemon provides an HTTP API allowing you to interface directly with running server
instances, fetch server logs, generate backups, and control all aspects of the server lifecycle.

In addition, Tyractyl Daemon ships with a built-in SFTP server allowing your system to remain free of Tyractyl specific
dependencies, and allowing users to authenticate with the same credentials they would normally use to access the Panel.

## Features

- **High Performance**: Built with Go for optimal performance and low resource usage
- **Secure**: All game servers run in isolated Docker containers
- **Built-in SFTP**: No external dependencies required for file management
- **HTTP API**: Comprehensive REST API for server management
- **Real-time**: WebSocket support for live server logs and console
- **Backup Support**: Local and S3 backup storage options
- **Transfer Support**: Server transfer between nodes with zero downtime

## Installation

### Prerequisites

- **Go 1.21+**
- **Docker 20.10+** with Docker Compose
- **Linux** (Ubuntu 20.04+, CentOS 8+, or similar)
- **Root** or **sudo** access
- **Git**

### Quick Install (Binary)

#### 1. Download and Install
```bash
# Download the latest binary
wget https://github.com/Tyractyl/Daemon/releases/latest/download/talon-linux-amd64

# Make it executable
chmod +x talon-linux-amd64

# Move to system path
sudo mv talon-linux-amd64 /usr/local/bin/talon
```

#### 2. Create Configuration
```bash
# Create config directory
sudo mkdir -p /etc/tyractyl

# Create configuration file
sudo nano /etc/tyractyl/config.yml
```

**Sample Configuration (`/etc/tyractyl/config.yml`):**
```yaml
# Panel Configuration
api:
  host: 0.0.0.0
  port: 8080
  ssl:
    enabled: false
    cert: /etc/ssl/certs/daemon.crt
    key: /etc/ssl/private/daemon.key

# Panel Connection
remote:
  host: https://your-panel-domain.com
  token: "your-panel-api-token"

# Docker Configuration
docker:
  network: 172.18.0.0/16
  timezone: UTC
  tmpfs_size: 100M

# SFTP Configuration
sftp:
  host: 0.0.0.0
  port: 2022
  key_size: 2048

# System Configuration
system:
  data_directory: /var/lib/tyractyl
  backup_directory: /var/lib/tyractyl/backups
  log_level: info
```

#### 3. Create System User
```bash
# Create dedicated user
sudo useradd -r -s /bin/false tyractyl

# Create directories
sudo mkdir -p /var/lib/tyractyl/{volumes,backups,logs}
sudo chown -R tyractyl:tyractyl /var/lib/tyractyl
```

#### 4. Create Systemd Service
```bash
sudo nano /etc/systemd/system/tyractyl-daemon.service
```

```ini
[Unit]
Description=Tyractyl Daemon
After=docker.service network.target
Requires=docker.service

[Service]
Type=simple
User=tyractyl
Group=tyractyl
LimitNOFILE=4096
PIDFile=/var/run/tyractyl/daemon.pid
ExecStart=/usr/local/bin/talon
Restart=always
RestartSec=5s
StartLimitInterval=0

# Environment
Environment="TALON_CONFIG=/etc/tyractyl/config.yml"

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=tyractyl-daemon

[Install]
WantedBy=multi-user.target
```

#### 5. Enable and Start Service
```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable service
sudo systemctl enable tyractyl-daemon

# Start service
sudo systemctl start tyractyl-daemon

# Check status
sudo systemctl status tyractyl-daemon
```

### Build from Source

#### 1. Clone Repository
```bash
git clone https://github.com/Tyractyl/Daemon.git
cd Daemon
```

#### 2. Build Binary
```bash
# Build for current platform
go build -o talon

# Or build for specific platform
GOOS=linux GOARCH=amd64 go build -o talon-linux-amd64
```

#### 3. Install Binary
```bash
sudo cp talon /usr/local/bin/talon
sudo chmod +x /usr/local/bin/talon
```

#### 4. Follow steps 2-5 from the binary installation above

### Docker Installation

#### 1. Using Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  tyractyl-daemon:
    image: tyractyl/daemon:latest
    container_name: tyractyl-daemon
    restart: unless-stopped
    privileged: true
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/tyractyl:/var/lib/tyractyl
      - /etc/tyractyl:/etc/tyractyl
      - /tmp:/tmp
    environment:
      - TALON_CONFIG=/etc/tyractyl/config.yml
```

#### 2. Start Container
```bash
# Create directories
sudo mkdir -p /var/lib/tyractyl/{volumes,backups,logs}
sudo mkdir -p /etc/tyractyl

# Start with docker-compose
docker-compose up -d
```

### Configuration

#### Panel Token Setup
1. Login to your Tyractyl Panel
2. Go to **Admin → API → Create New**
3. Give it a descriptive name (e.g., "Daemon Token")
4. Copy the generated token
5. Add it to your `config.yml` under `remote.token`

#### Docker Network
The daemon creates a dedicated Docker network for server isolation:
- Default network: `172.18.0.0/16`
- Each server gets its own isolated container
- Containers are managed automatically by the daemon

#### Firewall Configuration
Make sure these ports are open:
- **8080**: Daemon API (internal use)
- **2022**: SFTP server
- **Game ports**: As configured in your panel (e.g., 25565 for Minecraft)

```bash
# UFW example
sudo ufw allow 8080/tcp
sudo ufw allow 2022/tcp
sudo ufw allow 25565:25570/tcp  # Example for Minecraft servers
```

### Verification

#### 1. Check Daemon Status
```bash
# Check service status
sudo systemctl status tyractyl-daemon

# Check logs
sudo journalctl -u tyractyl-daemon -f
```

#### 2. Test API Connection
```bash
# Test daemon API
curl http://localhost:8080/api/system

# Should return system information
```

#### 3. Verify Panel Connection
1. Go to your Tyractyl Panel
2. Navigate to **Admin → Nodes**
3. Your daemon should appear as "Online"
4. Test creating a server to verify full functionality

### Troubleshooting

#### Common Issues

1. **Permission Denied**:
   ```bash
   sudo chown -R tyractyl:tyractyl /var/lib/tyractyl
   sudo usermod -aG docker tyractyl
   ```

2. **Docker Connection Failed**:
   ```bash
   sudo systemctl restart docker
   sudo usermod -aG docker tyractyl
   ```

3. **Panel Connection Failed**:
   - Verify panel URL is accessible from daemon
   - Check API token is correct and has proper permissions
   - Ensure firewall allows communication

#### Logs
```bash
# View daemon logs
sudo journalctl -u tyractyl-daemon -f

# View specific log levels
sudo journalctl -u tyractyl-daemon --priority=err
```

### Updates

#### Binary Updates
```bash
# Stop service
sudo systemctl stop tyractyl-daemon

# Download new version
wget https://github.com/Tyractyl/Daemon/releases/latest/download/talon-linux-amd64
chmod +x talon-linux-amd64
sudo mv talon-linux-amd64 /usr/local/bin/talon

# Start service
sudo systemctl start tyractyl-daemon
```

#### Source Updates
```bash
cd /path/to/Daemon
git pull origin main
go build -o talon
sudo cp talon /usr/local/bin/talon
sudo systemctl restart tyractyl-daemon
```

## Documentation

* [Panel Documentation](https://pterodactyl.io/panel/1.0/getting_started.html)
* [Daemon Documentation](https://pterodactyl.io/wings/1.0/installing.html)
* [Community Guides](https://pterodactyl.io/community/about.html)

## Reporting Issues

Please use the [Tyractyl/Panel](https://github.com/Tyractyl/Panel) repository to report any issues or make
feature requests for the Daemon.
