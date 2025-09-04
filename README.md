# N8N Cloudflare Setup

Production-ready Docker Compose setup for self-hosted n8n with Cloudflare Tunnel, PostgreSQL, and Qdrant.

## 🚀 Features

- Cloudflare Tunnel for secure remote access
- Automatic SSL/TLS with Let's Encrypt
- PostgreSQL database with health checks
- Qdrant vector database for AI features
- Production-ready with resource limits

## 📋 Prerequisites

- Docker and Docker Compose
- Cloudflare account with tunnel token
- Domain name (for SSL)

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Cloudflare    │────│   nginx-proxy   │────│      n8n        │
│     Tunnel      │    │  (SSL/TLS)      │    │   (Web App)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   PostgreSQL    │    │     Qdrant      │
                       │   (Database)    │    │ (Vector DB)     │
                       └─────────────────┘    └─────────────────┘
```

### Services

- **nginx-proxy**: Reverse proxy with automatic SSL
- **acme-companion**: Let's Encrypt certificate management
- **n8n**: Main workflow automation application
- **n8n-import**: One-time service for importing backups
- **postgres**: Primary database
- **qdrant**: Vector database for AI features
- **cloudflared**: Cloudflare tunnel for secure access

## 🚀 Quick Start

```bash
git clone <repository-url>
cd n8n-cloudflare
cp .env.example .env
# Edit .env with your values
docker compose up -d
```

Access: https://yourdomain.com

## ⚙️ Configuration

### Key Environment Variables

| Variable                         | Description                       | Required |
| -------------------------------- | --------------------------------- | -------- |
| `POSTGRES_USER`                  | Database username                 | Yes      |
| `POSTGRES_PASSWORD`              | Database password                 | Yes      |
| `POSTGRES_DB`                    | Database name                     | Yes      |
| `N8N_ENCRYPTION_KEY`             | Encryption key for sensitive data | Yes      |
| `N8N_USER_MANAGEMENT_JWT_SECRET` | JWT secret for user sessions      | Yes      |
| `WEBHOOK_URL`                    | Public URL for webhooks           | Yes      |
| `VIRTUAL_HOST`                   | Domain for nginx-proxy            | Yes      |
| `LETSENCRYPT_HOST`               | Domain for SSL certificate        | Yes      |
| `CLOUDFLARED_TOKEN`              | Cloudflare tunnel token           | Yes      |

## 🔧 Management

```bash
# Start/stop
docker compose up -d
docker compose down

# Logs
docker compose logs -f n8n

# Restart service
docker compose restart n8n

# Access container
docker compose exec n8n /bin/bash
```

## 🔄 Backup and Restore

### Automated n8n Backup

```bash
# Create dated backup directory
mkdir -p ./backup/$(date +%Y%m%d)

# Export Workflows
docker compose exec n8n n8n export:workflow --all --output=/backup/$(date +%Y%m%d)/workflows.json

# Export Credentials
docker compose exec n8n n8n export:credentials --all --output=/backup/$(date +%Y%m%d)/credentials.json

# Verify exports
ls -la ./backup/$(date +%Y%m%d)/
```

Files are saved to `./backup/YYYYMMDD/` on host.

### Restore After Fresh Install

```bash
# After restarting n8n with fresh storage
docker compose exec n8n n8n import:workflow --input=/backup/YYYYMMDD/workflows.json
docker compose exec n8n n8n import:credentials --input=/backup/YYYYMMDD/credentials.json
```

### Database Backup

```bash
docker exec postgres pg_dump -U n8n n8ndb > n8n_backup_$(date +%Y%m%d).sql
docker exec -i postgres psql -U n8n n8ndb < n8n_backup.sql
```

## 🐛 Troubleshooting

### Common Issues

**Service not accessible:**

```bash
docker compose ps
docker compose logs n8n
```

**Database issues:**

```bash
docker compose logs postgres
docker compose exec postgres pg_isready -U n8n -d n8ndb
```

**SSL issues:**

```bash
docker compose logs nginx-proxy
docker compose logs acme-companion
```

## 🆘 Support

- [n8n Documentation](https://docs.n8n.io/)
- [Community Forum](https://community.n8n.io/)
