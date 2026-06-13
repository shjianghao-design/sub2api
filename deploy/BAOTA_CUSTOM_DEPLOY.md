# Baota Production Deployment For Custom Development

Use this workflow when you run your own modified Sub2API source code in
production. Do not use `docker-compose.dev-hot.yml` in production; it is only
for local development.

## Recommended Layout

```bash
/opt/sub2api/
  source/   # your git repository
  deploy/   # deploy files, data, env
```

One practical setup:

```bash
mkdir -p /opt/sub2api
cd /opt/sub2api
git clone <your-repo-url> source
cd source/deploy
cp .env.baota.example .env.baota
```

Edit `.env.baota` and set database credentials, admin password, `JWT_SECRET`,
and `TOTP_ENCRYPTION_KEY`.

Generate secrets:

```bash
openssl rand -hex 32
openssl rand -hex 32
```

## Baota Database Preparation

In Baota PostgreSQL, create:

- database: `sub2api`
- user: `sub2api`
- password: same as `DATABASE_PASSWORD`

PostgreSQL must accept TCP connections on `127.0.0.1:5432`. Redis should be
available on `127.0.0.1:6379`; set `REDIS_PASSWORD` if Baota Redis has one.

## Start

```bash
cd /opt/sub2api/source/deploy
docker compose --env-file .env.baota -f docker-compose.baota.yml up -d --build
docker logs -f sub2api
```

The first start runs `AUTO_SETUP=true`: it writes `deploy/data/config.yaml`,
applies migrations, and creates the first admin user.

## Baota Reverse Proxy

Create a Baota website for your domain and reverse proxy it to:

```text
http://127.0.0.1:8080
```

Recommended Nginx snippets:

```nginx
underscores_in_headers on;
client_max_body_size 256m;
proxy_buffering off;
proxy_read_timeout 3600s;
proxy_send_timeout 3600s;
```

## Update After Code Changes

Commit or pull your custom code, then rebuild the image:

```bash
cd /opt/sub2api/source
git pull
cd deploy
docker compose --env-file .env.baota -f docker-compose.baota.yml up -d --build
```

If you build elsewhere, export and load the image:

```bash
# build machine
docker build -t sub2api-custom:latest .
docker save sub2api-custom:latest | gzip > sub2api-custom-latest.tar.gz

# server
gunzip -c sub2api-custom-latest.tar.gz | docker load
cd /opt/sub2api/source/deploy
docker compose --env-file .env.baota -f docker-compose.baota.yml up -d
```

## Backup Before Upgrades

Back up PostgreSQL and `deploy/data/` before each production upgrade.

```bash
pg_dump -h 127.0.0.1 -U sub2api -d sub2api -Fc -f sub2api-$(date +%F).dump
tar czf sub2api-data-$(date +%F).tar.gz data/
```
