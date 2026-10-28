# TerraWeek Day 5: Modules - Reusable, Composable Infrastructure

## Task 1: Modules — the Why
**What is a module?**
A Terraform module is simply a container for multiple resources that are used together. Every Terraform configuration has at least one module, known as its root module. 
**Root vs Child Module:**
- **Root Module:** The working directory where you run `terraform apply`. It acts as the orchestrator.
- **Child Module:** The module called by another module to instantiate specific, reusable infrastructure components.

**Benefits:** Reusability, standardization, encapsulation of complexity, and version control.
**Core Files:** `main.tf` (resources), `variables.tf` (inputs), `outputs.tf` (returns), and `README.md`.

*(Reference: [task-01-module-structure.png](screenshots/task-01-module-structure.png))*

## Task 2: Write Your Own Module
Built a custom, reusable EC2 instance module. Following best practices, the module does not perform its own data lookups (like AMI or Subnets). Instead, it accepts these IDs as variables from the root module, ensuring it can be reused across different VPCs and environments.

```hcl
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids
  tags = merge(
    { Name = "${var.environment}-${var.instance_name}" },
    var.tags,
  )
}
```

*(Reference: [task-02-ec2-module.png](screenshots/task-02-ec2-module.png))*

## Task 3: Modular Composition (for_each)
Instantiated the same custom EC2 module multiple times (`web-server` and `api-server`) simultaneously using the `for_each` meta-argument. This keeps the root module DRY (Don't Repeat Yourself).

```hcl
module "servers" {
  source             = "./modules/ec2-instance"
  for_each           = toset(["web-server", "api-server"])
  instance_name      = each.key
  ami_id             = data.aws_ami.amazon_linux.id
  subnet_id          = module.vpc.public_subnets[0]
  security_group_ids = [module.web_sg.sg_id]
}
```

*(Reference: [task-04-apply-modules.png](screenshots/task-04-apply-modules.png))*

## Task 4: Consume a Registry Module + Version Locking
Replaced a manually constructed VPC network with the official, battle-tested `terraform-aws-modules/vpc/aws` module from the Terraform Registry. A single module block flawlessly provisioned 17 complex networking resources (Subnets, IGW, Route Tables, etc.).

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"
  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
}
```

*(Reference: [task-05-registry-module.png](screenshots/task-05-registry-module.png))*

## Task 5: Ways to Lock Module Versions
Pinning module versions is crucial for reproducible builds and avoiding surprise breaking changes in production.
1. **Registry (Pessimistic Constraint):** `version = "~> 5.0"` (Allows 5.1.x, but strictly prevents 6.0.0).
2. **Git Tag/Release:** `source = "git::https://github.com/org/repo.git//path?ref=v1.2.0"` (Locks to a specific release tag).
3. **Git Commit SHA:** `source = "git::https://github.com/org/repo.git//path?ref=4a3b...` (Ultimate immutability, locking to an exact commit).
