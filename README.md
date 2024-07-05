
#  Assignment-7

## Introduction

This repository contains the steps and commands used to complete various tasks related to Azure management using Azure CLI and PowerShell. The tasks include observing subscriptions, managing Azure Entra ID, creating users and groups, assigning RBAC roles, creating virtual machines, configuring policies, managing Azure Key Vault, and retrieving secrets.

## Table of Contents

1. [Observe Assigned Subscriptions](#1-observe-assigned-subscriptions)
2. [Observe or Create Azure Entra ID](#2-observe-or-create-azure-entra-id)
3. [Create Test Users and Groups](#3-create-test-users-and-groups)
4. [Assign a RBAC Role to a User and Test](#4-assign-a-rbac-role-to-a-user-and-test)
5. [Create a Custom Role and Assign it to Users](#5-create-a-custom-role-and-assign-it-to-users)
6. [Create a Virtual Machine and VNet from Azure CLI](#6-create-a-virtual-machine-and-vnet-from-azure-cli)
7. [Create and Assign a Policy at Subscription Level](#7-create-and-assign-a-policy-at-subscription-level)
8. [Create an Azure Key Vault and Store Secrets](#8-create-an-azure-key-vault-and-store-secrets)
9. [Configure Access Policies for the Key Vault](#9-configure-access-policies-for-the-key-vault)
10. [Retrieve Secret from Key Vault Using Azure CLI](#10-retrieve-secret-from-key-vault-using-azure-cli)
11. [Create a VM from PowerShell](#11-create-a-vm-from-powershell)

---

## 1. Observe Assigned Subscriptions

**Objective:** List the subscriptions associated with your Azure account.

**Command:**

```bash
az account list --output table
```

**Output:**

| Name                    | CloudName | SubscriptionId                        | State     | IsDefault |
|-------------------------|-----------|---------------------------------------|-----------|-----------|
| My Subscription Name    | AzureCloud | d3afb101-3193-4965-b42a-1952fb376496 | Enabled   | True      |

**Description:** This command lists all subscriptions available to your account, showing details like subscription name, ID, state, and default status.

---

## 2. Observe or Create Azure Entra ID

**Objective:** View or create an Azure Entra ID.

**Command to View:**

```bash
az ad signed-in-user show --query "{Name:displayName,Email:userPrincipalName}" --output table
```

**Output:**

| Name               | Email                           |
|--------------------|---------------------------------|
| John Doe           | johndoe@example.com             |

**Command to Create a New Azure Entra ID:**

```bash
az ad user create --display-name "Jane Doe" --password "P@ssw0rd1234" --user-principal-name janedoe@example.com --force-change-password-next-sign-in
```

**Output:**

```json
{
  "displayName": "Jane Doe",
  "id": "new-user-id",
  "userPrincipalName": "janedoe@example.com"
}
```

**Description:** This command creates a new Azure Entra ID user with specified display name, email, and password.



---

## 3. Create Test Users and Groups

**Objective:** Create users and groups for testing.

**Command to Create a User:**

```bash
az ad user create --display-name "Test User" --password "TestP@ssw0rd1234" --user-principal-name testuser@example.com
```

**Command to Create a Group:**

```bash
az ad group create --display-name "Test Group" --mail-nickname testgroup
```

**Command to Add User to Group:**

```bash
az ad group member add --group "Test Group" --member-id "testuser-id"
```

**Description:** These commands create a test user and a test group, then add the user to the group.



---

## 4. Assign a RBAC Role to a User and Test

**Objective:** Assign a predefined RBAC role to a user and verify.

**Command to Assign a Role:**

```bash
az role assignment create --role "Contributor" --assignee testuser@example.com
```

**Command to Verify Role Assignment:**

```bash
az role assignment list --assignee testuser@example.com --query "[].{Role:roleDefinitionName,Scope:scope}" --output table
```

**Output:**

| Role        | Scope                                       |
|-------------|---------------------------------------------|
| Contributor | /subscriptions/d3afb101-3193-4965-b42a-1952fb376496 |

**Description:** Assigns the "Contributor" role to the user and verifies the assignment.


---

## 5. Create a Custom Role and Assign it to Users

**Objective:** Create a custom RBAC role and assign it to users.

**Command to Create a Custom Role:**

```bash
az role definition create --role-definition '{
  "Name": "CustomRole",
  "Description": "Custom role with read and write access",
  "Actions": [
    "Microsoft.Compute/*",
    "Microsoft.Network/*"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/d3afb101-3193-4965-b42a-1952fb376496"]
}'
```

**Command to Assign the Custom Role:**

```bash
az role assignment create --role "CustomRole" --assignee testuser@example.com
```

**Description:** Creates a custom role with specified actions and assigns it to a user.


---

## 6. Create a Virtual Machine and VNet from Azure CLI

**Objective:** Create a virtual machine and a virtual network.

**Commands:**

```bash
# Create a Virtual Network
az network vnet create --resource-group MyResourceGroup --name MyVNet --subnet-name MySubnet

# Create a Public IP Address
az network public-ip create --resource-group MyResourceGroup --name MyPublicIP --sku Standard --allocation-method Static

# Create a Network Interface
az network nic create --resource-group MyResourceGroup --name MyNIC --vnet-name MyVNet --subnet MySubnet --public-ip-address MyPublicIP

# Create the Virtual Machine
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --location eastus \
  --nics MyNIC \
  --image Win2019Datacenter \
  --admin-username azureuser \
  --admin-password MyP@ssw0rd1234 \
  --size Standard_DS1_v2 \
  --output json
```

**Output:**

```json
{
  "fqdns": "",
  "id": "/subscriptions/d3afb101-3193-4965-b42a-1952fb376496/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/MyVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-4F-62-07",
  "powerState": "VM running",
  "privateIpAddress": "10.0.1.4",
  "publicIpAddress": "74.235.217.124",
  "resourceGroup": "MyResourceGroup",
  "zones": ""
}
```

**Description:** Creates a virtual network, public IP address, network interface, and virtual machine.


## 7. Create and Assign a Policy at Subscription Level

**Objective:** Create a policy and assign it at the subscription level.

**Command to Create a Policy Definition:**

```bash
az policy definition create --name "myPolicy" --display-name "My Policy" --description "A policy to deny all public IPs" --rules '{
  "if": {
    "field": "type",
    "equals": "Microsoft.Network/publicIPAddresses"
  },
  "then": {
    "effect": "deny"
  }
}' --mode All
```

**Command to Assign the Policy:**

```bash
az policy assignment create --policy "myPolicy" --scope "/subscriptions/d3afb101-3193-4965-b42a-1952fb376496"
```

**Description:** Creates a policy to deny the creation of public IP addresses and assigns it at the subscription level.


## 8. Create an Azure Key Vault and Store Secrets

**Objective:** Create a Key Vault and store secrets.

**Command to Create a Key Vault:**

```bash
az keyvault create --name MyKeyVault --resource-group MyResourceGroup --location eastus
```

**Command to Store a Secret:**

```bash
az keyvault secret set --vault-name MyKeyVault --name MySecret --value "SuperSecretValue123!"
```

**Description:** Creates a Key Vault and stores a secret in it.




---

## 9. Configure Access Policies for the Key Vault

**Objective:** Configure access policies for Key Vault.

**Command to Set Access Policies:**

```bash
az keyvault set-policy --name MyKeyVault --upn testuser@example.com --secret-permissions get list
```

**Description:** Configures the access policy for the Key Vault to allow a user to get and list secrets.



---

## 10. Retrieve Secret from Key Vault Using Azure CLI

**Objective:** Retrieve a stored secret.

**Command:**

```bash
az keyvault secret show --vault-name MyKeyVault --name MySecret --query value --output tsv
```

**Output:**

```
SuperSecretValue123!
```

**Description:** Retrieves the value of a secret stored in the Key Vault.


---

## 11. Create a VM from PowerShell

**Objective:** Create a virtual machine using PowerShell.

**Command:**

```powershell
New-AzVM `
  -ResourceGroupName "MyResourceGroup" `
  -Name "MyVM" `
  -Location "EastUS" `
  -VirtualNetworkId "/subscriptions/d3afb101-3193-4965-b42a-1952fb376496/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet" `
  -SubnetId "/subscriptions/d3afb101-3193-4965-b42a-1952fb376496/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet/subnets/MySubnet" `
  -PublicIpAddressId "/subscriptions/d3afb101-3193-4965-b42a-1952fb376496/resourceGroups/MyResourceGroup/providers/Microsoft.Network/publicIPAddresses/MyPublicIP" `
  -NetworkInterfaceId "/subscriptions/d3afb101-3193-4965-b42a-1952fb376496/resourceGroups/MyResourceGroup/providers/Microsoft.Network/networkInterfaces/MyNIC" `
  -ImageName "Win2019Datacenter" `
  -Credential (Get-Credential) `
  -Size "Standard_DS1_v2"
```

**Description:** Creates a virtual machine with specified configurations using PowerShell.

## Additional Information
All screenshots related to the solutions for the tasks are available in the Screenshots folder.







---

