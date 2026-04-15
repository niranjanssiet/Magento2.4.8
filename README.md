# Magento 2.4.8 Community Edition Docker Environment

## Overview
This repository defines a fully containerized Magento 2.4.8 development stack.

- **PHP 8.2 FPM** with Magento-required extensions
- **Nginx 1.25** web server
- **MySQL 8.0** database
- **OpenSearch 2.11.1** search engine
- **Redis 7** cache/session storage
- **MailHog** email capture tool
- **phpMyAdmin** database UI

## Requirements
- Docker and Docker Compose
- At least 4GB RAM assigned to Docker
- Local ports: 80, 443, 3306, 9200, 6379, 8080, 1025, 8025

## Environment
Docker Compose reads the root `.env` file automatically. All service settings are driven from the repository root `.env` file.

## Quick Start

### 1. Start the stack
```bash
docker-compose up -d --build
```

### 2. Create Magento project inside the PHP container
```bash
docker-compose exec php composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition:2.4.8 .
```

There is no need to install Composer on the host machine; Composer is available inside the `php` container.

### 3. Prepare Magento environment
```bash
docker-compose exec php cp env.example .env
```

Inside the Magento root `.env`, update the database and search settings if needed:
```bash
MAGENTO_BACKEND_FRONTNAME=admin
DB_HOST=mysql
DB_NAME=magento2
DB_USER=magento
DB_PASSWORD=magento123
REDIS_HOST=redis
REDIS_PORT=6379
OPENSEARCH_HOST=opensearch
OPENSEARCH_PORT=9200
OPENSEARCH_ENABLE=1
```

### 4. Install Magento
```bash
docker-compose exec php bin/magento setup:install \
  --base-url=http://localhost/ \
  --db-host=mysql \
  --db-name=magento2 \
  --db-user=magento \
  --db-password=magento123 \
  --admin-firstname=Admin \
  --admin-lastname=User \
  --admin-email=admin@example.com \
  --admin-user=admin \
  --admin-password=Admin@123 \
  --language=en_US \
  --currency=USD \
  --timezone=America/Chicago \
  --use-rewrites=1 \
  --search-engine=opensearch \
  --opensearch-host=opensearch \
  --opensearch-port=9200 \
  --session-save=redis \
  --session-save-redis-host=redis \
  --session-save-redis-port=6379 \
  --cache-backend=redis \
  --cache-backend-redis-server=redis \
  --cache-backend-redis-port=6379
```

### 5. Deploy static content and compile
```bash
docker-compose exec php bin/magento setup:static-content:deploy -f
docker-compose exec php bin/magento setup:di:compile
```

### 6. Set file permissions
```bash
docker-compose exec php chown -R www-data:www-data /var/www/html
docker-compose exec php chmod -R 755 /var/www/html
docker-compose exec php chmod -R 777 /var/www/html/var /var/www/html/pub/media
```

## Access
- Storefront: http://localhost
- Admin Panel: http://localhost/admin
- OpenSearch: http://localhost:9200
- phpMyAdmin: http://localhost:8080
- MailHog: http://localhost:8025
- MySQL: localhost:3306

## Useful Commands

- Enter PHP container:
```bash
docker-compose exec php bash
```
- View logs:
```bash
docker-compose logs -f php
docker-compose logs -f nginx
docker-compose logs -f mysql
```
- Reindex:
```bash
docker-compose exec php bin/magento indexer:reindex
```
- Clear cache:
```bash
docker-compose exec php bin/magento cache:clean
docker-compose exec php bin/magento cache:flush
```
- Disable maintenance mode:
```bash
docker-compose exec php bin/magento maintenance:disable
```
- Database access:
```bash
docker-compose exec mysql mysql -u magento -pmagento123 magento2
```

## Troubleshooting

- If OpenSearch fails to start, verify Docker memory and ulimits.
- If Redis is unavailable, confirm `redis` container is running.
- If static content is missing, rerun deployment.
- If permissions fail, chown and chmod the Magento root.

## What changed
- `php` service now builds from `Dockerfile` and includes Composer.
- Required PHP extensions for Magento are installed during container build.
- Environment values are loaded from `.env`.
- No Composer install is needed on the host machine.
