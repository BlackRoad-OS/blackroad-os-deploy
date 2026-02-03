# blackroad-os-deploy

[![GitHub](https://img.shields.io/badge/GitHub-BlackRoad-OS-purple?style=for-the-badge&logo=github)](https://github.com/BlackRoad-OS/blackroad-os-deploy)
[![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)](https://github.com/BlackRoad-OS/blackroad-os-deploy)
[![BlackRoad](https://img.shields.io/badge/BlackRoad-OS-black?style=for-the-badge)](https://blackroad.io)

# BlackRoad Deploy 🚀

Railway-like deployment system for your Raspberry Pis and Droplets.

Deploy multi-language applications (Node.js, Python, Go, Rust) with automatic buildpack detection, GitHub webhooks, and DNS management.

## Features

✨ **Multi-Language Support**
- Node.js (auto-detects package.json)
- Python (requirements.txt, pyproject.toml, Pipfile)
- Go (go.mod)
- Rust (Cargo.toml)
- Docker (Dockerfile)

🔄 **Automatic Deployments**
- GitHub webhook integration
- Push to deploy
- Automatic rebuilds on code changes

🌐 **DNS Management**
- Cloudflare DNS automation
- Point domains to your apps
- Automatic SSL/TLS with Caddy

🎯 **Multiple Targets**
- aria64 (Raspberry Pi)
- shellfish (DigitalOcean Droplet)
- alice (Raspberry Pi)
- lucidia (Raspberry Pi)

## Quick Start

### 1. Install

```bash
cd ~/blackroad-deploy
chmod +x br-deploy scripts/*.sh scripts/*.py
```

### 2. Deploy Your First App

```bash
# Deploy a Node.js app
./br-deploy deploy ./my-node-app aria64

# Deploy a Python app to Shellfish
./br-deploy deploy ./my-python-api shellfish

# Deploy with custom name
./br-deploy deploy ./my-app aria64 custom-name
```

### 3. Manage Apps

```bash
# List deployed apps
./br-deploy list aria64

# View logs
./br-deploy logs my-app aria64

# Restart an app
./br-deploy restart my-app aria64

# Check server status
./br-deploy status shellfish
```

### 4. DNS Management

```bash
# Point domain to your app
./scripts/dns-manager.sh set myapp.blackroad.io aria64

# List all DNS records
./scripts/dns-manager.sh list

# Delete a record
./scripts/dns-manager.sh delete myapp.blackroad.io
```

### 5. GitHub Webhooks (Auto-Deploy)

```bash
# Add a repo to auto-deploy
./scripts/webhook-manager.sh add my-repo aria64 main

# Start the webhook server
./scripts/webhook-manager.sh start

# Check status
./scripts/webhook-manager.sh status

# View logs
tail -f ~/.blackroad-deploy/webhook.log
```

Then add webhook to GitHub:
- URL: `http://YOUR_IP:9000/webhook`
- Secret: `blackroad-deploy-secret`
- Content type: `application/json`
- Events: Just the push event

## Architecture

```
┌─────────────────┐
│  Your Computer  │
│   br-deploy     │
└────────┬────────┘
         │
         ├──────────────┐
         │              │
    ┌────▼────┐    ┌────▼────────┐
    │  aria64 │    │  shellfish  │
    │   Pi    │    │   Droplet   │
    └────┬────┘    └────┬────────┘
         │              │
    ┌────▼────────────────▼────┐
    │   Caddy Reverse Proxy    │
    │   (Auto SSL/TLS)         │
    └──────────┬────────────────┘
               │
         ┌─────▼─────┐
         │   Apps    │
         │ Port 3XXX │
         └───────────┘
```

## Directory Structure

```
~/blackroad-deploy/
├── br-deploy              # Main CLI tool
├── deploy.sh              # Original deploy script
├── README.md              # This file
├── dockerfiles/           # Language-specific Dockerfiles
│   ├── Dockerfile.nodejs
│   ├── Dockerfile.python
│   ├── Dockerfile.go
│   └── Dockerfile.rust
└── scripts/
    ├── dns-manager.sh     # Cloudflare DNS management
    ├── webhook-server.py  # GitHub webhook server
    └── webhook-manager.sh # Webhook configuration
```

## Environment Variables

```bash
# Cloudflare (for DNS management)
export CF_API_TOKEN="your-token"
export CF_ZONE_ID="your-zone-id"

# Webhook server
export WEBHOOK_PORT=9000
export WEBHOOK_SECRET="your-secret"
```

## Examples

### Deploy a Node.js Express App

```bash
# Your app structure:
# my-api/
#   ├── package.json
#   ├── index.js
#   └── ...

./br-deploy deploy ./my-api shellfish
```

### Deploy a Python FastAPI App

```bash
# Your app structure:
# my-api/
#   ├── requirements.txt
#   ├── app.py (or main.py)
#   └── ...

./br-deploy deploy ./my-api aria64 my-api
```

### Deploy with Auto-Deploy from GitHub

```bash
# 1. Configure auto-deploy
./scripts/webhook-manager.sh add my-repo aria64 main

# 2. Start webhook server
./scripts/webhook-manager.sh start

# 3. Add webhook in GitHub repo settings:
#    - Payload URL: http://YOUR_IP:9000/webhook
#    - Content type: application/json
#    - Secret: blackroad-deploy-secret
#    - Events: push

# 4. Push to GitHub - automatically deploys!
git push origin main
```

### Set Up Custom Domain

```bash
# 1. Deploy your app
./br-deploy deploy ./my-app shellfish

# 2. Note the port (e.g., 3150)

# 3. Set up DNS
./scripts/dns-manager.sh set myapp.blackroad.io shellfish

# 4. Configure Caddy reverse proxy (on target server)
# Add to Caddyfile:
# myapp.blackroad.io {
#     reverse_proxy localhost:3150
# }
```

## Troubleshooting

### Check deployment status
```bash
./br-deploy list aria64
./br-deploy logs my-app aria64
```

### Check server resources
```bash
./br-deploy status aria64
```

### Webhook not working?
```bash
# Check if server is running
./scripts/webhook-manager.sh status

# View logs
tail -f ~/.blackroad-deploy/webhook.log

# Restart server
./scripts/webhook-manager.sh stop
./scripts/webhook-manager.sh start
```

## Advanced Usage

### Custom Dockerfile

If you have a Dockerfile, it will be used automatically:

```bash
./br-deploy deploy ./my-docker-app aria64
```

### Environment Variables

Create a `.env` file in your app directory or pass via Docker:

```bash
# Modify br-deploy script to load .env
# Or use docker run -e flags
```

### Multiple Environments

```bash
# Production on shellfish
./br-deploy deploy ./my-app shellfish my-app-prod

# Staging on aria64
./br-deploy deploy ./my-app aria64 my-app-staging
```

## Supported Frameworks

- **Node.js**: Express, Fastify, NestJS, Next.js, etc.
- **Python**: Flask, FastAPI, Django, etc.
- **Go**: net/http, Gin, Echo, Fiber, etc.
- **Rust**: Actix, Rocket, Axum, etc.

## Ports

- 3000-3099: Node.js apps
- 3100-3199: Python apps
- 3200-3299: Go apps
- 3300-3399: Rust apps
- 8000: LLM service
- 9000: Webhook server
- 80/443: Caddy reverse proxy

## Tips

1. **Health Checks**: Add a `/health` endpoint to your apps
2. **Logs**: Use `./br-deploy logs <app> <target>` to debug
3. **DNS**: Always set DNS after deploying
4. **Webhooks**: Great for CI/CD automation
5. **Caddy**: Use for automatic HTTPS

## What's Next?

- [ ] Add database provisioning
- [ ] Environment variable management UI
- [ ] Metrics and monitoring
- [ ] Load balancing across Pis
- [ ] Auto-scaling based on traffic

---

Made with ❤️ for BlackRoad OS

---

## 📜 License & Copyright

**Copyright © 2026 BlackRoad OS, Inc. All Rights Reserved.**

**CEO:** Alexa Amundson | **PROPRIETARY AND CONFIDENTIAL**

This software is NOT for commercial resale. Testing purposes only.

### 🏢 Enterprise Scale:
- 30,000 AI Agents
- 30,000 Human Employees
- CEO: Alexa Amundson

**Contact:** blackroad.systems@gmail.com

See [LICENSE](LICENSE) for complete terms.
