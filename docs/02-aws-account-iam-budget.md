# 2. AWS account, IAM and budget

## Do not use root for normal work

Use the root user only for account-level recovery and tasks that explicitly require it. Enable root MFA and store recovery details with the client.

## Create administrative access

1. Open **IAM Identity Center**.
2. Enable it for the AWS account.
3. Create a user with a real individual email address.
4. Assign the user to the account with an administrator permission set during initial setup.
5. Sign out of root and sign in through the AWS access portal.
6. Require MFA before production handover.

Identity Center users do not replace or disable the root user. At least one client employee should retain administrator access.

## Configure a budget

In **Billing and Cost Management → Budgets**:

1. Create or review a monthly cost budget.
2. Configure alerts at useful thresholds such as 50%, 80% and 100%.
3. Send alerts to a client-controlled email address.

A budget sends notifications; it does not normally stop resources automatically.

## Create the EC2 role for Session Manager

1. Open **IAM → Roles → Create role**.
2. Select **AWS service** and **EC2**.
3. Attach `AmazonSSMManagedInstanceCore`.
4. Name it, for example, `ERPNextEC2Role`.
5. Attach the role to the EC2 instance during launch.

This enables browser-based Session Manager access without storing an SSH private key on every administrator's computer.

## Cost reminder

IAM users and roles do not have a separate hourly charge. The main costs are EC2, EBS storage, public IPv4/Elastic IP, snapshots, S3 backup storage, data transfer and optional monitoring services. Check the current AWS pricing pages for the selected region.
