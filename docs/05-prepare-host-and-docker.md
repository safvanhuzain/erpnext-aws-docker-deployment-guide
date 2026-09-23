# 5. Prepare Ubuntu and install Docker

## Update the host

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git curl ca-certificates gnupg unzip jq dnsutils
```

Messages stating that disabled/static services were not started are usually informational. If `needrestart` reports a pending kernel upgrade, reboot:

```bash
sudo reboot
```

The Session Manager terminal will disconnect. Wait for the instance to return, then reconnect and verify:

```bash
uname -r
apt list --upgradable
```

## Install Docker Engine

Use Docker's current official Ubuntu installation instructions. The repository-based flow is preferred over an unverified convenience script.

After installation:

```bash
sudo systemctl enable --now docker
sudo systemctl is-active docker
sudo docker version
sudo docker compose version
sudo docker buildx version
```

Validate ARM64 containers:

```bash
sudo docker run --rm hello-world
sudo docker run --rm alpine uname -m
```

Expected:

```text
aarch64
```

## Prepare the application directory

```bash
sudo mkdir -p /opt/erpnext
sudo chown "$USER":"$USER" /opt/erpnext
cd /opt/erpnext
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

For reproducibility, check out a reviewed commit or tag rather than silently following the changing `main` branch.

## Shell prompt confusion

Session Manager may open `/bin/sh`, displaying only `$`. This is not an error. Commands such as `read -s` require Bash:

```sh
bash
```

Do not type the prompt text, hostname or current directory as a command. Use:

```bash
whoami
hostname
pwd
```
