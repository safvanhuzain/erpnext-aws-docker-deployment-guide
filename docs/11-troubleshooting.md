# 11. Troubleshooting

## Session Manager `Connect` is disabled

Check the EC2 IAM role, `AmazonSSMManagedInstanceCore`, SSM Agent status and outbound network connectivity. After attaching a role, allow several minutes for the instance to appear as a managed node.

## `$` prompt instead of `user@host:path$`

Session Manager opened `/bin/sh`. This is valid but lacks some Bash features.

```sh
bash
```

## `read: Illegal option -s`

`read -s` is unavailable in `/bin/sh`. Start Bash first:

```sh
bash
read -rsp "Secret: " SECRET_VALUE
echo
```

Always `unset SECRET_VALUE` after use.

## `Permission denied` after typing a directory path

A path alone is treated as an executable command. Use `cd`:

```bash
cd /opt/erpnext/frappe_docker
```

## Pending kernel upgrade

Package installation may report that the running kernel is older than the installed kernel. Reboot and reconnect:

```bash
sudo reboot
```

## Image appears to be missing

Check exact tags:

```bash
sudo docker images --no-trunc
sudo docker image inspect example-erpnext:v16.30.0-1
```

Images are local to the Docker daemon. A failed or interrupted build does not produce the final tagged image, but BuildKit may retain reusable cache layers.

## Browser session expired during a long build

Run long builds in tmux. If `tmux ls` says a session is attached and the current shell is already inside tmux, check:

```bash
echo "$TMUX"
```

Detach with `Ctrl+B`, then `D`; reattach with:

```bash
tmux attach -d -t erpnext-build
```

## Private repository API returns `404`

GitHub intentionally obscures private repositories when authentication is missing or unauthorized. Verify:

- Token owner can access the repository
- Fine-grained token selects the exact repository
- Repository permission **Contents: Read-only** is enabled
- Token is not expired or awaiting organization approval
- Repository owner and name are correct

Test the Git operation used by the build with a temporary `.netrc`; do not paste the token into the repository URL.

## `bench init` fails cloning the custom app

Typical final message:

```text
CommandFailedError: git clone https://github.com/... --branch main
```

Confirm `apps.json`, branch name, private access and the Containerfile secret mount. BuildKit secrets must target `/home/frappe/.netrc` with permissions readable by the `frappe` user.

## Unable to scroll through build output

Log output to a file:

```bash
sudo docker build ... 2>&1 | tee /tmp/erpnext-build.log
less /tmp/erpnext-build.log
```

Inside `less`, use arrows/Page Up/Page Down and press `q` to exit.

## Compose shows `platform: linux/amd64`

Apply the ARM64 override after the base Compose files and inspect the rendered configuration:

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  config > /tmp/erpnext-compose.yml

grep 'platform:' /tmp/erpnext-compose.yml | sort -u
```

Expected: `linux/arm64`.

## `.site-admin-password` is one byte or empty

The shell variable was empty when written. Generate/read the password again, verify the file size without printing the secret, then unset the variable.

Never use `cat` to display production passwords.

## `configurator` shows `Exited (0)`

Normal: it is a one-shot initialization service. Exit code `0` means success.

## ERPNext works locally but not through the domain

Check:

```bash
dig +short A app.example.com
curl -I http://127.0.0.1
sudo docker compose ... ps -a
```

Then verify the Elastic IP association, security-group rules, DNS record and Traefik routing rule.

## Let's Encrypt reports `NXDOMAIN`

Example:

```text
DNS problem: NXDOMAIN looking up A for app.example.com
```

`SITES_RULE` contains a hostname that does not exist publicly or DNS has not propagated. Correct it to the exact verified hostname:

```dotenv
SITES_RULE=Host(`app.example.com`)
```

Validate Compose and run `up -d` again. One accidental failed request normally causes no damage, but repeated failures can hit Let's Encrypt rate limits.

## HTTPS certificate works but the wrong site/404 appears

Traefik's hostname and Frappe's internal site mapping are separate:

```dotenv
SITES_RULE=Host(`app.example.com`)
FRAPPE_SITE_NAME_HEADER=erp.internal.example
```

Confirm the internal site exists:

```bash
sudo docker compose ... exec backend ls -1 sites
```

## Duplicate keys in `.env`

Docker Compose may use the later occurrence, creating confusing behavior. Detect duplicates:

```bash
grep -nE '^(HTTP_PUBLISH_PORT|HTTPS_PUBLISH_PORT|FRAPPE_SITE_NAME_HEADER|SITES_RULE|LETSENCRYPT_EMAIL)=' .env
```

Keep only one copy of each variable.

## Administrator cannot log in

Reset without opening the UI:

```bash
sudo docker compose ... exec backend \
  bench --site erp.internal.example set-admin-password "$NEW_ADMIN_PASSWORD"
```

Use the case-sensitive username `Administrator`, clear cache and retry in a private browser window.

## Useful diagnostic bundle

Do not include secrets when sharing output:

```bash
uname -m
sudo docker version
sudo docker compose version
sudo docker compose ... ps -a
sudo docker compose ... logs --tail=200 backend frontend proxy
df -h
free -h
```
