# Day 02: HCL Deep Dive - Variables, Types & Expressions

## Task 1: Master HCL Syntax
Understanding the HashiCorp Configuration Language (HCL) is critical for writing reusable and maintainable infrastructure code.

**1. The Anatomy of a Block**
A block is a container for content. It is structured as:
`block_type "label_one" "label_two" { argument = value }`
*   **Example:** `resource "aws_vpc" "main" { cidr_block = "10.0.0.0/16" }`
*   Here, `resource` is the block type, `aws_vpc` is the provider-specific resource type (label 1), and `main` is the local identifier (label 2).

**2. Argument vs. Block**
*   **Argument:** Assigns a value to a particular name (e.g., `ami = "ami-12345"`). It does not have curly braces.
*   **Block:** A collection of arguments nested inside curly braces `{}`. Blocks can contain other blocks.

**3. Expressions**
Expressions represent a value.
*   **References:** Fetching a value from another resource: `aws_subnet.public.id`
*   **String Interpolation:** Injecting a value into a string using `${}` syntax: `"TerraWeek-VPC-${var.environment}"`

## Task 2: Variables, Types & Validation
Created `variables.tf` to make the configuration dynamic, defining primitive types (`string`, `bool`) and collections (`list(string)`).

**Implementation of Validation & Defaults:**
```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "environment must be one of: dev, staging, prod."
  }
}
```
*Official Concept:* Validation blocks ensure that user inputs meet specific criteria before Terraform even attempts to run an execution plan, preventing API failures down the line.

## Task 3: Locals, Outputs & Functions
**Locals:** Used to compute values once and reference them multiple times.
```hcl
locals {
  common_tags = {
    Project     = "TerraWeek"
    Environment = var.environment
  }
}
```

**Outputs:** Created `outputs.tf` to extract necessary information after deployment.
*   *Execution Output:* `server_public_ip = "13.201.36.6"`, `vpc_id = "vpc-04951540add890021"`

**Built-in Functions (Explored via `terraform console`):**
1.  **`upper()`:** `upper("terraweek")` resulted in `"TERRAWEEK"`.
2.  **`merge()`:** `merge({a=1}, {b=2})` combined two maps into `{ "a" = 1, "b" = 2 }`.
3.  **`join()`:** `join("-", ["tws", "terraweek", "2026"])` formatted a list into a single string `"tws-terraweek-2026"`.

## Task 4: Variable Precedence in Action
Instead of the Docker provider, we executed this task on our existing AWS infrastructure configuration to test how Terraform prioritizes variable inputs.

1.  **Default State:** The `environment` variable defaulted to `"dev"`.
2.  **Command Line Override (`-var`):** 
    Running `terraform plan -var="environment=prod"` successfully planned to change all resource tags from `"dev"` to `"prod"`.
3.  **File Override (`terraform.tfvars`):** 
    Created a `.tfvars` file assigning `environment = "staging"`. Running `terraform plan` automatically picked up this file and planned to change the tags to `"staging"`.

**Variable Precedence Rule (Highest to Lowest):**
1.  `-var` or `-var-file` command-line flags
2.  `*.auto.tfvars` or `*.auto.tfvars.json` files
3.  `terraform.tfvars` file
4.  Environment variables (`TF_VAR_name`)
5.  `default` value declared in the `variable` block
