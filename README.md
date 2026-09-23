# ERPNext on AWS EC2 with Docker

Production-oriented, beginner-friendly deployment guide for ERPNext on a single AWS EC2 instance using the official [`frappe_docker`](https://github.com/frappe/frappe_docker) project.

The examples use generic names and placeholders. Never commit real passwords, tokens, customer names, AWS account IDs, private repository URLs, IP addresses, or `.env` files.

## Tested reference architecture

- AWS Region: choose the region nearest the users
- EC2: `t4g.medium` (2 vCPU, 4 GiB RAM, ARM64/Graviton)
- OS: Ubuntu Server 24.04 LTS ARM64
- Storage: 50 GiB encrypted `gp3`
- Access: AWS Systems Manager Session Manager
- Runtime: Docker Engine and Docker Compose
- Application: pinned Frappe v16 + ERPNext v16 + optional custom app
- Data services: MariaDB and Redis containers with persistent Docker volumes
- Public endpoint: Elastic IP, DNS, Traefik and Let's Encrypt
- TLS renewal: automatic through Traefik

This is a practical single-server design for a small deployment. It is not a high-availability architecture: the EC2 instance remains a single point of failure.

## Deployment flow

| Stage | Guide | Outcome |
|---|---|---|
| 1 | [Planning and architecture](docs/01-planning-and-architecture.md) | Understand components, limitations and required inputs |
| 2 | [AWS account, IAM and budget](docs/02-aws-account-iam-budget.md) | Use non-root administration, enable SSM and configure cost controls |
| 3 | [VPC, security group and Elastic IP](docs/03-networking-and-security.md) | Prepare public networking without exposing unnecessary ports |
| 4 | [Launch the EC2 instance](docs/04-launch-ec2.md) | Create the ARM64 Ubuntu host and attach its IAM role |
| 5 | [Prepare Ubuntu and install Docker](docs/05-prepare-host-and-docker.md) | Patch the OS and validate Docker on ARM64 |
| 6 | [Build the ERPNext image](docs/06-build-erpnext-image.md) | Build a pinned image containing ERPNext and an optional private custom app |
| 7 | [Start the stack and create the site](docs/07-deploy-and-create-site.md) | Run MariaDB, Redis and Frappe services; create the ERPNext site |
| 8 | [Configure DNS and HTTPS](docs/08-dns-and-https.md) | Publish the site securely with automatically renewed TLS |
| 9 | [Operations, backups and upgrades](docs/09-operations-backups-upgrades.md) | Operate, back up, restore and update safely |
| 10 | [CI/CD roadmap](docs/10-cicd-roadmap.md) | Move from manual builds to GitHub Actions and ECR |
| 11 | [Troubleshooting](docs/11-troubleshooting.md) | Diagnose the errors encountered during installation |
| 12 | [Publish this documentation to GitHub](docs/12-publish-to-github.md) | Create and push the sanitized documentation repository |

## Quick status checks

Run commands from the cloned `frappe_docker` directory:

```bash
cd /opt/erpnext/frappe_docker
```

```bash
sudo docker compose \
  -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  -f overrides/compose.arm64.yaml \
  ps -a
```

```bash
curl -I https://app.example.com
```

## Placeholder convention

| Placeholder | Example meaning |
|---|---|
| `app.example.com` | Public hostname opened by users |
| `erp.internal.example` | Frappe site directory/name |
| `example-erpnext` | Custom image repository name |
| `example_custom_app` | Python/Frappe app name |
| `YOUR_ELASTIC_IP` | EC2 Elastic IP |
| `YOUR_EMAIL@example.com` | Let's Encrypt contact email |
| `YOUR_PRIVATE_REPOSITORY` | Private custom-app Git repository |

## Security rules

- Do not use the AWS root user for daily administration.
- Require MFA for privileged AWS identities.
- Never place a GitHub token inside an image, Dockerfile, `apps.json`, repository or shell history.
- Never commit `.env`, `.netrc`, database passwords or site backups.
- Keep HTTP 80 and HTTPS 443 public; restrict SSH 22 to known IPs or avoid SSH and use Session Manager.
- Keep at least one client-controlled administrator and client-controlled recovery email.
- Test backups by performing periodic restores to a separate environment.

## Official references

- [Frappe Docker](https://github.com/frappe/frappe_docker)
- [Frappe Docker build setup](https://github.com/frappe/frappe_docker/blob/main/docs/02-setup/02-build-setup.md)
- [Frappe Docker ARM64 guide](https://github.com/frappe/frappe_docker/blob/main/docs/01-getting-started/03-arm64.md)
- [Frappe Docker TLS/SSL setup](https://github.com/frappe/frappe_docker/blob/main/docs/03-production/01-tls-ssl-setup.md)
- [AWS Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [AWS Elastic IP addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

## Scope warning

Version tags, container images, AWS pricing and upstream Compose files change. Pin versions, read release notes and test a new image before production deployment. Do not blindly substitute `latest`.
