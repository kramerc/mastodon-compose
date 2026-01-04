# Mastodon Docker Compose Setup

This repository contains a Docker Compose configuration for running a Mastodon instance.

## Architecture

The setup consists of two main services:

- **Mastodon**: The social media platform
- **Redis**: In-memory data store for caching and queues

## Services

### Mastodon Service

The Mastodon service runs the social media platform.

**Image**: `lscr.io/linuxserver/mastodon:latest`

**Environment Variables**:
- `PUID=1000`: User ID for file ownership
- `PGID=1000`: Group ID for file ownership  
- `TZ=America/Los_Angeles`: Timezone configuration

**Ports**:
- `8080:80`: HTTP port
- `8443:443`: HTTPS port

**Volumes**:
- `/mnt/mastodon:/config`: Mastodon configuration directory

### Redis Service

The Redis service provides in-memory data storage for caching and background job queues.

**Image**: `redis:8`

**Volumes**:
- `./redis:/data`: Redis data persistence directory

**Command**: `redis-server --appendonly yes` (enables data persistence)

## Environment Configuration

The Mastodon service uses environment variables defined in `.env.production`. Here are the key configuration sections:

### Basic Configuration

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `LOCAL_DOMAIN` | Your Mastodon instance domain | `kramerc.social` |
| `SINGLE_USER_MODE` | Enable single-user mode | `true` |

### Security Keys

| Variable | Description |
|----------|-------------|
| `SECRET_KEY_BASE` | Rails secret key for encryption |
| `OTP_SECRET` | One-time password secret |
| `VAPID_PRIVATE_KEY` | Web Push private key |
| `VAPID_PUBLIC_KEY` | Web Push public key |

### Database Encryption

| Variable | Description |
|----------|-------------|
| `ACTIVE_RECORD_ENCRYPTION_DETERMINISTIC_KEY` | Deterministic encryption key |
| `ACTIVE_RECORD_ENCRYPTION_KEY_DERIVATION_SALT` | Key derivation salt |
| `ACTIVE_RECORD_ENCRYPTION_PRIMARY_KEY` | Primary encryption key |

### Database Configuration

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `DB_HOST` | PostgreSQL database host | `jasper.kekra.net` |
| `DB_PORT` | PostgreSQL database port | `5432` |
| `DB_NAME` | Database name | `mastodon_production` |
| `DB_USER` | Database username | `mastodon` |
| `DB_PASS` | Database password | `[configured]` |

### Redis Configuration

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `REDIS_HOST` | Redis server host | `redis` |
| `REDIS_PORT` | Redis server port | `6379` |

### S3 Storage Configuration

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `S3_ENABLED` | Enable S3 storage | `true` |
| `S3_PROTOCOL` | S3 protocol | `https` |
| `S3_REGION` | S3 region | `us-east-1` |
| `S3_ENDPOINT` | S3 endpoint URL | `https://jasper.kekra.net:9000` |
| `S3_HOSTNAME` | S3 hostname | `jasper.kekra.net:9000` |
| `S3_BUCKET` | S3 bucket name | `files-kramerc-social` |
| `S3_ALIAS_HOST` | S3 alias hostname | `files.kramerc.social` |
| `AWS_ACCESS_KEY_ID` | AWS access key | `mastodon` |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key | `[configured]` |

### SMTP Configuration

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `SMTP_SERVER` | SMTP server hostname | `smtp.fastmail.com` |
| `SMTP_PORT` | SMTP server port | `587` |
| `SMTP_LOGIN` | SMTP username | `kramer@kramerc.com` |
| `SMTP_PASSWORD` | SMTP password | `[configured]` |
| `SMTP_AUTH_METHOD` | SMTP authentication method | `plain` |
| `SMTP_OPENSSL_VERIFY_MODE` | SSL verification mode | `none` |
| `SMTP_ENABLE_STARTTLS` | Enable STARTTLS | `auto` |
| `SMTP_FROM_ADDRESS` | From email address | `'Mastodon <notifications@kramerc.social>'` |

## File Structure

```
├── compose.yaml           # Docker Compose configuration
├── .env.production       # Environment variables for Mastodon
├── update.sh            # Update script
├── redis/               # Redis data directory
└── /mnt/mastodon/       # Mastodon configuration files (mounted)
    ├── keys/           # SSL certificates
    ├── log/            # Log files
    ├── mastodon/       # Mastodon data
    ├── nginx/          # Nginx configuration
    ├── php/            # PHP configuration
    └── www/            # Web files
```

## Usage

### Starting the Services

```bash
docker compose up -d
```

### Updating the Services

Use the provided update script:

```bash
./update.sh
```

This script will:
1. Pull the latest Docker images
2. Restart services with updated images
3. Remove orphaned containers
4. Clean up unused Docker resources

### Stopping the Services

```bash
docker compose down
```

### Viewing Logs

```bash
# View all service logs
docker compose logs

# View specific service logs
docker compose logs mastodon
docker compose logs redis

# Follow logs in real-time
docker compose logs -f
```

## Security Considerations

1. **Environment Variables**: The `.env.production` file contains sensitive credentials. Ensure it's properly secured and not committed to version control.

2. **SSL/TLS**: The configuration includes SSL certificates in the `/mnt/mastodon/keys/` directory for secure communications.

3. **Ports**: The Mastodon service exposes ports 8080 (HTTP) and 8443 (HTTPS). Ensure proper firewall rules are in place.

## Backup Considerations

Important directories to backup:
- `/mnt/mastodon/` - Contains Mastodon configuration and data
- `./redis/` - Contains Redis data
- `.env.production` - Contains environment configuration

## Troubleshooting

1. **Permission Issues**: Ensure the `PUID` (1000) and `PGID` (1000) values match your system's user and group IDs for `/mnt/mastodon`.

2. **Redis Connection**: If Mastodon can't connect to Redis, ensure the Redis service is running and `REDIS_HOST=redis` in `.env.production`.

3. **Database Connection**: Verify the database credentials and network connectivity to the PostgreSQL server.

4. **S3 Storage**: Ensure S3 credentials and endpoint configuration are correct for file uploads.

## Maintenance

Regular maintenance tasks:
1. Run `./update.sh` to keep services updated
2. Monitor log files in `/mnt/mastodon/log/`
3. Check disk usage for the `/mnt/mastodon/` and `./redis/` directories
4. Verify backup procedures
