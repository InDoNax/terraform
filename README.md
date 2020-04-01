# Terraform - Azure Deployment Guide

This is a guide for my users to help them deploy Azure resources.


## Prerequisites

- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Terraform](https://learn.hashicorp.com/terraform/getting-started/install)



## Login to Azure



Login in to Azure CLI.

```bash
az login
```

To make sure you are logged in you can run this command.

```bash
az account show
```

The output should look like this:
```bash
{
  "environmentName": "AzureCloud",
  "homeTenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Subscription-Name",
  "state": "Enabled",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
  "user": {
    "name": "user@mail.com",
    "type": "user"
  }
}
```

Make sure you are on the correct subscription, if you want to change to a different subscription you can list all of them by running `az account list` picking a subscription name and then running.
```bash
az account set --subscription *name-of-sub-you-want*
```


## Terraform

If you have your Azure account ready now you can start working on the file.

With the Terraform file in your folder run.

```bash
terraform init
```
This will initialize your session and config files.

Removing objects from the terraform file means deleting them and adding objects means creating.

After you are done with your infrastructure descisions you can run.

```bash
terraform plan
```
This will run a simulation of what is going to change with the current configuration


Once you are satisfied with your decisions you can run.
```bash
terraform apply
```
By typing yes in the "are you sure?" section you are applying the changes to Azure and your resources should be changed(created or deleted).

## Terraform Templates

```terraform
resource "azurerm_linux_virtual_machine" "vm-resource" {
    name                  = "some-vm-01"
    location              = "northeurope"
    resource_group_name   = azurerm_resource_group.resource-group.name
    network_interface_ids = [azurerm_network_interface.network-interface.id]
    size                  = "Standard_DS1_v2"

    os_disk {
        name              = "myOsDisk"
        caching           = "ReadWrite"
        storage_account_type = "Standard_LRS"
    }

    source_image_reference {
        publisher = "Canonical"
        offer     = "UbuntuServer"
        sku       = "18.04-LTS"
        version   = "latest"
    }

    computer_name  = "tera-test-01"
    admin_username = "admin"
    disable_password_authentication = false
    admin_password = "Example123123!"

    boot_diagnostics {
        storage_account_uri = azurerm_storage_account.mystorageaccount.primary_blob_endpoint
    }

    tags = {
        environment = "New Environment"
    }
}
```


