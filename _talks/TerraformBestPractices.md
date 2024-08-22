---
title: "Terraform best practices - Simple enough which all projects need to include"
collection: talks
type: "Infrastructure as code"
permalink: /talks/TerraformBestPractices
venue: "Teraform, Azure"
location: "Teraform"
date: 2024-07-01
---


# 1. Output Information:
Provide outputs for key resource attributes so that they can be easily referenced after deployment. This can achieved by adding outputs whenever majob actions like creates are performed.

```tf
output "resource_group_id" {
  value = azurerm_resource_group.rg.id
}
```

# Use Variable

# Modularity
Modularity in Terraform is a practice that involves organizing your infrastructure into smaller, reusable modules. This approach enhances maintainability, promotes reusability, and simplifies the management of large and complex infrastructure.

## Terragrunt

Terragrunt is a tool that acts as a wrapper around Terraform, providing additional features to simplify the management of complex infrastructure setups, especially when dealing with multiple environments or shared configurations. Here's how Terragrunt can enhance modularity and configuration management:

lets organize how we can use Terragrunt. I will use an example and slowly build on top of it. We need to unify organization of Environment structure along with Functional Strucure. We will use Terragrunt to unify this and orchestrate multiple modular terraforms.

### Organizing Modularity

#### Environment Structure
* best-practices/infrastructure/live
* best-practices/infrastructure/live/dev/
* best-practices/infrastructure/live/prod/

#### Functional Structure
* best-practices/infrastructure/modules
* best-practices/infrastructure/modules/resource-group
* best-practices/infrastructure/modules/secure





DRY (Don't Repeat Yourself) Principle:
Shared Configuration: Terragrunt allows you to keep common configurations (like backend settings, provider configurations, or input variables) in one place, reducing duplication across multiple environments or modules.
Terragrunt Configuration Blocks: You can define common configurations in a terragrunt.hcl file and reuse them across different modules and environments.
Automated Environment Management:
Environment-Specific Overlays: Terragrunt makes it easy to create environment-specific configurations by overlaying environment-specific settings on top of shared settings. This means you only need to define what’s different in each environment.
Environment Directory Structure: Terragrunt encourages an organized directory structure, where each environment can have its own configuration files, simplifying environment management.
Module Dependency Management:
Automatic Dependency Resolution: Terragrunt can manage the dependencies between different Terraform modules. It ensures that modules are applied in the correct order and that dependencies are handled automatically.
Remote State Management:
Centralized State Management: Terragrunt simplifies remote state management by centralizing backend configuration. You can define the backend configuration once and apply it across all environments.
Locking and Concurrency: Terragrunt handles state locking and concurrency issues, ensuring that multiple people or systems can work on the same infrastructure without causing conflicts.
Consistent Command Execution:
Terraform Command Wrapper: Terragrunt wraps Terraform commands, allowing you to run the same command across multiple modules or environments consistently. This is useful for tasks like running terraform apply across all environments with a single command.



