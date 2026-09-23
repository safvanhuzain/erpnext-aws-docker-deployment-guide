# 3. VPC, security group and Elastic IP

## Default VPC or a new VPC

For a small first deployment, the AWS default VPC is acceptable if its public subnet, internet gateway and route table are intact. A dedicated VPC provides cleaner isolation but requires more networking knowledge.

The default security group is created automatically with a VPC. Do not use it for the ERPNext server. Create a dedicated security group with explicit rules.

## Create a security group

Example name: `erpnext-production-sg`.

Inbound rules:

| Type | Port | Source | Purpose |
|---|---:|---|---|
| HTTP | 80 | `0.0.0.0/0` | Let's Encrypt HTTP challenge and redirect |
| HTTPS | 443 | `0.0.0.0/0` | ERPNext web access |
| SSH | 22 | Administrator's current public IP `/32` | Optional emergency access |

Do not expose MariaDB `3306`, Redis `6379`, Frappe backend `8000` or Socket.IO `9000` publicly.

If all administration uses Session Manager, SSH can be omitted.

## Subnet and public IP

Choose a public subnet and enable auto-assigned public IPv4 during launch. After the instance works, allocate an Elastic IP and associate it with the instance.

An ordinary EC2 public IP can change after stop/start. The associated Elastic IP remains stable until it is disassociated or released. AWS charges for public IPv4 addresses; confirm current regional pricing.

## DNS

Create an `A` record:

```text
app.example.com -> YOUR_ELASTIC_IP
```

Do not create an `AAAA` record unless the server is correctly configured for IPv6.

Verify:

```bash
dig +short A app.example.com
dig +short AAAA app.example.com
```

The first command must return the Elastic IP. The second should return nothing for an IPv4-only deployment.
