# Day 01: Terraform Foundations and Infrastructure as Code

## Task 1: Understand Infrastructure as Code (IaC) and Terraform

**What is IaC?**
Infrastructure as Code (IaC) is the practice of managing and provisioning computing infrastructure through machine-readable definition files rather than interactive, manual configuration tools. It allows infrastructure to be version-controlled, shared, and reused.

**What is Terraform?**
Terraform is an open-source IaC software tool created by HashiCorp. It enables users to define and provision a datacenter infrastructure using a declarative configuration language known as HashiCorp Configuration Language (HCL). 

**Terraform vs. Other Tools:**
- **OpenTofu:** A community-driven, open-source fork of Terraform created after HashiCorp changed its licensing model.
- **Pulumi:** Uses familiar programming languages (Python, TypeScript, Go) for IaC, whereas Terraform uses domain-specific HCL.
- **CloudFormation:** AWS-native IaC tool; it only supports AWS resources. Terraform is cloud-agnostic and supports multiple providers.
- **Ansible:** A procedural configuration management tool designed to configure software inside existing servers. Terraform is a declarative provisioning tool designed to build the servers themselves.

## Task 2: Install Terraform
- **Installation:** Used Homebrew on macOS to install the latest version (`brew tap hashicorp/tap` and `brew install hashicorp/tap/terraform`).
- **Verification:** Successfully ran `terraform -version` (Output: `v1.15.x`).
- **Editor Setup:** Installed the official "HashiCorp Terraform" extension in VS Code for HCL syntax highlighting and auto-completion.

## Task 3: 6 Crucial Terraform Terminologies
1. **Provider:** A plugin that allows Terraform to interact with external APIs (like AWS, Azure, GCP, or even local files). Example: `provider "aws" { }`.
2. **Resource:** The most important element in the Terraform language. It describes one or more infrastructure objects (e.g., virtual networks, compute instances, or higher-level components like DNS records). Example: `resource "aws_instance" "web"`.
3. **State:** A file (`terraform.tfstate`) used by Terraform to map real-world resources to your configuration, keep track of metadata, and improve performance for large infrastructures.
4. **Plan:** A command (`terraform plan`) that creates an execution plan, letting you preview the changes Terraform will make before you apply them.
5. **HCL (HashiCorp Configuration Language):** The declarative, human-readable configuration language used to write Terraform configurations.
6. **Module:** A container for multiple resources that are used together. Modules allow you to create reusable, organized, and modular infrastructure code.

## Task 4: First Terraform Configuration (Local Provider)

Instead of using a cloud provider, this task demonstrates Terraform's core workflow using the `local` provider to manage a file on the local machine.

**Code (`main.tf`):**
```hcl
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version = "~> 2.0"
    }
  }
}

resource "local_file" "greeting" {
  filename = "greeting.txt"
  content  = "Hello Rashid Bhai! Welcome to TerraWeek Day 01."
}
```

**Execution Workflow:**
1. `terraform init`: Downloaded the `hashicorp/local` provider plugin.
2. `terraform fmt`: Automatically formatted the HCL code for consistent styling.
3. `terraform validate`: Checked the configuration for syntactical correctness.
4. `terraform plan`: Displayed that 1 resource (`local_file.greeting`) would be created.
5. `terraform apply`: Generated the `greeting.txt` file locally.
6. `cat greeting.txt`: Verified the output file content successfully.
7. `terraform destroy`: Safely removed the created text file to clean up the environment.

**Screenshots Required:**
- [Insert Screenshot: Terminal showing terraform apply output and the greeting.txt content]
