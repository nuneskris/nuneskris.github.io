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

![image](https://github.com/user-attachments/assets/baf2d2db-b87d-4884-8173-ae677e02c972)


## Creating a Key Vault

```tf
# resource "azurerm_key_vault": This block creates an Azure Key Vault. A Key Vault is a secure place where you can store secrets, such as API keys, passwords, or connection strings.
# name = "kfnstudy-key-vault": Specifies the name of the Key Vault as kfnstudy-key-vault.
# location = azurerm_resource_group.rg.location: Specifies the location where the Key Vault will be created. It uses the location of the existing resource group (azurerm_resource_group.rg).
# resource_group_name = azurerm_resource_group.rg.name: Specifies the resource group where the Key Vault will be created. It uses the name of the existing resource group (azurerm_resource_group.rg).
# tenant_id = "c459871e-5392-4422-8fa5-bc1834d14b9a": Specifies the tenant ID associated with the Azure Active Directory (AAD) in which the Key Vault will be created. This is required for Azure AD-based identity and access management.
# sku_name = "standard": Specifies the SKU (pricing tier) for the Key Vault. The "standard" SKU is used here, which provides a good balance of features and cost.

resource "azurerm_key_vault" "azure_key_vault" {
  name                = "kfnstudy-key-vault"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  tenant_id           = "c459871e-5392-4422-8fa5-bc1834d14b9a"
  sku_name            = "standard"
}

# data "azurerm_key_vault_secret" "synapse_username": This block retrieves a secret from the Key Vault.
# name = "synapse-username": Specifies the name of the secret you want to retrieve. In this case, it's synapse-username, which would store the username used to access the Synapse Analytics workspace.
# key_vault_id = azurerm_key_vault.azure_key_vault.id: Specifies the ID of the Key Vault from which the secret is retrieved. It refers to the Key Vault created in the previous step (azurerm_key_vault.azure_key_vault).

data "azurerm_key_vault_secret" "synapse_username" {
  name         = "synapse-username" # Name of the existing secret
  key_vault_id = azurerm_key_vault.azure_key_vault.id
}

# data "azurerm_key_vault_secret" "sqlsynapseanalytics_password": Similar to the previous block, this retrieves another secret from the Key Vault.
# name = "sqlsynapseanalytics-password": Specifies the name of the secret you want to retrieve, in this case, sqlsynapseanalytics-password, which likely stores the password for the Synapse SQL Administrator account.
# key_vault_id = azurerm_key_vault.azure_key_vault.id: Again, this specifies the ID of the Key Vault from which to retrieve the secret, referencing the Key Vault created earlier.
data "azurerm_key_vault_secret" "sqlsynapseanalytics_password" {
  name         = "sqlsynapseanalytics-password" # Name of the existing secret
  key_vault_id = azurerm_key_vault.azure_key_vault.id
}
```
 ![image](https://github.com/user-attachments/assets/d815455d-598b-44b8-958e-c57459bf3e16)

 I updated the keys seperately.

## Creating of Synapse workspace

```tf
# resource "azurerm_storage_data_lake_gen2_filesystem": This block creates an Azure Data Lake Gen2 filesystem. Data Lake Gen2 is a set of capabilities dedicated to big data analytics built on top of Azure Blob storage. A filesystem in this context is similar to a container in traditional blob storage.
# name = "kfngen2filesystem": This specifies the name of the filesystem as kfngen2filesystem.
# storage_account_id = azurerm_storage_account.storage.id: This references the ID of the storage account where the filesystem will be created. The azurerm_storage_account.storage.id points to a previously created storage account resource.

resource "azurerm_storage_data_lake_gen2_filesystem" "kfngen2filesystem" {
  name               = "kfngen2filesystem"
  storage_account_id = azurerm_storage_account.storage.id
}

# resource "azurerm_storage_container": This block creates a container within the Azure Storage account. Containers are used to organize blobs within the storage account.
# name = "kfncontainer": Specifies the name of the storage container as kfncontainer.
# storage_account_name = azurerm_storage_account.storage.name: This references the name of the storage account in which the container will be created.
# container_access_type = "private": This specifies the access level for the container. Setting it to "private" ensures that the blobs within the container can only be accessed by authorized users.

resource "azurerm_storage_container" "synapse_container" {
  name                  = "kfncontainer"
  storage_account_name  = azurerm_storage_account.storage.name
  container_access_type = "private"
}

# resource "azurerm_synapse_workspace": This block creates an Azure Synapse Analytics workspace, which is a unified analytics platform that brings together big data and data warehousing.
# name = "kfnsynapsesnlyticsworkspace": Specifies the name of the Synapse workspace.
# resource_group_name = azurerm_resource_group.rg.name: Specifies the resource group in which the Synapse workspace will be created. It references the name of the existing resource group.
# location = azurerm_resource_group.rg.location: Specifies the Azure region where the Synapse workspace will be deployed.
# storage_data_lake_gen2_filesystem_id = azurerm_storage_data_lake_gen2_filesystem.kfngen2filesystem.id: This references the ID of the Data Lake Gen2 filesystem, linking the workspace to the storage account where the data will be stored.
# sql_administrator_login = data.azurerm_key_vault_secret.synapse_username.value: This sets the SQL administrator login for the Synapse workspace. The value is retrieved from a Key Vault secret (synapse-username) to keep credentials secure.
# sql_administrator_login_password = data.azurerm_key_vault_secret.sqlsynapseanalytics_password.value: This sets the SQL administrator login password for the Synapse workspace, also retrieved securely from a Key Vault secret.
# identity { type = "SystemAssigned" }: This enables a system-assigned managed identity for the Synapse workspace. A managed identity is automatically managed by Azure and can be used to authenticate to any service that supports Azure AD authentication without needing credentials.
# tags = { environment = "terraform" }: Tags are used to organize resources and apply metadata to them. In this case, a single tag environment = "terraform" is added to the Synapse workspace.

resource "azurerm_synapse_workspace" "kfn_synapse_workspace" {
  name                                 = "kfnsynapsesnlyticsworkspace"
  resource_group_name                  = azurerm_resource_group.rg.name
  location                             = azurerm_resource_group.rg.location
  storage_data_lake_gen2_filesystem_id = azurerm_storage_data_lake_gen2_filesystem.kfngen2filesystem.id
  sql_administrator_login              = data.azurerm_key_vault_secret.synapse_username.value
  sql_administrator_login_password = data.azurerm_key_vault_secret.sqlsynapseanalytics_password.value

  identity {
    type = "SystemAssigned"
  }

  tags = {
    environment = "terraform"
  }
}
```

![image](https://github.com/user-attachments/assets/11ea4fa3-d612-4364-9ad9-7601b69c1550)

## Azure Data Factory

```tf
resource "azurerm_data_factory" "adf" {
  name                = "kfnTFDataFactory"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  identity {
    type = "SystemAssigned"
  }

  tags = {
    environment = "terraform"
  }
}
resource "azurerm_data_factory_linked_service_azure_blob_storage" "blob_storage_linked_service" {
  name                = "kfnTFblobLinkedService"
  data_factory_id     = azurerm_data_factory.adf.id
  connection_string   = "DefaultEndpointsProtocol=https;AccountName=${azurerm_storage_account.storage.name};AccountKey=${azurerm_storage_account.storage.primary_access_key};EndpointSuffix=core.windows.net"
}
```

Data Factory and Data Set created.
![image](https://github.com/user-attachments/assets/1832ff2b-cab1-44a0-b5ab-563cba3cf790)

![image](https://github.com/user-attachments/assets/96258f1e-57fd-45d6-8efb-d88f5b1a1010)


