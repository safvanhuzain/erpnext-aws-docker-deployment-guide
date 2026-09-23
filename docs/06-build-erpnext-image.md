# 6. Build the ERPNext image

## Pin and verify versions

Example tested pair:

```bash
git ls-remote --exit-code --tags https://github.com/frappe/frappe.git refs/tags/v16.29.0
git ls-remote --exit-code --tags https://github.com/frappe/erpnext.git refs/tags/v16.30.0
```

Do not assume equal tag numbers are required. Verify the combination using upstream release information and a test image.

## Official image check

```bash
sudo docker pull --platform linux/arm64 frappe/erpnext:v16.30.0
sudo docker image inspect frappe/erpnext:v16.30.0 --format '{{.Os}}/{{.Architecture}}'
sudo docker run --rm frappe/erpnext:v16.30.0 bench version
```

## Define apps

Create `apps.json` with public URLs only—never embed a token:

```json
[
  {
    "url": "https://github.com/frappe/erpnext.git",
    "branch": "v16.30.0"
  },
  {
    "url": "https://github.com/EXAMPLE-OWNER/EXAMPLE-CUSTOM-APP.git",
    "branch": "main"
  }
]
```

```bash
chmod 600 apps.json
jq empty apps.json
```

Frappe is installed first by `bench init`; apps in `apps.json`, including ERPNext, are then fetched in listed order.

## Private repository authentication

Generate a fine-grained GitHub token with read-only **Contents** access to only the custom-app repository.

Enter it without putting it in shell history:

```bash
bash
read -rsp "Paste GitHub token: " GITHUB_TOKEN
echo
umask 077
printf 'machine github.com\nlogin EXAMPLE-OWNER\npassword %s\n' "$GITHUB_TOKEN" > github.netrc
unset GITHUB_TOKEN
chmod 600 github.netrc
```

Test Git access:

```bash
ln -sf "$PWD/github.netrc" "$PWD/.netrc"
HOME="$PWD" GIT_TERMINAL_PROMPT=0 git ls-remote \
  https://github.com/EXAMPLE-OWNER/EXAMPLE-CUSTOM-APP.git \
  refs/heads/main
rm "$PWD/.netrc"
```

A private GitHub REST URL may return `404` when authentication is missing or invalid. A successful `git ls-remote` confirms the credential and repository/branch access required by the build.

## Add the netrc BuildKit secret mount

Create a working copy of the layered Containerfile:

```bash
cp images/layered/Containerfile images/layered/Containerfile.private
```

In its `bench init` `RUN` instruction, mount both secrets:

```Dockerfile
RUN --mount=type=secret,id=apps_json,target=/opt/frappe/apps.json,uid=1000,gid=1000 \
    --mount=type=secret,id=netrc,target=/home/frappe/.netrc,uid=1000,gid=1000,mode=0600 \
```

Confirm both mounts exist:

```bash
grep -n -- '--mount=type=secret' images/layered/Containerfile.private
```

## Run the build safely in tmux

```bash
tmux new -s erpnext-build
```

Build a versioned image:

```bash
sudo docker build \
  --progress=plain \
  --platform linux/arm64 \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=v16.29.0 \
  --secret=id=apps_json,src=apps.json \
  --secret=id=netrc,src=github.netrc \
  --tag=example-erpnext:v16.30.0-1 \
  --file=images/layered/Containerfile.private \
  . 2>&1 | tee /tmp/erpnext-build.log
```

Detach with `Ctrl+B`, then `D`. Reattach later:

```bash
tmux attach -t erpnext-build
```

If already inside tmux, do not nest another session. Check with `echo "$TMUX"`.

## Verify the finished image

```bash
sudo docker image inspect example-erpnext:v16.30.0-1 \
  --format '{{.Os}}/{{.Architecture}}'

sudo docker run --rm example-erpnext:v16.30.0-1 bench version

sudo docker run --rm example-erpnext:v16.30.0-1 \
  bash -lc 'ls -1 apps'
```

Expected architecture: `linux/arm64`. Expected apps include `frappe`, `erpnext` and the custom app.

After a successful build, securely remove the temporary credential:

```bash
shred -u github.netrc
```

Recreate it only when a future private-app build requires it.
