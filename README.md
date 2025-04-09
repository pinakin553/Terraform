# Terraform
# Terraform: State Management & Core Commands

## 📂 How do you manage state for large-scale applications?

Managing Terraform state efficiently is critical in large projects. Keeping everything in a single state file can lead to several issues:

### ❌ Problems with a monolithic state file:
- Slow `plan`/`apply` times
- More frequent state conflicts (especially in team environments)
- Higher blast radius (a small change impacts a large set of resources)
- Harder rollback or recovery
- Poor modularization and separation of concerns

### ✅ Solution: Split your Terraform state

Splitting the state improves:
- Isolation
- Parallelism
- Manageability

### 🛠️ Steps to manage state efficiently:
1. Break monolithic `main.tf` code into modules/directories/resource-specific files.
2. Configure **separate backends** for each module.
3. Use `terraform state mv` to migrate resources from the old state to the new, split state.
4. Use **remote state data source** to link dependencies:
   - The **source module** should expose values using `output`.
   - The **destination module** should use a remote data source with a **custom backend** (same as source backend).
   - Use that data source to fetch the desired property.

---

## 📦 What are the primary commands in a Terraform workflow?

```bash
terraform init       # Initialize working directory, download providers, configure backend
terraform plan       # Show execution plan
terraform apply      # Apply changes to infrastructure
terraform destroy    # Tear down infrastructure
terraform state      # Manage Terraform state
terraform taint      # Force recreation of a resource

## 🎯 **How do you target specific resources in a plan or apply?
**
- Use the `-target` argument in your Terraform commands:

```bash
terraform plan -target=aws_instance.web
terraform apply -target=aws_instance.web


## 📄 W**hat does `terraform state list` do?**

- Lists all resources currently **tracked in the Terraform state file**.

```bash
terraform state list

## 🗑️** How do you remove a resource from state without deleting it?**

```bash
terraform state rm aws_s3_bucket.old_logs

