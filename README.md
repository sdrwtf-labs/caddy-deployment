# Caddy Reverse Proxy Deployment

This repository handles the deployment of Caddy as the central reverse proxy for the homelab. It includes a custom build process to integrate the Cloudflare DNS module for automatic Let's Encrypt wildcard certificate generation via the DNS-01 challenge.

## Prerequisites
- A working Docker host.
- Ports `80` and `443` must be completely free on the host.
- A Cloudflare API token with `Zone.DNS` edit permissions for the target domain.

## Deployment

1. Prepare the deployment directory:
```bash
sudo mkdir -p /opt/caddy
sudo chown $USER:$USER /opt/caddy
cd /opt/caddy

git clone git@github.com:sdrwtf-labs/caddy-deployment.git .
```

2. Configure the environment:

```bash
cp .env.example .env
```

Edit `.env` and provide your real Cloudflare API token and email address.

3. Build and start the proxy:

```bash
docker compose up -d --build
```

*(The `--build` flag is crucial for the first run or after Caddy version updates to compile the Cloudflare module).*


## Adding New Services

To add a new service, edit the `Caddyfile` and add a new `@service` block within the wildcard section. Then apply the changes without downtime:

```bash
docker compose exec caddy caddy reload -c /etc/caddy/Caddyfile
```
