# 🚀 Self-Hosting n8n with Docker + Cloudflare Tunnel

This comprehensive guide will help you set up **n8n** (workflow automation tool) on your own hardware and securely expose it to the internet using **Cloudflare Tunnel** with zero-trust security.

## 📖 Table of Contents
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Detailed Setup Instructions](#-detailed-setup-instructions)
- [Configuration](#-configuration)
- [Accessing n8n](#-accessing-n8n)
- [Security Notes](#-security-notes)
- [Troubleshooting](#-troubleshooting)
- [Maintenance](#-maintenance)

---

## 📋 Prerequisites

Before starting, ensure you have:

- **Hardware/Server**: Any system capable of running Docker
  - Minimum: 1GB RAM, 1 CPU core, 10GB storage
  - Recommended: 2GB+ RAM, 2+ CPU cores, 20GB+ storage
  - OS: Linux (preferred), macOS, or Windows with WSL2

- **Software Requirements**:
  - [Docker](https://docs.docker.com/get-docker/) (v20.10+)
  - [Docker Compose](https://docs.docker.com/compose/install/) (v2.0+)

- **Cloudflare Account**:
  - Active Cloudflare account
  - Domain registered and managed through Cloudflare
  - Access to Cloudflare Zero Trust (free tier available)

---

## ⚡ Quick Start

```bash
# 1. Create project directory
mkdir n8n-selfhost && cd n8n-selfhost

# 2. Download docker-compose.yml
curl -O https://github.com/patel5d2/n8n/blob/main/docker-compose.yml

# 3. Edit the tunnel token (see detailed instructions below)
nano docker-compose.yml

# 4. Start the services
docker compose up -d

# 5. Check status
docker compose logs -f
```

---

## ⚙️ Detailed Setup Instructions

### Step 1: Create Project Directory
```bash
mkdir n8n-selfhost
cd n8n-selfhost
```

### Step 2: Create Docker Compose Configuration

Create the `docker-compose.yml` file:

```bash
nano docker-compose.yml
```

Copy and paste the following configuration:

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=changeme123
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=https://your-domain.com
      - GENERIC_TIMEZONE=UTC
      - N8N_LOG_LEVEL=info
    volumes:
      - n8n_data:/home/node/.n8n
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "5678:5678"
    networks:
      - n8n_network

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel --no-autoupdate run
    environment:
      - TUNNEL_TOKEN=<YOUR_TUNNEL_TOKEN>
    networks:
      - n8n_network
    depends_on:
      - n8n

volumes:
  n8n_data:
    driver: local

networks:
  n8n_network:
    driver: bridge
```

### Step 3: Configure Cloudflare Tunnel

1. **Login to Cloudflare Dashboard**
   - Navigate to [Cloudflare Dashboard](https://dash.cloudflare.com)
   - Select your domain

2. **Create a New Tunnel**
   - Go to **Zero Trust** → **Networks** → **Tunnels**
   - Click **Create a tunnel**
   - Select **Cloudflared**
   - Give your tunnel a descriptive name (e.g., `n8n-production`)

3. **Choose Environment**
   - Select **Docker** as your environment
   - Copy the tunnel token that appears

4. **Configure Public Hostname**
   - Add a public hostname (e.g., `n8n.yourdomain.com`)
   - Set service type to **HTTP**
   - Set URL to `n8n:5678` (internal Docker service name and port)
   - Save the tunnel

### Step 4: Update Configuration

Replace `<YOUR_TUNNEL_TOKEN>` in your `docker-compose.yml` with the actual token from Cloudflare:

```bash
# Use your preferred text editor
nano docker-compose.yml

# Or use sed for quick replacement
sed -i 's/<YOUR_TUNNEL_TOKEN>/your-actual-token-here/' docker-compose.yml
```

### Step 5: Launch the Services

```bash
# Start services in detached mode
docker compose up -d

# Monitor logs (optional)
docker compose logs -f

# Check service status
docker compose ps
```

---

## 🔧 Configuration

### Environment Variables

Key n8n environment variables you can customize:

| Variable | Default | Description |
|----------|---------|-------------|
| `N8N_BASIC_AUTH_USER` | admin | Basic auth username |
| `N8N_BASIC_AUTH_PASSWORD` | changeme123 | Basic auth password |
| `WEBHOOK_URL` | - | Public webhook URL for external services |
| `GENERIC_TIMEZONE` | UTC | Server timezone |
| `N8N_LOG_LEVEL` | info | Log verbosity (error, warn, info, verbose, debug) |

### Security Recommendations

1. **Change Default Credentials**:
   ```yaml
   environment:
     - N8N_BASIC_AUTH_USER=your-username
     - N8N_BASIC_AUTH_PASSWORD=your-secure-password
   ```

2. **Use Environment File**:
   Create a `.env` file for sensitive data:
   ```bash
   echo "TUNNEL_TOKEN=your-token-here" > .env
   echo "N8N_AUTH_PASSWORD=your-password" >> .env
   ```

   Then reference in docker-compose.yml:
   ```yaml
   environment:
     - TUNNEL_TOKEN=${TUNNEL_TOKEN}
     - N8N_BASIC_AUTH_PASSWORD=${N8N_AUTH_PASSWORD}
   ```

---

## ✅ Accessing n8n

### Via Cloudflare Tunnel (Recommended)
1. Check your Cloudflare dashboard for the public URL
2. Or check the container logs: `docker compose logs cloudflared`
3. Access your n8n instance at `https://n8n.yourdomain.com`

### Local Access (Development/Testing)
- Direct access: `http://localhost:5678`
- Use this only for local development

### First Time Setup
1. Navigate to your n8n URL
2. Enter your configured username and password
3. Follow the n8n setup wizard
4. Start creating your first workflow!

---

## 🔒 Security Notes

### Production Security
- ✅ **Always use HTTPS** via Cloudflare (secure cookies require it)
- ✅ **Strong authentication** - Change default credentials
- ✅ **Regular updates** - Keep Docker images updated
- ✅ **Network isolation** - Use Docker networks
- ✅ **Backup data** - Regular backups of n8n_data volume

### Cloudflare Tunnel Benefits
- **Zero-trust security** - No open ports on your server
- **Automatic HTTPS** - SSL/TLS termination at Cloudflare edge
- **DDoS protection** - Built-in protection
- **Global CDN** - Fast access worldwide

### Important Security Warnings
- ⚠️ **Never share your tunnel token publicly**
- ⚠️ **Use strong, unique passwords**
- ⚠️ **Regularly rotate credentials**
- ⚠️ **Monitor access logs**

---

## 🔧 Troubleshooting

### Common Issues

#### 1. Container Won't Start
```bash
# Check container logs
docker compose logs n8n
docker compose logs cloudflared

# Check container status
docker compose ps

# Restart services
docker compose restart
```

#### 2. Tunnel Connection Issues
```bash
# Verify tunnel token
docker compose logs cloudflared | grep -i "tunnel"

# Test local n8n access
curl http://localhost:5678
```

#### 3. Permission Issues
```bash
# Fix volume permissions
sudo chown -R 1000:1000 n8n_data/

# Or recreate with correct permissions
docker compose down -v
docker compose up -d
```

#### 4. Port Conflicts
```bash
# Check if port 5678 is in use
sudo netstat -tulpn | grep 5678

# Change port in docker-compose.yml if needed
ports:
  - "5679:5678"  # Use different external port
```

### Health Checks
```bash
# Check if services are healthy
docker compose ps

# Test n8n API
curl -I http://localhost:5678

# Check Cloudflare tunnel status
docker compose exec cloudflared cloudflared tunnel list
```

---

## 🛠️ Maintenance

### Regular Updates
```bash
# Pull latest images
docker compose pull

# Recreate containers with new images
docker compose up -d --force-recreate

# Clean up old images
docker image prune -f
```

### Backup and Restore
```bash
# Backup n8n data
docker run --rm -v n8n-selfhost_n8n_data:/data -v $(pwd):/backup alpine tar czf /backup/n8n-backup-$(date +%Y%m%d).tar.gz /data

# Restore from backup
docker run --rm -v n8n-selfhost_n8n_data:/data -v $(pwd):/backup alpine tar xzf /backup/n8n-backup-20231201.tar.gz -C /
```

### Monitoring
```bash
# Monitor resource usage
docker stats

# Check logs
docker compose logs -f --tail=50

# View n8n workflows
curl -u admin:password http://localhost:5678/rest/workflows
```

---

## 📚 Additional Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Cloudflare Tunnel Documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [n8n Community](https://community.n8n.io/)

---

## 🤝 Contributing

Feel free to submit issues, fork the repository, and create pull requests for any improvements.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**⭐ If this guide helped you, please give it a star!**

> **Tip**: Bookmark this guide for future reference and updates.
