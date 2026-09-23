# 7. Deploy the stack and create the site

## ARM64 override

Use the official ARM64 override if the checked-out repository provides it. Validate the rendered configuration before starting:

```bash
grep 'platform:' /tmp/erpnext-compose.yml | sort -u
```

All application services should resolve to `linux/arm64`.

## Create `.env`

Example:

```dotenv
CUSTOM_IMAGE=example-erpnext
CUSTOM_TAG=v16.30.0-1
PULL_POLICY=never
RESTART_POLICY=unless-stopped
HTTP_PUBLISH_PORT=80
FRAPPE_SITE_NAME_HEADER=erp.internal.example
DB_PASSWORD=GENERATE_A_LONG_RANDOM_PASSWORD
```

```bash
chmod 600 .env
```

Never commit `.env`.

## Start HTTP-only services initially

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  config --quiet
```

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  up -d
```

Check status:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  ps -a
```

`configurator` exiting with code `0` is normal. MariaDB should be healthy and runtime services should be up.

## Create the site without exposing passwords in history

```bash
bash
read -rsp "Database root password: " DB_PASSWORD
echo
read -rsp "ERPNext Administrator password: " ADMIN_PASSWORD
echo
```

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench new-site erp.internal.example \
    --db-root-username root \
    --db-root-password "$DB_PASSWORD" \
    --admin-password "$ADMIN_PASSWORD" \
    --mariadb-user-host-login-scope='%' \
    --install-app erpnext \
    --set-default
```

```bash
unset DB_PASSWORD ADMIN_PASSWORD
```

Install an image-bundled custom app if `bench new-site` did not install it:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench --site erp.internal.example install-app example_custom_app
```

Then:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench --site erp.internal.example migrate
```

Enable the scheduler:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench --site erp.internal.example enable-scheduler
```

Verify:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.noproxy.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench --site erp.internal.example list-apps

curl -I http://127.0.0.1
```
