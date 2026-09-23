# Security policy for this guide

This repository is documentation and example configuration, not a place to store operational secrets.

Never commit:

- `.env`
- `.netrc` or `github.netrc`
- GitHub personal access tokens
- AWS access keys
- Database or Administrator passwords
- Site backups
- `sites/*/site_config.json`
- Customer names, domains, IP addresses, account IDs or private repository URLs

If a credential is committed, revoke/rotate it immediately and remove it from Git history. Deleting it only from the latest commit is insufficient.

Use least privilege, MFA, client-controlled ownership, GitHub OIDC for AWS automation, encrypted backup storage and periodic restore tests.
