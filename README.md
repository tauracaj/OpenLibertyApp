# Open Liberty App
## NOTE: The Infrastructre for AKS can be split to multiple modules such as:
### Module for VNET
### Module for KEY VAULT
### etc..

## Addittional Note
- Locals and dev.auto.tfvars are left for local purposes


## Step-00: Prerequisites
- Create Github Action to create AKS using Terraform 
- Azure AD Directory Write permission for Service Principal
- Check Here - > for further details



## Step-01: Main
### 01-main.tf
- required_providers are commented as the this is created as a module
- Configure Terraform State Storage is commented as the this is created as a module

## Step-02: Variables
### 02-variables.tf
Important note: Minimum required variables to be set are:

| Name | Type | Description |
|------|--------|---------|
| location | string | Azure Region where all resources will be created |
| acr_id | string | ID of ACR |
| ssh_public_key | string | This variable defines the SSH Public Key for Linux k8s Worker nodes |
| address_space | string | his Variable defines node pool app |
```yaml
  address space {VNET_SPACE, GATEWAY_SUBNET, PRIVA_SUBNET, PUBA_SUBNET, BASTION_SUBNET, AKS_SUBNET}
  address_space = ["172.20.0.0/16","172.20.16.0/27","172.20.17.0/24","172.20.18.0/24","172.20.19.0/24","172.20.24.0/21"] 
```


Full Variable List

| Name | Type | Description |
|------|--------|---------|
| location | string | Azure Region where all resources will be created |
| resource_group_name | string | Name of Resource group |
| environment | string | Name of Environment |
| aks_name | string | This defines AKS Name. This variable will be comined with the Environment var to create the name |
| aks_umi_type | string | This defines identity type. Possible values: UserAssigned or SystemAssigned |
| owner | string | This is a variable used for TAG |
| department | string | This is a variable used for TAG |
| ssh_public_key | string | This variable defines the SSH Public Key for Linux k8s Worker nodes |
| linux_admin_username | string | This variable defines the Linux admin username k8s |
| nodepool_type_system | string | This Variable defines node pool label/tag type |
| nodepool_app | string | This Variable defines node pool label/tag app |
| nodepool_name | string | This Variable defines node pool label/tag name |
| aks_network_profile_plugin | string | This Variable defines Network Profile plugin type |
| lin_nodepool_name | string | This Variable defines node pool linux |
| nodepool_type_user | string | This Variable defines node pool linux type |
| lin_nodepool_vm_size | string | This Variable defines node pool  size |
| lin_nodepool_app | string | This Variable defines node pool app |
| lin_nodepool_os | string | This Variable defines node pool os |
| win_nodepool_vm_size | string | This Variable defines node pool size |
| win_nodepool_app | string | This Variable defines node pool app |
| win_nodepool_os | string | This Variable defines node pool os |
| networkrule | array | This Variable defines Network Rules | 
| networkruleappgw | array | This Variable defines Network Rules for app gw |
| address_space | string | This Variable defines node pool app |
| appgw_name | string | This Variable defines appgw name |
| appgw_sku_capacity | string | Application gateway's SKU capacity, Default is 2 |
| appgw_sku_name | string | Application gateway's SKU name, default is WAF_v2 |
| appgw_zones | string | Application gateway's Zones to use, default 1,2,3 |
| appgw_sku_tier | string | Application gateway's SKU tier, default is WAF_v2 |
| appgw_public_ip_allocation_method | string | This Variable defines appgw ip allocation method, default is static |
| appgw_frontend_port_name | string | This Variable defines appgw ip frontend port name, default is Port80 |
| appgw_ip_configuration_name | string | Name of the appgw gateway ip configuration, default is IPConfig |
| appgw_backend_address_pool_name | string | Name of the appgw gateway ip configuration, default is IPConfig |
| appgw_public_ip_sku | string | This Variable defines appgw ip Sku-Possible values: Standard and Basic, default is Standard |
| key_vault | string | This defines key vault name |


