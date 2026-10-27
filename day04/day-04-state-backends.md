# TerraWeek Day 4 — State & Remote Backends (Native Locking)

## Task 1: Why State Matters

According to the official Terraform documentation, state management is the most critical component for team collaboration and safe infrastructure deployment.

**Questions and Official Answers:**
1.  **What is the terraform.tfstate file and what does it store?**
    Terraform uses state to map real world resources to your configuration, keep track of metadata, and improve performance for large infrastructures. The `.tfstate` file is a JSON formatted document that stores the bindings between the declarative HCL objects in your workspace and the physical cloud resources provisioned by Terraform.
2.  **Why should you never edit it by hand or commit it to Git?**
    -   *Manual Editing:* Modifying the JSON directly circumvents Terraform's validations, risking syntax corruption or state-to-infrastructure mismatch, which can lead to catastrophic deletion of production resources during the next apply.
    -   *Committing to Git:* State files contain sensitive data in plaintext. Pushing it to a remote VCS repository exposes this data to anyone with access to the repository.
3.  **What is state drift, and how does terraform plan / terraform refresh relate to it?**
    State drift occurs when real-world infrastructure is modified outside of Terraform's control (e.g., via AWS Console or CLI). `terraform refresh` queries the cloud provider APIs to update the state file with the current reality. `terraform plan` implicitly runs a refresh operation first, compares the updated state against your `.tf` configuration files, and highlights the drift.
4.  **Why is state sensitive?**
    Terraform state stores the complete attribute dataset of all managed resources in plaintext. This includes inherently sensitive values such as initial database passwords, generated private SSH keys, access tokens, and other secrets necessary for infrastructure provisioning.

## Task 2: Explore Local State & terraform state

Below are the commands used to inspect and manipulate the state without destroying real infrastructure, based on official HashiCorp definitions:

-   **`terraform state list`**: Used to list all resources within a Terraform state. It outputs a plain text list of resource addresses, proving what Terraform currently manages.
-   **`terraform state show <resource_address>`**: Used to output the attributes of a single resource in the Terraform state. It displays every computed and defined attribute for inspection.
-   **`terraform state mv <src> <dest>`**: Used to move an item in a Terraform state to a new address. This is highly utilized during configuration refactoring (renaming resources or moving them to modules) to prevent destructive recreation.
-   **`terraform state rm <resource_address>`**: Used to remove an item from the Terraform state. The physical cloud resource remains running in AWS, but Terraform relinquishes management control over it.
-   **`terraform show`**: Used to provide human-readable output from a state or plan file. It prints the entire state file in an easy-to-read format rather than raw JSON.

## Task 3: Bootstrap the Backend Infrastructure

To enable remote state, the S3 bucket must exist prior to initializing the remote backend. A separate localized Terraform configuration (`backend_infra/main.tf`) was utilized to bootstrap this bucket (`terraweek-state-rashid-2026`) with versioning, server-side encryption, and strict public access blocks.

*(Reference: [task-03-bootstrap-bucket.png](task-03-bootstrap-bucket.png))*

## Task 4: Configure the Remote Backend with Native Locking

Transitioned from local state to a remote AWS S3 backend utilizing the modern Terraform 1.11+ methodology. Native S3 state locking (`use_lockfile = true`) was enabled, entirely bypassing the deprecated DynamoDB table requirement.

```hcl
terraform {
  backend "s3" {
    bucket       = "terraweek-state-rashid-2026"
    key          = "day04/backend_demo/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true   # Native S3 state locking (No DynamoDB)
  }
}
```
Upon executing `terraform init`, the local state was successfully migrated to the secure S3 bucket. Simulating concurrent operations triggered a native lock, verifying that `.tflock` mechanism operates correctly.

*(Reference: [task-04-state-migrated.png](task-04-state-migrated.png) and [task-03-lock-error.png](task-03-lock-error.png))*

## Task 5: Import an Existing Resource

Manually provisioned an AWS S3 bucket and adopted it into the Terraform state utilizing the declarative 1.5+ `import {}` block.

```hcl
import {
  to = aws_s3_bucket.imported
  id = "terraweek-import-rashid-2026"
}
```
Executing `terraform plan -generate-config-out=generated.tf` successfully queried the AWS API and automatically scaffolded the HCL resource code, seamlessly bridging existing cloud assets into IaC management.

*(Reference: [task-04-import-plan.png](task-04-import-plan.png) and [task-04-import-list.png](task-04-import-list.png))*
