# azure-iac-automation

## ✨ Description
Comprehensive examples of Infrastructure as Code (IaC) automation using Azure technologies such as Terraform, ARM Templates, Linked/Nested Templates, and Azure DevOps Pipelines with Azure CLI and DSC.

---

## 🚀 How to Run the Project

### ✅ Prerequisites
1. Azure Subscription
2. Azure DevOps Project
3. Terraform installed (locally or in pipeline)
4. Azure CLI installed
5. Service Principal credentials (Client ID, Secret, Tenant ID)

---

## 📂 Repository Folder Structure
```
azure-iac-automation/
|
|-- 01-publish-linked-template-artifact/
|   |-- azure-pipeline.yml
|
|-- 02-delete-resources-cli-pipeline/
|   |-- delete-resources.sh
|
|-- 03-create-resources-cli-pipeline/
|   |-- create-resources.sh
|
|-- 04-terraform-basic-config/
|   |-- main.tf
|   |-- variables.tf
|   |-- outputs.tf
|
|-- 05-terraform-add-delete-resources/
|   |-- main.tf
|
|-- 06-terraform-sql-server-db/
|   |-- main.tf
|
|-- 07-terraform-azure-pipeline-integration/
|   |-- azure-pipeline.yml
|
|-- 08-arm-appservice-deployment/
|   |-- appservice-template.json
|
|-- 09-arm-sqlserver-db-deployment/
|   |-- sqlserver-template.json
|
|-- 10-arm-storage-deployment/
|   |-- storage-template.json
|
|-- 11-linked-template/
|   |-- mainTemplate.json
|   |-- linkedTemplate.json
|
|-- 12-nested-template/
|   |-- nestedTemplate.json
|
|-- 13-arm-integration-release-part1/
|   |-- azure-release-pipeline.yml
|
|-- 14-arm-integration-release-part2/
|   |-- azure-release-pipeline.yml
|
|-- 15-azure-automation-dsc/
    |-- dsc-config.ps1
```

---

## 🌟 Step-by-Step Execution

### 01. Publish Linked Template as Artifact
- Define `azure-pipeline.yml`
```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: 'templates'
    ArtifactName: 'linked-templates'
    publishLocation: 'Container'
```
- Run the pipeline in Azure DevOps.

---

### 02. Delete Resources using Azure CLI in Release Pipeline
- Script: `delete-resources.sh`
```bash
az login --service-principal -u $SP_USER -p $SP_PASS --tenant $TENANT_ID
az group delete --name myResourceGroup --yes --no-wait
```
- Add Azure CLI task in release pipeline to run this script.

---

### 03. Create Resources using Azure CLI in Release Pipeline
- Script: `create-resources.sh`
```bash
az group create --name myResourceGroup --location eastus
az storage account create --name mystorageacct --resource-group myResourceGroup --location eastus --sku Standard_LRS
```

---

### 04. Terraform Basic Config
```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "tf-rg"
  location = "East US"
}
```
Run commands:
```bash
terraform init
terraform plan
terraform apply -auto-approve
```

---

### 05. Terraform Add/Delete Resources
```hcl
resource "azurerm_storage_account" "example" {
  name                     = "examplestoracc"
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

---

### 06. Terraform - Azure SQL Server & DB
```hcl
resource "azurerm_sql_server" "sql" {
  name                         = "sqlserverdemo"
  resource_group_name          = azurerm_resource_group.rg.name
  location                     = azurerm_resource_group.rg.location
  version                      = "12.0"
  administrator_login          = "adminsql"
  administrator_login_password = "Password123!"
}

resource "azurerm_sql_database" "sqldb" {
  name                = "mysqldb"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_sql_server.sql.location
  server_name         = azurerm_sql_server.sql.name
  sku_name            = "S0"
}
```

---

### 07. Terraform Azure Pipeline Integration
```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: TerraformInstaller@1
  inputs:
    terraformVersion: '1.5.0'
- script: |
    terraform init
    terraform plan
    terraform apply -auto-approve
  displayName: 'Terraform Apply'
```

---

### 08. Deploy App Service using ARM Template
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2021-02-01",
      "name": "myAppServicePlan",
      "location": "[resourceGroup().location]",
      "sku": { "name": "F1", "tier": "Free" },
      "properties": {}
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2021-02-01",
      "name": "myWebApp",
      "location": "[resourceGroup().location]",
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', 'myAppServicePlan')]"
      }
    }
  ]
}
```
Deploy using:
```bash
az deployment group create \
  --resource-group myRG \
  --template-file appservice-template.json
```

---

### 09. Deploy SQL Server & DB using ARM Template
- Similar to above, use `az deployment group create` with `sqlserver-template.json`

---

### 10. Deploy Storage Account using ARM Template
- Use `az deployment group create` with `storage-template.json`

---

### 11. Linked ARM Templates
- Deploy:
```bash
az deployment group create \
  --resource-group myRG \
  --template-file mainTemplate.json
```

---

### 12. Nested ARM Templates
- Nested templates embedded in main template file using `templateLink` or inline `template`

---

### 13 & 14. ARM Integration with Release Pipeline (Build & Release)
- Build pipeline: Publish templates as artifacts
- Release pipeline: Use `Azure Resource Group Deployment` task

---

### 15. Azure Automation DSC
PowerShell:
```powershell
Configuration SampleConfig {
  Node "localhost" {
    WindowsFeature IIS {
      Name   = "Web-Server"
      Ensure = "Present"
    }
  }
}
SampleConfig
```

Deploy via Azure:
```bash
az automation dsc node configure \
  --automation-account-name myAA \
  --configuration-name SampleConfig \
  --node-configuration-name SampleConfig.localhost
```

---

## 📘 Notes & Tips
- Use separate resource groups for each test/demo
- Always destroy infra post-demo using Terraform destroy or Azure CLI
- Store secrets in Azure Key Vault or DevOps library securely

---

Let me know if you want a ZIP or real GitHub repo initialized.