EXAMPLE FOR Network Rule:
  ```yaml
      networkrule = [
        {
          name                       = "test123"
          priority                   = 100
          direction                  = "Inbound"
          access                     = "Allow"
          protocol                   = "*"
          source_port_range          = "*"
          destination_port_range     = "*"
          source_address_prefix      = "*"
          destination_address_prefix = "*"
        }  
      ] 
```
EXAMPLE FOR Network Rule APPGW:
```yaml
  networkruleappgw = [
    {
      name                       = "Internet"
      priority                   = 100
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "65200-65535"
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    },
    {
      name                       = "test123"
      priority                   = 101
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "*"
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    } 
  ]
```
EXAMPLE FOR Address space:
```yaml
  address space {VNET_SPACE, GATEWAY_SUBNET, PRIVA_SUBNET, PUBA_SUBNET, BASTION_SUBNET, AKS_SUBNET}
  address_space = ["172.20.0.0/16","172.20.16.0/27","172.20.17.0/24","172.20.18.0/24","172.20.19.0/24","172.20.24.0/21"] 
```

## Step-03: Resource groups
### 03-resource-group.tf
- Creates a Resource group for AKS azurerm_resource_group 

## Step-04: AKS Versrions DS
### 04-aks-versions-datasource.tf
- We will get the latest version of AKS using this datasource. 
- `include_preview = false` will ensure that preview versions are not listed

## Step-05: Log Analytics Workspace
### 05-log-analytics-workspace.tf
- Log Analytics workspace will be created per environment. 
- Example Name:
  - logs-some-random-petname-{env}

## Step-06: AD group for Admins
### 06-aks-administrators-azure-ad.tf
- We are going to create Azure AD Group per environment for AKS Admins
- To create this group we need to ensure Azure AD Directory Write permission is there for our Service Principal (Service Connection) created in Azure DevOps

## Step-07: Create User Assigned
### 07-user-assigned-identity.tf
- Create User Assigned Managed Identity 

## Step-08: Virtual Network, Subnets and NSGs
### 08-virtual-network.tf
- Creates virtual network
- Creates subnet for Gateway
- Creates subnet Priv A
- Creates subnet Pub A
- Creates subnet AzureBastionSubnet
- Creates subnet AKS Subnet
- Creates NSG Priv A
- Creates NSG Pub A
- Creates NSG AKS
- Associates NSG PRIV A with Subnet Priv A
- Associates NSG Pub A with Subnet Pub A
- Associates NSG AKS A with Subnet AKS A
- Important note for NSGs is that here the security rules are created dynamically based on the number of rules you define on var.network rule variable

## Step-09: Virtual Network Subnet Datasources
### 09-virtual-network-subnet-datasources.tf
- Exposes datasource of Pub A subnet
- Exposes datasource of AKS subnet


## Step-10: Application Gateway
### 10-application-gateway.tf
- Creates Public IP
- Creates WAF Policy
- Create App Gw

## Step-11: Key Vault
### 11-key-vault.tf
- Creates Key Vault
- Creates key_vault_access

## Step-12: AKS cluster 
### 12-aks-cluster.tf
- Prerequsite for this is to generate SSH key for Linux
- Name of the AKS Cluster going to be **var.aks_name-var.environment**
- Example Names:
  - terraform-aks-qa
-  Node Lables and Tags will have a environment with respective environment name  

## Step-13: AKS cluster Linux User Node Pool
### 13-aks-cluster-linux-user-nodepools.tf
- Creates Linux User Node pool

## Step-15: Role Assignments
### 15-role-assignments.tf
- Assignes Network Contributor to aks identity uai on aks subnet
- Assignes Managed Identity Operator to aks identity
- Assignes Contributor to aks identity on App Gw
- Assignes Reader to aks identity on AKS Resource Group
- Assignes ACR PULL to aks uai on ACR
- Assignes ACR PUSH to aks uai on ACR
- Assignes AcrImageSigner to aks uai on ACR
- Assignes Monitoring Metrics Publisher to aks identity on AKS
- Assignes ACR PULL to aks kubelet identity on AKS

## Step-16: Outputs
### 16-outputs.tf 
- The output variables can be used when the module is executed standalone
- So when called from a root the output variables needs to be defined in the root.

- Output values
- Resource Group 
  - Location
  - Name
  - ID
  - Tags
- AKS Cluster 
  - AKS Versions
  - AKS Latest Version
  - AKS Cluster ID
  - AKS Cluster Name
  - AKS Cluster Kubernetes Version
- VNET 
  - Name
  - AKS Subnet Name
  - PRIV A Subnet Name
  - PUB A Subnet Name
- APP GW
  - ID
  - WAF POLICY
  - PUBLIC IP ID
- KEY VAULT
  - NAME  

