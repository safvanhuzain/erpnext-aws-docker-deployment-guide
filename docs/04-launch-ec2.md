# 4. Launch the EC2 instance

Open **EC2 → Instances → Launch instances**.

## Recommended selections

| Setting | Value |
|---|---|
| Name | `erpnext-production` |
| AMI | Ubuntu Server 24.04 LTS, 64-bit ARM |
| Instance type | `t4g.medium` |
| Key pair | Optional when using Session Manager |
| VPC | Chosen/default VPC |
| Subnet | Public subnet |
| Auto-assign public IP | Enable |
| Security group | Existing `erpnext-production-sg` |
| Root volume | 50 GiB `gp3`, encrypted |
| IAM instance profile | `ERPNextEC2Role` |
| Shutdown behavior | Stop |
| Termination protection | Enable |
| Stop protection | Enable if operationally appropriate |
| Purchasing option | On-Demand (`None`) |
| Credit specification | Unlimited |

`Unlimited` permits a burstable T-family instance to consume surplus CPU credits and can add charges during sustained high CPU use. Monitor CPU credit balance and surplus charges. It does not create a large fixed charge by itself.

## Connect using Session Manager

1. Wait until the instance status checks pass.
2. Open the instance.
3. Select **Connect → Session Manager → Connect**.

If Session Manager is disabled, check:

- IAM role is attached
- `AmazonSSMManagedInstanceCore` is present
- SSM Agent is running
- Instance has outbound internet/NAT or SSM VPC endpoints

## Verify host resources

```bash
uname -m
cat /etc/os-release
df -h /
free -h
```

Expected architecture:

```text
aarch64
```

Create swap on a 4 GiB host if required by the chosen operational policy; monitor memory pressure rather than treating swap as additional RAM.
