# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Docker Compose environment for PHP development, providing a complete development stack with Nginx, PHP (8.4 and 7.4), Redis, and optional services (MySQL, MongoDB, Elasticsearch, RabbitMQ, Consul).

The `app/` directory contains application code that is mounted into the containers. The ERP applications (am-erp-php and am-erp-vue) in `app/erp/` are separate git repositories.

## Architecture

### Active Services
- **php-fpm**: PHP 8.4 with Swoole 5.x, Redis, AMQP, Soap extensions - ports 9000, 9501-9510
- **php-fpm74**: PHP 7.4 with Swoole, Redis, Mongo, AMQP, Soap - port 9601
- **nginx**: Nginx 1.24 - ports 80, 8080, 443
- **redis-db**: Redis 7.0 - port 6379

### Optional Services (Commented Out)
Uncomment in `services/docker-compose.yml` to enable:
- **mysql-db**: MySQL 8.0 - port 3306
- **mongo-db**: MongoDB 6.0 - port 27017
- **elasticsearch**: Elasticsearch 8.7 - ports 9100, 9200
- **rabbitmq**: RabbitMQ 3.11 - ports 5672, 15672 (management UI)
- **consul**: Consul - port 8500

## Directory Structure

- `app/` - Application code, mapped to `/data/www` in PHP containers
- `services/` - Docker Compose files and service configurations
  - `docker-compose.yml` - Main orchestration file
  - `php/`, `php74/`, `nginx/`, `redis/`, etc. - Service-specific configs
- `data/` - Persistent data for databases (survives container deletion)
- `logs/` - Log files from all services

## Docker Commands

**All commands must be run from the `services/` directory:**

```bash
cd services

# Build all containers
docker-compose build

# Start all services in background
docker-compose up -d

# Stop all services
docker-compose down

# View running services
docker-compose ps

# View logs
docker-compose logs -f php-fpm
docker-compose logs -f nginx

# Rebuild specific service
docker-compose build php-fpm
docker-compose up -d php-fpm
```

## Accessing Containers

```bash
cd services

# Enter PHP 8.4 container
docker-compose exec php-fpm bash

# Enter PHP 7.4 container
docker-compose exec php-fpm74 bash

# Enter Nginx container
docker-compose exec nginx bash

# Enter Redis container
docker-compose exec redis-db sh
```

## PHP Container Details

### PHP 8.4 (php-fpm)

Built from `services/php/Dockerfile`:
- Base: PHP 8.3-fpm (upgraded to 8.4 compatible)
- Extensions: gd, zip, pdo_mysql, opcache, mysqli, bcmath, sockets, soap, pcntl, redis, swoole 5.x
- Tools: git, vim, curl, wget, composer
- Timezone: Asia/Shanghai
- Composer mirror: Tencent Cloud

Working directory in container: `/data/www`

Configuration files:
- `services/php/php.ini` → `/usr/local/etc/php/php.ini`
- `services/php/php-fpm.conf` → `/usr/local/etc/php-fpm.conf`

### PHP 7.4 (php-fpm74)

Similar setup but with PHP 7.4 for legacy support.

## Volume Mappings

Host → Container:
- `../app` → `/data/www` (read-write)
- `./php/php.ini` → `/usr/local/etc/php/php.ini` (read-only)
- `./php/php-fpm.conf` → `/usr/local/etc/php-fpm.conf` (read-only)
- `../logs/php-fpm` → `/var/log/php-fpm` (read-write)

## Nginx Configuration

- Config directory: `services/nginx/conf.d/`
- SSL certificates: `services/nginx/certs/`
- Main config: `services/nginx/nginx.conf`

Nginx is linked to both php-fpm and php-fpm74, allowing you to route different applications to different PHP versions.

## Network Configuration

Docker daemon.json configuration (recommended):
```json
{
  "bip": "172.16.10.1/24",
  "default-address-pools": [{"base": "10.10.0.0/16", "size": 24}],
  "dns": ["114.114.114.114"],
  "registry-mirrors": ["https://docker.xuanyuan.me"]
}
```

## Troubleshooting

### Elasticsearch Won't Start

Increase `vm.max_map_count`:

**Windows (WSL2):**
```bash
wsl -d docker-desktop
sysctl -w vm.max_map_count=262144
```

**Mac:**
```bash
docker-machine ssh
sudo sysctl -w vm.max_map_count=262144
```

**Linux:**
```bash
sudo sysctl -w vm.max_map_count=262144
# Or persist it:
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
sysctl -p
```

### MySQL on ARM Processors

If running on ARM (Apple Silicon), change `services/mysql/Dockerfile` base image from `mysql` to `mysql/mysql-server`.

### Port Already in Use

Check `services/docker-compose.yml` for port mappings and modify if conflicts exist with other services on your host.

## Adding New Services

1. Create a new directory in `services/` for the service
2. Add Dockerfile and configuration files
3. Add service definition to `services/docker-compose.yml`
4. Link to other services as needed
5. Run `docker-compose build` and `docker-compose up -d`

## Important Notes

- Changes to code in `app/` are immediately reflected in containers (mounted volume)
- Changes to Dockerfiles require rebuilding: `docker-compose build <service>`
- Changes to docker-compose.yml require recreation: `docker-compose up -d`
- Database data in `data/` persists even when containers are removed
- Logs in `logs/` are mounted volumes - can be viewed from host or container
