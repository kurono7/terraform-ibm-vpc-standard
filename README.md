# terraform-ibm-vpc-standard

This module provisions a **standard, isolated network environment** on IBM Cloud VPC following company-approved architecture. All new services must use this module as the foundation for their networking layer.

---

## What this module creates

| Resource | Name pattern | Purpose |
|---|---|---|
| `ibm_is_vpc` | `<name>-vpc` | Isolated Virtual Private Cloud |
| `ibm_is_subnet` | `<name>-subnet` | Public subnet in the specified zone |
| `ibm_is_public_gateway` | `<name>-pgw` | Outbound internet access from the subnet |
| `ibm_is_security_group` | `<name>-sg` | Firewall rules for the VPC |
| `ibm_is_security_group_rule` | — | Allow inbound SSH (22), ICMP, and all outbound |

---

## Usage

```hcl
module "network" {
  source  = "app.terraform.io/ibm-jcueva-org/vpc-standard/ibm"
  version = "1.0.0.0"

  name = "payments-dev"
}
```

The `name` variable is used as a prefix for all resources. For example, `payments-dev` produces:
- `payments-dev-vpc`
- `payments-dev-subnet`
- `payments-dev-pgw`
- `payments-dev-sg`

---

## Inputs

| Name | Type | Default | Required | Description |
|---|---|---|---|---|
| `name` | `string` | — | **yes** | Base name prefix for all resources |
| `region` | `string` | `us-south` | no | IBM Cloud region |
| `zone` | `string` | `us-south-1` | no | IBM Cloud zone within the region |
| `subnet_cidr` | `string` | `10.240.0.0/24` | no | IPv4 CIDR block for the subnet |
| `allow_ssh_cidr` | `string` | `0.0.0.0/0` | no | CIDR allowed to reach port 22 |
| `resource_group_id` | `string` | `""` | no | IBM Cloud resource group ID |
| `tags` | `list(string)` | `[]` | no | Tags to attach to all resources |

---

## Outputs

| Name | Description |
|---|---|
| `vpc_id` | ID of the created VPC |
| `vpc_name` | Name of the created VPC |
| `subnet_id` | ID of the created subnet |
| `subnet_cidr` | CIDR block of the subnet |
| `security_group_id` | ID of the security group |
| `public_gateway_id` | ID of the public gateway |
| `zone` | Zone where resources were created |

---

## Company standards enforced by this module

- **One VPC per service** — no shared networks between services.
- **Isolated security group per environment** — inbound SSH and ICMP only; all other inbound traffic denied by default.
- **Public gateway for outbound** — instances can reach the internet without requiring a floating IP on every VM.
- **Naming convention** — all resources follow the `<service>-<environment>` prefix pattern (e.g. `payments-dev`, `payments-prod`).
- **Tagging** — use the `tags` variable to attach cost allocation and ownership tags.

---

## Example — full service environment

```hcl
module "network" {
  source  = "app.terraform.io/ibm-jcueva-org/vpc-standard/ibm"
  version = "1.0.0.0"

  name   = "inventory-prod"
  region = "us-south"
  zone   = "us-south-1"
  tags   = ["env:prod", "service:inventory", "team:platform"]
}

# Use module outputs to attach compute resources
output "vpc_id"           { value = module.network.vpc_id }
output "subnet_id"        { value = module.network.subnet_id }
output "security_group_id" { value = module.network.security_group_id }
```

---

## Requirements

| Name | Version |
|---|---|
| Terraform | `>= 1.5.0` |
| IBM Cloud provider (`IBM-Cloud/ibm`) | `>= 1.65.0` |

**Authentication:** Set the `IC_API_KEY` environment variable or use the `ibm-cloud-creds` variable set in Terraform Cloud.

---

## Versioning

This module follows [Semantic Versioning](https://semver.org/).

| Version | Changes |
|---|---|
| `1.0.0` | Initial release — VPC, Subnet, Public Gateway, Security Group |

---

## Maintainers

Platform Engineering team — open an issue or contact the team before modifying this module.
