# Terraform — Interview Q&A

**JD: reusable modules, workspaces. Both are on your resume — be ready.**

---

## Q1. What is Terraform and why use it (IaC)?

- Terraform = **Infrastructure as Code**. You write infrastructure (VPC, EC2, EKS, S3) as code in `.tf` files, then apply it.
- **Why:** repeatable, version-controlled (Git), reviewable (like code PRs), consistent across environments, easy rollback/destroy.
- **Key idea:** declarative — you define the DESIRED state, Terraform makes it happen.

---

## Q2. Terraform workflow — core commands?

1. `terraform init` — download providers and modules.
2. `terraform plan` — show what WILL change (dry run, safe).
3. `terraform apply` — apply changes (also does a plan first).
4. `terraform destroy` — remove everything created.
5. `terraform fmt` / `terraform validate` — format and check syntax.

**Rule:** always run `plan` and review before `apply`. Never apply blindly on production.

---

## Q3. What is Terraform state? Why store it remotely?

- State = a JSON file recording what resources Terraform created (resource → real AWS resource ID).
- It's how Terraform knows what to update/destroy — **without state, Terraform doesn't know what it owns**.
- **Best practice:** remote backend — S3 bucket + **DynamoDB for state locking**:
  - S3 = shared state file (team works together).
  - DynamoDB = lock so two people can't apply at the same time.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state"
    key            = "env/prod/network.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "tf-lock"
  }
}
```

---

## Q4. How do you manage multiple environments (dev/stage/prod)?

**Three common ways:**
1. **Workspaces** (on your resume): same code, different state files. `terraform workspace new dev`, `terraform workspace select dev`. Downside: all state in one backend.
2. **Separate state files per environment (recommended):** same modules, different `key` in backend (`dev/vpc.tfstate`, `prod/vpc.tfstate`) and different tfvars files (`dev.tfvars`, `prod.tfvars`). Run `terraform apply -var-file=dev.tfvars`.
3. **Separate directories/repos per env:** full isolation, used in big orgs.

**Best practice:** same Terraform modules, one state per environment, env values via `.tfvars`. Prod changes go through code review + PR.

---

## Q5. How do you write a reusable Terraform module?

- A module is a folder with `.tf` files that takes **inputs (variables)** and gives **outputs**.
- Example: a VPC module with variables `cidr_block`, `azs`, `env`.

```hcl
module "vpc" {
  source      = "./modules/vpc"
  cidr_block  = "10.0.0.0/16"
  azs         = ["ap-south-1a", "ap-south-1b"]
  env         = "prod"
}
```

- **Outputs** pass values out (e.g., `vpc_id`, `subnet_ids`) so other resources use them.
- Modules = DRY — write once, use for dev/stage/prod.

---

## Q6. What is drift in Terraform?

- **Drift** = someone changed resources manually in AWS console, but Terraform state/code doesn't match.
- `terraform plan` shows the difference. `terraform apply` brings it back to code (converges).
- Import a resource created outside Terraform: `terraform import aws_s3_bucket.my_bucket bucket-name`.
- **Best practice:** make changes ONLY through Terraform to prevent drift.

---

## Q7. Terraform vs Ansible vs CloudFormation?

- **Terraform:** IaC for ALL clouds, declarative, has state. Best for provisioning AWS infra.
- **Ansible:** configuration management — installs software, configures servers, agentless (SSH/WinRM). Best for "what's installed on the server".
- **CloudFormation:** AWS-only IaC. Terraform is preferred — cloud-agnostic and more popular.
- **Together (common pattern):** Terraform creates VPC/EKS/servers; Ansible configures the servers/Windows (install agents, configure apps).

---

## Quick commands to remember

| Command | What it does |
|---|---|
| `terraform init` | Download providers and modules |
| `terraform plan` | Show what will change (nothing applied) |
| `terraform apply` | Apply the changes |
| `terraform destroy` | Remove everything |
| `terraform workspace new dev` | Create a dev workspace |
| `terraform apply -var-file=dev.tfvars` | Apply with dev values |
| `terraform import <address> <id>` | Import existing resource |