# 8. DNS and HTTPS

This setup uses the official `overrides/compose.https.yaml`, which runs Traefik and obtains a free Let's Encrypt certificate.

## Prerequisites

- DNS `A` record resolves to the Elastic IP
- No incorrect `AAAA` record exists for an IPv4-only server
- Security group allows TCP 80 and 443 from the internet
- `overrides/compose.https.yaml` exists
- Traefik image supports ARM64

```bash
dig +short A app.example.com
dig +short AAAA app.example.com
test -f overrides/compose.https.yaml && echo OK
sudo docker buildx imagetools inspect traefik:v3.7 | grep -A2 linux/arm64
```

## Internal site name versus public hostname

They may differ:

```dotenv
FRAPPE_SITE_NAME_HEADER=erp.internal.example
SITES_RULE=Host(`app.example.com`)
```

- `FRAPPE_SITE_NAME_HEADER` tells the frontend which existing Frappe site to serve.
- `SITES_RULE` tells Traefik which public hostname to route and certify.

If both names are identical, use the same hostname in both settings.

## Configure `.env`

Back it up:

```bash
cp .env .env.before-https
chmod 600 .env.before-https
```

Add exactly one copy of each setting:

```dotenv
HTTP_PUBLISH_PORT=80
HTTPS_PUBLISH_PORT=443
FRAPPE_SITE_NAME_HEADER=erp.internal.example
SITES_RULE=Host(`app.example.com`)
LETSENCRYPT_EMAIL=YOUR_EMAIL@example.com
```

Detect duplicates:

```bash
grep -nE '^(HTTP_PUBLISH_PORT|HTTPS_PUBLISH_PORT|FRAPPE_SITE_NAME_HEADER|SITES_RULE|LETSENCRYPT_EMAIL)=' .env
```

There should be exactly five lines.

## Validate and switch to HTTPS

Do not use `compose.noproxy.yaml` together with `compose.https.yaml`.

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  config --quiet
```

No output means validation passed.

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  up -d --remove-orphans
```

This does not rebuild the custom image. It pulls Traefik if necessary, recreates the frontend networking and starts the proxy.

## Verify certificate and redirect

```bash
curl -I https://app.example.com
curl -I http://app.example.com
```

```bash
echo | openssl s_client \
  -connect app.example.com:443 \
  -servername app.example.com 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

Set the public URL in Frappe:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench --site erp.internal.example set-config host_name https://app.example.com
```

## Automatic renewal

Traefik handles renewal automatically and stores ACME data in the persistent `cert-data` volume. No annual cron job is required.

Renewal requires:

- DNS remains pointed to the server
- Ports 80 and 443 remain reachable
- Traefik remains running
- The `cert-data` volume is not deleted
- The HTTPS override remains in future Compose commands

Never run `docker compose down -v`; `-v` deletes persistent volumes.
