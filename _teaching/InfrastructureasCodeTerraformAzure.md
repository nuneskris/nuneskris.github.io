---
title: "Infrastructure as Code - Terraform for Azure"
collection: teaching
type: "Infrastructure"
permalink: /teaching/InfrastructureasCodeTerraformAzure
venue: "Azure"
location: "Terraform"
date: 2024-06-01
---


# Setup and Introduction

```console
az login
az account set --subscription "<my-subscription-id>"
mkdir learn-terraform-azure
terraform init
```

## create a main.tf

```tf
# Configure the Azure provider
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0.2"
    }
  }

  required_version = ">= 1.1.0"
}

resource "azurerm_resource_group" "rg" {
  name     = "kfnTFResourceGroup"
  location = "westus2"
}
```
## simple run
```console
terraform plan
terraform apply
terraform show
```

## creating a storage account

```tf
# Type: StorageV2 (supports both Blob and Data Lake Gen2 features)
# Hierarchical Namespace (HNS) Enabled: true (Data Lake Gen2 capabilities)
# Replication Type: LRS (Locally Redundant Storage)
# Primary Blob Endpoint: https://kfnstudytfstorageacct.blob.core.windows.net/
# Primary DFS Endpoint: https://kfnstudytfstorageacct.dfs.core.windows.net/
# Tags: { "environment" = "terraform" }

resource "azurerm_storage_account" "storage" {
  name                     = "kfnstudytfstorageacct"
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  blob_properties {
    delete_retention_policy {
      days = 7
    }
  }
  account_kind             = "StorageV2"

  enable_https_traffic_only = true
  is_hns_enabled           = true

  tags = {
    environment = "terraform"
  }
}
```
