---
title: "10 Terraform best practices - Simple enough and all projects need to include"
collection: talks
type: "Infrastructure as code"
permalink: /talks/TerraformBestPractices
venue: "Teraform, Azure"
location: "Teraform"
date: 2024-07-01
---
![image](https://github.com/user-attachments/assets/c9017e3a-a2b2-42b9-8770-1c0cb59e72d0)

Every project need to have its infrastrcuture as code. I understand many do not have the budget for a comprehensive DevOPS. But using the below simple best practices which are simple to implement and brings much needed discipline into infrastructure management.

I will use Terraform on Azure. I hvae sandbox on using Terraform on Azure to build multiple components in this [page](https://nuneskris.github.io/teaching/InfrastructureasCodeTerraformAzure).

1. Display and Log Output Informaiton
2. Use Variables
3. Tags and Metadata
4. Seperate terraform, variables and output files
5. Modularity
6. Use Terragrunt to Orchestrate
7. Same Structure for all environments
8. Use consistent naming using input variables and string interpolation
9. Generate Backend and Provider Configuration
10. Document Code
   

# 1. Output Information
Provide outputs for key resource attributes so that they can be easily referenced after deployment. This can achieved by adding outputs whenever majob actions like creates are performed.

```tf
// Example output (modules/resource_group/outputs.tf)
output "resource_group_id" {
  value = azurerm_resource_group.rg.id
}
output "resource_group_name" {
  value = azurerm_resource_group.rg.name
}
```
![image](https://github.com/user-attachments/assets/ee54b602-d45b-4f18-a343-c0ef84b5a022)


# 2. Use Variable

```tf
variable "resource_group_name" {
  type        = string
  description = "The name of the resource group which will be provisioned for the data platform"
}

variable "location" {
  type        = string
  description = "The Azure location where the resource group should be created"
}
```
# 3 Tags and Metadata
Tags and metadata should be consistently applied to all resources for easier management, billing, and compliance tracking.

```hcl
tags = {
    environment = "dev"                        # Specifies the environment (e.g., dev, test, prod).
    project     = "kfn-study"                  # Identifies the project or organization.
    owner       = "krisnunes"                  # The owner or team responsible for the resource.
  }
```
![image](https://github.com/user-attachments/assets/d954ae88-fc82-4d97-a085-1728d8b31825)

# 4. Seperate terraform, variables and output files
![image](https://github.com/user-attachments/assets/9cec18ac-b5b1-4b8a-8579-eee0c1c42ee2)

# 5. Modularity
Modularity in Terraform is a practice that involves organizing your infrastructure into smaller, reusable modules. This approach enhances maintainability, promotes reusability, and simplifies the management of large and complex infrastructure. Keep environment-specific configurations (like this terragrunt.hcl) separate from the core Terraform modules to enhance reusability and maintainability.

![image](https://github.com/user-attachments/assets/ae030fd8-d379-48a5-bdcb-c42debb77a95)

# 6. Use Terragrunt
Terragrunt is a tool that acts as a wrapper around Terraform, providing additional features to simplify the management of complex infrastructure setups, especially when dealing with multiple environments or shared configurations. Here's how Terragrunt can enhance modularity and configuration management. lets organize how we can use Terragrunt. I will use an example and slowly build on top of it. We need to unify organization of Environment structure along with Functional Strucure. We will use Terragrunt to unify this and orchestrate multiple modular terraforms. Below shows how we can orchestrate a terraform file with selective values for variables based on the environment

<img width="612" alt="image" src="https://github.com/user-attachments/assets/64287fc0-9b24-4eef-a2a7-e21ee4f7481a">

***Parent Terragrunt***

```hcl
# best-practices/infrastructure/live/dev/terragrunt.hcl
inputs = {
  location            = "westus2"
  tenant_id           = "c459871e-5392-4422-8fa5-bc1834d14b9a"
  environment         = "dev"
  prefix              = "kfn-study"
}
```
***module Terragrunt***
```hcl
# best-practices/infrastructure/live/dev/resource_group/terragrunt.hcl
include {
  path = find_in_parent_folders()
}
terraform {
  source = "${get_terragrunt_dir()}/../../../../modules/resource_group"
}
inputs = {
  resource_type     = "rg"
}
```

# 7. Same Structure for all environments
Use a clean environment structure to seperate dev, prod etc.
* best-practices/infrastructure/live
* best-practices/infrastructure/live/dev/
* best-practices/infrastructure/live/prod/

### Functional Structure
* best-practices/infrastructure/modules
* best-practices/infrastructure/modules/resource-group
* best-practices/infrastructure/modules/secure

```tf
# best-practices/modules/resource_group/main.tf
resource "azurerm_resource_group" "rg" {
  name     = "${var.prefix}-${var.resource_type}-${var.environment}"
  location = var.location
  tags = {
    environment = "dev"                        # Specifies the environment (e.g., dev, test, prod).
    project     = "kfn-study"                  # Identifies the project or organization.
    owner       = "krisnunes"                  # The owner or team responsible for the resource.
  }
}

```

```tf
// Example output (modules/resource_group/outputs.tf)
output "resource_group_id" {
  value = azurerm_resource_group.rg.id
}
output "resource_group_name" {
  value = azurerm_resource_group.rg.name
}
```


```tf
// Example for a resource group module (modules/resource_group/variables.tf)  
variable "prefix" {
  description = "Prefix indicating the project or organization"
  type        = string
  default     = "kfn-study"
}

variable "resource_type" {
  description = "Resource type abbreviation (e.g., rg for Resource Group)"
  type        = string
  default     = "rg"
}

variable "environment" {
  description = "Specifies the environment (e.g., dev, test, prod)"
  type        = string
}

variable "location" {
  description = "Azure region where the resource group should be created"
  type        = string
}

variable "resource_group_name" {
  description = "Name of the resource group"
  type        = string
  default     = "" # You can either keep this empty or remove it if not required anymore
}

```


# 8. Use consistent naming
Consistent naming convention for resources in Terraform, can construct the name field using a combination of input variables and string interpolation. This allows us to dynamically create resource names that follow our specified naming convention. Here's how we can implement this for your azurerm_resource_group resource

```tf
resource "azurerm_resource_group" "rg" {
  name     = "${var.prefix}-${var.resource_type}-${var.environment}"
  location = var.location
}

```

# 9. Generate Backend and Provider Configuration
By generating the provider.tf and backend.tf files dynamically, you ensure that all environments use a consistent configuration, reducing the likelihood of configuration drift.
The backend configuration is critical for managing Terraform state files across different environments. The configuration ensures that state is stored securely and reliably.
```hcl

# The generate block dynamically creates a provider.tf file within the same directory. 
# This file configures the Azure provider (azurerm) for Terraform. 
# The if_exists = "overwrite" option ensures that the provider configuration is always up-to-date.
generate "provider" {
  path      = "provider.tf"                   # The name of the file to be generated.
  if_exists = "overwrite"                     # Ensures the file is overwritten if it already exists.
  contents  = <<EOF
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"           # Specifies the Azure provider from HashiCorp's registry.
      version = "~> 3.0.2"                    # Locks the provider version to avoid breaking changes.
    }
  }
  required_version = ">= 1.1.0"               # Ensures Terraform is at least version 1.1.0.
}

provider "azurerm" {
  features {}                                 # Enables all necessary provider features.
}
EOF
}
# The backend configuration is critical for managing Terraform state files across different environments. 
# The configuration ensures that state is stored securely and reliably.
generate "backend" {
  path      = "backend.tf"
  if_exists = "overwrite"
  contents  = <<EOF
terraform {
  backend "azurerm" {
    resource_group_name  = "kfn-study-rg-dev"
    storage_account_name = "kfnterrastorage" # Azure Storage Account for storing Terraform state.
    container_name       = "tfstate"          # The container within the storage account that will hold the state files.
    key                  = "terraform.tfstate"  #  The name of the state file for the current environment, ensuring that each environment has its own state file.
  }
}
EOF
}
```
