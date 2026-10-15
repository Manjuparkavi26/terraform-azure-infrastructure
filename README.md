# Terraform-Azure-Infrastructure


Simple Terraform configuration that provisions a minimal Azure environment:
a resource group, virtual network + subnet, network security group (with an SSH rule),
public IP, network interface, and a single Ubuntu Linux VM. The AzureRM provider is pinned to v3.0.0.

## Quick summary
- Purpose: Create a small, example Azure infra for testing or learning Terraform + Azure.
- Main file: `main.tf` (contains all resources and a variable for SSH source CIDR).
- Note: `backend.tf` is present but empty — no remote state backend configured by default.

## Prerequisites
- Terraform (install from https://www.terraform.io). (This configuration uses the azurerm provider v3.0.0.)
- An Azure subscription and authentication:
  - Quick: `az login` (interactive) and `az account set --subscription <ID>`, or
  - For automation: create a service principal and set ARM_* environment variables:
    - ARM_CLIENT_ID, ARM_CLIENT_SECRET, ARM_SUBSCRIPTION_ID, ARM_TENANT_ID
- An SSH public key at `~/.ssh/id_rsa.pub` (or edit `main.tf` to point to a different file).
- Your public IP in CIDR form (for example `203.0.113.10/32`) to set `ssh_allowed_source`.

## Files to know
- `main.tf` — Terraform configuration: provider, variable validation, and resources (RG, VNet, Subnet, NSG, Public IP, NIC, VM).
- `backend.tf` — currently empty; add a backend config if you want remote state (recommended for teams).
- `.terraform.lock.hcl` — provider dependency lock file.
- `Dummy.pub` — a small public key file (likely placeholder).

## How to use (short)
1. Clone:
   git clone https://github.com/Manjuparkavi26/terraform-azure-infrastructure.git
   cd terraform-azure-infrastructure

2. Initialize providers:
   terraform init

3. Preview changes (replace with your IP in CIDR form):
   terraform plan -var 'ssh_allowed_source=203.0.113.10/32'

   Or export as env var:
   export TF_VAR_ssh_allowed_source="203.0.113.10/32"
   terraform plan

4. Apply:
   terraform apply -var 'ssh_allowed_source=203.0.113.10/32'

5. Cleanup:
   terraform destroy -var 'ssh_allowed_source=203.0.113.10/32'

## Important details & defaults in this repo
- Provider: `azurerm` pinned to `=3.0.0` (set in `main.tf`).
- Location used: `EAST US` (hard-coded in `main.tf`).
- VM:
  - Image: Ubuntu 22.04 LTS (Canonical).
  - Size: `Standard_D2as_v4`
  - Admin user: `adminuser`
  - SSH key: read from `~/.ssh/id_rsa.pub` via `file(pathexpand("~/.ssh/id_rsa.pub"))`.
- Network security: `ssh_allowed_source` variable requires an IPv4 CIDR and is validated in `main.tf` (regex + cidrhost check).

## Security & operational notes
- Because the azurerm provider is pinned to `=3.0.0`, upgrades may require config changes; review provider release notes before changing.
- `backend.tf` is empty — add a remote backend (e.g., Azure Storage account) before using in teams to avoid state conflicts.
- The SSH rule uses the `ssh_allowed_source` variable; do not leave it wide open (0.0.0.0/0).

## Troubleshooting tips
- If Terraform cannot authenticate: ensure `az login` succeeded or export ARM_* service principal environment variables.
- If `~/.ssh/id_rsa.pub` is missing:
  ssh-keygen -t rsa -b 4096 -C "you@example.com"
- If you want the VM public IP after apply, add an `output` in `main.tf` for the public IP resource `azurerm_public_ip.example`.

## Next improvements you might want
- Parameterize names, location, and VM size into variables.
- Add `outputs.tf` with useful outputs (public IP, VM id).
- Configure a remote `backend.tf` (Azure Storage or Terraform Cloud).
- Consider using modules to split infra (networking vs compute).
