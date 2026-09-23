# 12. Publish this documentation to GitHub

Recommended repository name:

```text
erpnext-aws-docker-deployment-guide
```

The documentation is sanitized, but review it again before publishing.

## Create the GitHub repository

1. Sign in to GitHub.
2. Select **New repository**.
3. Enter `erpnext-aws-docker-deployment-guide`.
4. Add a short description such as: `Step-by-step ERPNext deployment on AWS EC2 using Docker, ARM64 and HTTPS.`
5. Choose **Public** only if the final content contains no client information or credentials; otherwise choose **Private**.
6. Do not initialize it with another README, `.gitignore` or license because this folder already includes them.
7. Create the repository.

## Perform a final secret review

From the documentation folder:

```bash
git grep -nEi '(password|token|secret|access.key|private.key|account.id|[0-9]{1,3}(\.[0-9]{1,3}){3})'
```

This intentionally produces some matches because the guides discuss passwords and tokens. Review every match and confirm that it is explanatory text or a placeholder—not a real value.

Also verify that no customer identifiers remain:

```bash
git grep -nEi '(CUSTOMER_NAME|REAL_DOMAIN|REAL_EMAIL|REAL_REPOSITORY_OWNER)'
```

Replace the patterns with any real identifiers known to the project before running the check.

## Initialize and push

```bash
git init
git branch -M main
git add .
git status
git commit -m "docs: add ERPNext AWS Docker deployment guide"
git remote add origin https://github.com/YOUR-OWNER/erpnext-aws-docker-deployment-guide.git
git push -u origin main
```

Check `git status` and the staged diff before committing:

```bash
git diff --cached --stat
git diff --cached
```

## Protect the documentation

For a shared repository:

- Require pull requests for `main`.
- Enable secret scanning where available.
- Enable Dependabot only if executable dependencies are later added.
- Require review for operational command changes.
- Use issues to track unverified version updates.
- Add a release/tag when a deployment guide version has been tested.

## Update policy

Before changing pinned Frappe, ERPNext, MariaDB, Redis or Traefik versions:

1. Review official release notes.
2. Test the complete build and restore process.
3. Update the relevant guide.
4. Record the tested versions and date.
5. Submit the change through review.
