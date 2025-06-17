# Terraform

### 📦 What are the primary commands in a Terraform workflow?

```bash
terraform init       # Initialize working directory, download providers, configure backend
terraform plan       # Show execution plan
terraform apply      # Apply changes to infrastructure
terraform destroy    # Tear down infrastructure
terraform state      # Manage Terraform state
terraform taint      # Force recreation of a resource
terraform show       # To analyse what exactly changed
```

### 📂 How do you manage state for large-scale applications?

Managing Terraform state efficiently is critical in large projects. Keeping everything in a single state file can lead to several issues:

#### ❌ Problems with a monolithic state file:
- Slow `plan`/`apply` times
- More frequent state conflicts (especially in team environments)
- Higher blast radius (a small change impacts a large set of resources)
- Harder rollback or recovery
- Poor modularization and separation of concerns

#### ✅ Solution: Split your Terraform state

Splitting the state improves:
- Isolation
- Parallelism
- Manageability

#### 🛠️ Steps to manage state efficiently:
- Break monolithic `main.tf` code into modules/directories/resource-specific files.
- Configure **separate backends** for each module.
- Use `terraform state mv` to migrate resources from the old state to the new, split state.
- Use **remote state data source** to link dependencies:
   - The **source module** should expose values using `output`.
   - The **destination module** should use a remote data source with a **custom backend** (same as source backend).
   - Use that data source to fetch the desired property.

### 🎯 How do you target specific resources in a plan or apply?

- Use the `-target` argument in your Terraform commands:

```bash
terraform plan -target=aws_instance.web
terraform apply -target=aws_instance.web
```


### 📄 What does `terraform state list` do?

- Lists all resources currently **tracked in the Terraform state file**.

```bash
terraform state list
```

### 🗑️ How do you remove a resource from state without deleting it?

```bash
terraform state rm aws_s3_bucket.old_logs
```

### How do you import an existing resource into the Terraform state?
```bash
terraform import aws_instance.web i-1234567890abcdef0.
```
You must define the resource in code first. Helps when adopting existing infrastructure into IaC.

### How do you validate your Terraform code?
```bash
terraform validate      # Check syntax and internal logic
terraform fmt -check    # Check formatting consistency
```

### How do you debug Terraform issues?
```bash
TF_LOG=DEBUG terraform apply
```

### What does terraform taint do?
```bash
terraform taint aws_instance.web
```
- Forces a resource to be destroyed and recreated on the next apply.
- Useful when the infrastructure is acting up and you suspect a "dirty" resource.

### What’s the difference between terraform plan -out and terraform apply <file>?
```bash
terraform plan -out=tfplan
terraform apply tfplan
```
Saves the plan for manual review, approval gates, or CI/CD pipelines.
Prevents drift between the plan and the apply.

# 🚀 Terraform AWS EC2 Module Example

This project demonstrates how to use a **custom Terraform module** to provision **multiple EC2 instances** with support for:

- `count`
- `connection`
- `provisioner` (`remote-exec`)
- `lifecycle` rules
- Dynamic `tags` and `SSH` access
- Module reusability

---

## 📁 Directory Structure

```
terraform-ec2/
├── main.tf
├── variables.tf
├── outputs.tf
├── modules/
│ └── ec2/
│ ├── main.tf
│ ├── variables.tf
│ └── outputs.tf
```

---

## Module: `modules/ec2/main.tf`

```
variable "instance_count" {
  type    = number
  default = 1
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "ami_id" {
  type = string
}

variable "key_name" {
  type = string
}

variable "tags" {
  type = map(string)
  default = {}
}

resource "aws_instance" "ec2" {
  count         = var.instance_count
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
  }

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt update -y",
      "sudo apt install -y nginx",
      "echo Hello from Terraform >> /tmp/welcome.txt"
    ]
  }

  tags = merge(var.tags, {
    Name = "Terraform-EC2-${count.index}"
  })
}

output "instance_ips" {
  value = aws_instance.ec2[*].public_ip
}
```
---

## Remote: `main.tf`
```
provider "aws" {
  region = "us-east-1"
}

module "web_servers" {
  source         = "./modules/ec2"
  instance_count = 2
  ami_id         = "ami-0c55b159cbfafe1f0"
  instance_type  = "t2.micro"
  key_name       = "my-key"

  tags = {
    Environment = "dev"
    Project     = "TerraformExample"
  }
}

