# N8N Cloudflare Setup

A production-ready Docker Compose setup for self-hosted n8n workflow automation with Cloudflare Tunnel integration, PostgreSQL database, and Qdrant vector database.

## 🚀 Features

- **Secure Environment Configuration**: All sensitive data externalized to `.env` file
- **Cloudflare Tunnel**: Secure remote access without exposing ports
- **SSL/TLS**: Automatic Let's Encrypt certificates via nginx-proxy
- **Database**: PostgreSQL with health checks and proper configuration
- **Vector Database**: Qdrant for AI/ML workflows
- **Resource Management**: CPU and memory limits for all services
- **Production Ready**: Pinned Docker image versions and proper orchestration

## 📋 Prerequisites

- Docker and Docker Compose
- Cloudflare account with tunnel token
- Domain name (optional, for SSL)

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

### 1. Clone and Setup

```bash
git clone <repository-url>
cd n8n-cloudflare
```

### 2. Environment Configuration

```bash
# Copy the example environment file
cp .env.example .env

# Edit .env with your values
nano .env
```

### 3. Generate Secure Keys (Optional)

If you want to regenerate the encryption keys:

```bash
# Generate new encryption key
openssl rand -hex 32

# Generate new JWT secret
openssl rand -base64 32

# Generate new database password
openssl rand -base64 16
```

### 4. Start Services

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Check status
docker-compose ps
```

### 5. Access n8n

- **Local**: http://localhost:5678
- **Cloudflare Tunnel**: https://yourdomain.com (if configured)

## ⚙️ Configuration

### Environment Variables

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

### Domain Configuration

Update these variables in `.env`:

```bash
WEBHOOK_URL=https://yourdomain.com
VIRTUAL_HOST=yourdomain.com
LETSENCRYPT_HOST=yourdomain.com
LETSENCRYPT_EMAIL=your-email@example.com
```

## 🔧 Management Commands

```bash
# Stop all services
docker-compose down

# Restart specific service
docker-compose restart n8n

# View logs
docker-compose logs n8n
docker-compose logs -f  # Follow logs

# Update services
docker-compose pull
docker-compose up -d

# Backup database
docker exec -t postgres pg_dump -U n8n n8ndb > backup.sql

# Access n8n container
docker exec -it n8n /bin/bash
```

## 🔒 Security Features

- **Environment Variables**: Sensitive data not in Docker Compose
- **Cloudflare Tunnel**: Secure remote access without port exposure
- **SSL/TLS**: Automatic HTTPS with Let's Encrypt
- **Resource Limits**: Prevents resource exhaustion
- **Network Isolation**: Services in separate networks
- **Version Pinning**: Reproducible deployments

## 📊 Monitoring

### Basic Monitoring

```bash
# Check service health
docker-compose ps

# View resource usage
docker stats

# Check logs for errors
docker-compose logs --tail=100
```

### Health Checks

- PostgreSQL: Database connectivity check
- Services: Automatic restart on failure
- Resources: CPU and memory limits enforced

## 🔄 Backup and Restore

### Database Backup

```bash
# Create backup
docker exec postgres pg_dump -U n8n n8ndb > n8n_backup_$(date +%Y%m%d_%H%M%S).sql

# Restore backup
docker exec -i postgres psql -U n8n n8ndb < n8n_backup.sql
```

### n8n Data Backup

n8n data is automatically backed up to `./n8n/backup/` directory:

- Workflows: `./n8n/backup/workflows/`
- Credentials: `./n8n/backup/credentials/`

## 🐛 Troubleshooting

### Common Issues

**n8n not accessible:**

```bash
# Check if services are running
docker-compose ps

# Check n8n logs
docker-compose logs n8n

# Verify environment variables
docker-compose exec n8n env | grep N8N_
```

**Database connection issues:**

```bash
# Check PostgreSQL logs
docker-compose logs postgres

# Test database connectivity
docker-compose exec postgres pg_isready -U n8n -d n8ndb
```

**SSL certificate issues:**

```bash
# Check nginx-proxy logs
docker-compose logs nginx-proxy

# Check acme-companion logs
docker-compose logs acme-companion
```

### Debug Mode

```bash
# Start with debug logging
docker-compose up  # Without -d flag

# Access container shell
docker-compose exec n8n /bin/bash
```

## 📈 Scaling

### Development Setup

For development with additional services:

```bash
# Create override file
cp docker-compose.override.yml.example docker-compose.override.yml

# Edit for development needs
nano docker-compose.override.yml
```

### Production Scaling

- Use Docker Swarm or Kubernetes for orchestration
- Add load balancers for multiple n8n instances
- Implement database clustering for high availability
- Add monitoring and alerting

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

- **Documentation**: https://docs.n8n.io/
- **Community**: https://community.n8n.io/
- **Issues**: Create an issue in this repository

---

**Note**: This setup is configured for production use with security best practices. For development, consider using the override file to modify resource limits and add debugging tools.
