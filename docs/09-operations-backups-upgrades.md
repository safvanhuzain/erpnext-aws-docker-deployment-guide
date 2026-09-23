# 9. Operations, backups and upgrades

Define a shell helper or script for the repeated production Compose file list. Until then, always include the HTTPS override:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  ps -a
```

## Logs

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  logs --tail=200 backend frontend scheduler queue-short queue-long proxy
```

## Reset Administrator password

```bash
bash
read -rsp "New Administrator password: " NEW_ADMIN_PASSWORD
echo
```

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench --site erp.internal.example set-admin-password "$NEW_ADMIN_PASSWORD"

unset NEW_ADMIN_PASSWORD
```

Login username is case-sensitive: `Administrator`.

## Local backup

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  exec backend \
  bench --site erp.internal.example backup --with-files --compress
```

A backup stored only on the same EC2/EBS volume does not protect against instance, volume or account loss. Copy encrypted backups to separate object storage and apply lifecycle retention.

For Frappe v16, evaluate the official [`frappe/offsite_backups`](https://github.com/frappe/offsite_backups) `version-16` branch in a test image. Do not install it interactively into running containers; bake it into a new image. Verify its Python dependencies and test backup plus restore before production rollout.

## Safe application update flow

1. Review Frappe, ERPNext and custom-app changes.
2. Pin new versions in `apps.json` and build a new immutable tag, for example `v16.31.0-1`.
3. Verify architecture and `bench version` in the image.
4. Create database and files backups; verify off-host upload.
5. Test the image against a restored non-production site.
6. Change `CUSTOM_TAG` in `.env`.
7. Run `docker compose config --quiet`.
8. Start the new image.
9. Run `bench --site ... migrate`.
10. Restart services and perform smoke tests.
11. Retain the previous image tag for rollback until validation is complete.

Never overwrite a production tag. Do not use `latest`.

## Host maintenance

- Apply Ubuntu security updates regularly.
- Reboot when a kernel upgrade requires it.
- Monitor disk, memory, CPU credits and Docker volume growth.
- Take EBS snapshots as an additional layer, not as the only database backup.
- Periodically test a complete restore.

Useful checks:

```bash
df -h
free -h
sudo docker system df
sudo docker volume ls
```
