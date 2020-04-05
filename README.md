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
By typing yes in the "are you sure?" section you are applying the changes to Azure and your resources should be changed(created or deleted) accordingly.

## Terraform Templates



### Linux VM

```terraform
resource "azurerm_linux_virtual_machine" "vm-resource" {
    name                  = "some-vm-01"
    location              = "northeurope"
    resource_group_name   = azurerm_resource_group.terraform-rg.name
    network_interface_ids = [azurerm_network_interface.terraform-nic.id]
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

    # in case of ssh key set "disable_password_authentication = true"
    #admin_ssh_key {
    #    username   = "adminuser"
    #    public_key = file("~/.ssh/id_rsa.pub")


    boot_diagnostics {
        storage_account_uri = azurerm_storage_account.terraform-storageacc.primary_blob_endpoint
    }

    tags = {
        environment = "New Environment"
    }
}
```


### Resouce Group

```terraform
resource "azurerm_resource_group" "terraform-rg" {
    name     = "myResourceGroup"
    location = "northeurope"

    tags = {
        environment = "New Environment"
    }
}
```

### Virtual Network

```terraform
resource "azurerm_virtual_network" "terraform-vnet" {
    name                = "myVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "northeurope"
    resource_group_name = azurerm_resource_group.terraform-rg.name

    tags = {
        environment = "New Environment"
    }
}
```

### Subnet

```terraform
resource "azurerm_subnet" "terraform-subnet" {
    name                 = "mySubnet"
    resource_group_name  = azurerm_resource_group.terraform-rg.name
    virtual_network_name = azurerm_virtual_network.terraform-vnet.name
    address_prefix       = "10.0.2.0/24"
}
```

### Public IP

```terraform
resource "azurerm_public_ip" "terraform-publicip" {
    name                         = "myPublicIP"
    location                     = "northeurope"
    resource_group_name          = azurerm_resource_group.terraform-rg.name
    allocation_method            = "Dynamic"

    tags = {
        environment = "New Environment"
    }
}
```

### Network Security Group

```terraform
resource "azurerm_network_security_group" "terraform-nsg" {
    name                = "myNetworkSecurityGroup"
    location            = "northeurope"
    resource_group_name = azurerm_resource_group.terraform-rg.name
    
    security_rule {
        name                       = "SSH"
        priority                   = 1001
        direction                  = "Inbound"
        access                     = "Allow"
        protocol                   = "Tcp"
        source_port_range          = "*"
        destination_port_range     = "22"
        source_address_prefix      = "*"
        destination_address_prefix = "*"
    }

    tags = {
        environment = "New Environment"
    }
}
```

### Virtual Network Interface Card

```terraform
resource "azurerm_network_interface" "terraform-nic" {
    name                        = "new-nic"
    location                    = "northeurope"
    resource_group_name         = azurerm_resource_group.terraform-rg.name

    ip_configuration {
        name                          = "myNicConfiguration"
        subnet_id                     = azurerm_subnet.terraform-subnet.id
        private_ip_address_allocation = "Dynamic"
        public_ip_address_id          = azurerm_public_ip.terraform-publicip.id
    }

    tags = {
        environment = "New Environment"
    }
}

# Connect the security group to the network interface
resource "azurerm_network_interface_security_group_association" "example" {
    network_interface_id      = azurerm_network_interface.terraform-nic.id
    network_security_group_id = azurerm_network_security_group.terraform-nsg.id
}
```

### Storage Account with Random Generated ID

```terraform
resource "random_id" "randomId" {
    keepers = {
        # Generate a new ID only when a new resource group is defined
        resource_group = azurerm_resource_group.terraform-rg.name
    }
    
    byte_length = 8
}

resource "azurerm_storage_account" "terraform-storageacc" {
    name                        = "diag${random_id.randomId.hex}"
    resource_group_name         = azurerm_resource_group.terraform-rg.name
    location                    = "northeurope"
    account_replication_type    = "LRS"
    account_tier                = "Standard"

    tags = {
        environment = "New Environment"
    }
}
```