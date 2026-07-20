# Caddy Reverse Proxy Deployment

This repository handles the deployment of Caddy as the central reverse proxy for the homelab. It includes a custom build process to integrate the Cloudflare DNS module for automatic Let's Encrypt wildcard certificate generation via the DNS-01 challenge.

## Prerequisites
- A working Docker host.
- Ports `80` and `443` must be completely free on the host.
- A Cloudflare API token with `Zone.DNS` edit permissions for the target domain.

## Split-Brain DNS & ACME Challenge Configuration
In a homelab environment with a local authoritative DNS server (Split-Brain DNS), the Let's Encrypt DNS-01 challenge can fail if Caddy queries the local DNS for the Start of Authority (SOA) record, or if it checks for the TXT record before it has fully propagated across Cloudflare's global network.

To prevent this, this deployment includes specific overrides:

1. **`compose.yaml` (Container DNS):**
   The `dns:` block forces the Caddy container to use public resolvers (e.g., Cloudflare `1.1.1.1` and `1.0.0.1`) instead of inheriting the Docker host's local DNS. This ensures the ACME client resolves the correct public SOA record rather than receiving a false negative from the local DNS server.

2. **`Caddyfile` (TLS Block):**
   - `resolvers 1.1.1.1:53 1.0.0.1:53`: Forces the Let's Encrypt plugin to check the challenge status strictly against public resolvers.
   - `propagation_delay 60s`: Forces Caddy to wait 60 seconds after creating the TXT record before verifying it. This allows Cloudflare enough time to propagate the record globally, preventing negative DNS caching (NXDOMAIN)

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
