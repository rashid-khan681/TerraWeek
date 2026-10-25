# Day 03: Providers, Resources & Your First Cloud Infra

## Task 1: Providers & Version Pinning

Configured the AWS provider with strict version pinning to ensure environment consistency.

```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
provider "aws" {
  region = var.region
}
```

**Q: Why does version pinning matter and what does the `~>` operator do?**
According to the official Terraform documentation, providers and Terraform Core are released independently. Version pinning (`required_providers`) ensures that a configuration strictly uses known, compatible versions of provider plugins, preventing unexpected infrastructure drift or breaking changes caused by automatic major version upgrades.
The `~>` (pessimistic constraint) operator allows only the rightmost version component to increment. For example, `~> 5.0` allows `5.1.0` or `5.100.0` (minor/patch updates) but strictly rejects `6.0.0` (major breaking changes).

**Bonus: When would you use a second provider alias?**
A provider alias (`alias = "secondary"`) is required when managing resources across multiple regions (e.g., `us-east-1` and `ap-south-1`) or multiple AWS accounts within the same Terraform configuration file. Resources explicitly attach to the alias using the `provider = aws.secondary` argument.

## Task 2: Resources vs Data Sources

Implemented both constructs in the configuration:
-   **Resource:** `resource "aws_vpc" "main"`
-   **Data Source:** `data "aws_availability_zones" "available"`

**Q: Explain the difference between resources and data sources.**
-   **Resources** are the most important element in the Terraform language. Each resource block describes one or more infrastructure objects, such as virtual networks or compute instances. Terraform is responsible for creating, updating, and destroying these elements to match the configuration.
-   **Data Sources** allow data to be fetched or computed for use elsewhere in Terraform configuration. They act strictly as read-only queries against an external API or existing infrastructure, without creating or modifying any underlying resources.

## Task 3: Provision a Cloud Stack

Successfully provisioned a complete AWS networking and compute stack:
-   Virtual Private Cloud (VPC)
-   Public Subnet
-   Internet Gateway (IGW)
-   Route Table & Association
-   Security Group (HTTP 80, SSH 22)
-   EC2 Instance (Amazon Linux 2023 via Data Source)

**Verification (`terraform state list`):**
```text
aws_instance.web
aws_internet_gateway.igw
aws_route_table.public_rt
aws_route_table_association.public_assoc
aws_s3_bucket.app_logs
aws_security_group.web_sg
aws_subnet.public
aws_vpc.main
```

## Task 4: Meta-Arguments in Action

Terraform meta-arguments modify the default behavior of resources:

1.  **`depends_on` (Explicit Ordering):** Used in the `aws_s3_bucket` block. It overrides Terraform's implicit dependency graph, explicitly forcing the S3 bucket creation to pause until the EC2 instance is fully initialized.
2.  **`lifecycle` (Custom Operations):** Used in the `aws_instance` block (`create_before_destroy = true`). When replacing the EC2 instance due to an AMI update, this rule forced Terraform to provision the new instance completely before terminating the existing one, enabling zero-downtime deployments.
3.  **`count` (Iteration):** Accepts a whole number and creates that many identical instances of the resource or module. Best used when identical resources are required (e.g., creating 3 identical web servers).
4.  **`for_each` (Iteration):** Accepts a map or a set of strings, creating an instance for each item. Preferred over `count` because it indexes resources by a specific key rather than an integer index, preventing massive resource recreation if an item is removed from the middle of the list.

## Task 5: Update & Destroy

1.  **Update Event:** Modified the `instance_type` from `t2.micro` to `t3.small` by passing `prod` environment variables. Executing `terraform plan` confirmed an in-place update for the instance type without triggering resource destruction.
2.  **Destruction Event:** Executed `terraform destroy`. Terraform intelligently determined the reverse dependency graph and successfully destroyed all 8 resources to prevent orphaned infrastructure and ongoing billing.
