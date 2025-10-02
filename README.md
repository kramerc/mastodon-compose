# Mastodon Docker Compose Setup

This repository contains a Docker Compose configuration for running a Mastodon instance with WireGuard VPN support.

## Architecture

The setup consists of two main services:

- **WireGuard**: Provides VPN connectivity and network isolation
- **Mastodon**: The social media platform running through the WireGuard network

## Services

### WireGuard Service

The WireGuard service provides secure VPN connectivity and acts as the network gateway for the Mastodon instance.

**Image**: `lscr.io/linuxserver/wireguard:latest`

**Capabilities**:
- `NET_ADMIN`: Network administration capabilities
- `SYS_MODULE`: System module loading (optional)

**Environment Variables**:
- `PUID=1451001103`: User ID for file ownership
- `PGID=1451000513`: Group ID for file ownership
- `TZ=America/Los_Angeles`: Timezone configuration
- `LOG_CONFS=true`: Enable configuration logging (optional)

**Volumes**:
- `./wireguard:/config`: WireGuard configuration directory
- `/lib/modules:/lib/modules`: Kernel modules (optional)

### Mastodon Service

The Mastodon service runs the social media platform and uses the WireGuard service for network connectivity.

**Image**: `lscr.io/linuxserver/mastodon:latest`

**Network Mode**: `service:wireguard` (shares network stack with WireGuard)

**Environment Variables**:
- `PUID=1451001103`: User ID for file ownership
- `PGID=1451000513`: Group ID for file ownership  
- `TZ=America/Los_Angeles`: Timezone configuration

**Volumes**:
- `./config:/config`: Mastodon configuration directory

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
| `REDIS_HOST` | Redis server host | `jasper.kekra.net` |
| `REDIS_PORT` | Redis server port | `30059` |
| `REDIS_PASSWORD` | Redis authentication password | `[configured]` |

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
├── config/              # Mastodon configuration files
│   ├── keys/           # SSL certificates
│   ├── log/            # Log files
│   ├── mastodon/       # Mastodon data
│   ├── nginx/          # Nginx configuration
│   ├── php/            # PHP configuration
│   └── www/            # Web files
└── wireguard/          # WireGuard configuration
    ├── privatekey
    ├── publickey
    ├── coredns/
    ├── templates/
    └── wg_confs/
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
docker compose logs wireguard

# Follow logs in real-time
docker compose logs -f
```

## Security Considerations

1. **Environment Variables**: The `.env.production` file contains sensitive credentials. Ensure it's properly secured and not committed to version control.

2. **WireGuard Configuration**: The WireGuard service has elevated privileges (`NET_ADMIN`, `SYS_MODULE`). Ensure the WireGuard configuration is properly secured.

3. **Network Isolation**: Mastodon runs through the WireGuard network, providing an additional layer of network isolation.

4. **SSL/TLS**: The configuration includes SSL certificates in the `config/keys/` directory for secure communications.

## Backup Considerations

Important directories to backup:
- `./config/` - Contains Mastodon configuration and data
- `./wireguard/` - Contains WireGuard configuration
- `.env.production` - Contains environment configuration

## Troubleshooting

1. **Permission Issues**: Ensure the `PUID` and `PGID` values match your system's user and group IDs.

2. **Network Connectivity**: If Mastodon can't connect to external services, check the WireGuard configuration and routing.

3. **Database Connection**: Verify the database credentials and network connectivity to the PostgreSQL server.

4. **S3 Storage**: Ensure S3 credentials and endpoint configuration are correct for file uploads.

## Maintenance

Regular maintenance tasks:
1. Run `./update.sh` to keep services updated
2. Monitor log files in `config/log/`
3. Check disk usage for the `config/` directory
4. Verify backup procedures
