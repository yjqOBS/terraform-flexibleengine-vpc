# Flexible Engine VPC Terraform Module
---

Terraform module which creates VPC, subnets, NAT gateway resources and SNAT rules on Flexible Engine

## Terraform Version

> **Important Note regarding update from v1.2.0 to v1.3.0**

> If you use a NAT Gatewy and SNAT rules, switching module version from v1.2.0 to v1.3.0 will delete the current SNAT rule public IP and create a new one.
>
> Thus, ECSs outbound public IP will be replaced by a new one.
>
> But now, you can reserve public IP (EIP) thanks to terraform-flexibleegine-eip module and assign it to your SNAT rules with this terraform-flexibleegine-vpc module.

## Module Version

> **Important Notes regarding update module from v2.0.1 and earlier to v 2.1.0 and later**

> A compatibility break has been introduced in new module version 2.1.0. The subnets list is no more compute as a list but as a map.
> This will allow developpers to remove or add a subnet in the middle of the list and prevent the module from deleting and re-creating the subnets in the list after the added or removed item.

> A shell script `upgradTFState.sh` will help you in updating the Terraform state (modify subnet resource indexes by the CIDR of the subnets)
> Run this shell script only one time, right after the module version upgrade.
>
> Shell script available at: https://raw.githubusercontent.com/FlexibleEngineCloud/terraform-flexibleengine-vpc/upgrade-tfstate/upgradeTFState.sh

> After `upgradTFState.sh` please run a `terraform plan` in order to chekch the subnet indexes have been modified by the script. SNAT rules will have to be re-create because they are also based on CIDR index.
>
> You will have to run a `terraform apply` to re-create the SNAT rules. A short Internet access outage may be observed during the SNAT rules deletion and creation. You may also have to run twice the `terraform apply` command because SNAT rules creation may occure whereas SNAT rule deletion is not yet done.


## Usage : Terraform

```hcl
module "vpc" {
  source = "FlexibleEngineCloud/vpc/flexibleengine"
  version = "2.0.1"

  vpc_name = "my-vpc"
  vpc_cidr = "10.0.0.0/16"

  vpc_subnets = [
    {
      subnet_name       = "my-public-subnet-1"
      subnet_cidr       = "10.0.1.0/24"
      subnet_gateway_ip = "10.0.1.1"
    },
    {
      subnet_name       = "my-public-subnet-2"
      subnet_cidr       = "10.0.2.0/24"
      subnet_gateway_ip = "10.0.2.1"
    },
    {
      subnet_name       = "my-private-subnet"
      subnet_cidr       = "10.0.3.0/24"
      subnet_gateway_ip = "10.0.3.1"
    },
  ]

  vpc_snat_subnets = [
    "my-public-subnet-1",
    "my-public-subnet-2"
  ]

  enable_nat_gateway      = true
  new_eip                 = true
  eip_bandwidth           = 500
  nat_gateway_name        = "my-nat-gateway"
  nat_gateway_type        = "1"
  nat_gateway_subnet_name = "my-public-subnet-1"
}
```

## Usage : Terragrunt

```hcl
################################
### Terragrunt Configuration ###
################################

terraform {
  source = "FlexibleEngineCloud/vpc/flexibleengine"
  version = "2.0.1"
}

include {
  path = find_in_parent_folders()
}

##################
### Parameters ###
##################

inputs = {
  vpc_name = "vpc-main"
  vpc_cidr = "192.168.0.0/16"
  vpc_subnets = [
    {
      subnet_name       = "subnet-1"
      subnet_cidr       = "192.168.1.0/24"
      subnet_gateway_ip = "192.168.1.253"
    }
  ]
  vpc_snat_subnets = [
    "subnet-1"
  ]
  enable_nat_gateway      = true
  new_eip                 = true
  eip_bandwidth           = 500
  nat_gateway_name        = "nat-gateway-1"
  nat_gateway_type        = "1"
  nat_gateway_subnet_name = "subnet-1"
}

```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:-----:|
| create\_vpc | Controls if VPC should be created (it affects almost all resources) | `bool` | `true` | no |
| eip\_bandwidth | Bandwidth of the EIP in Mbit/s | `number` | n/a | yes |
| eip\_pool\_name | Name of eip pool | `string` | `"admin_external_net"` | no |
| enable\_nat\_gateway | Should be true if you want to provision NAT Gateways for your networks | `bool` | `false` | no |
| existing\_eip\_id | ID of an existing EIP | `string` | n/a | yes |
| nat\_gateway\_name | Name of the NAT gateway | `string` | `""` | no |
| nat\_gateway\_subnet\_name | Name of subnet used by the NAT Gateway | `string` | `""` | no |
| nat\_gateway\_type | Type of NAT gateway. 4 values (1,2,3,4). 1 is small type, and 4 the Extra-large | `number` | `"1"` | no |
| new\_eip | Whether or not attach new Elastic IP (public IP) to NAT Gateway | `bool` | `false` | no |
| primary\_dns | IP address of primary DNS | `string` | `"100.125.0.41"` | no |
| secondary\_dns | IP address of secondary DNS | `string` | `"100.126.0.41"` | no |
| vpc\_cidr | The CIDR for the VPC. Default value is a valid CIDR, but not acceptable by FlexibleEngine and should be overridden | `string` | `"0.0.0.0/0"` | no |
| vpc\_name | Name of the VPC to create | `string` | `"vpc-main"` | no |
| vpc\_snat\_subnets | json description of subnets included in SNAT rules | `list(string)` | `[]` | no |
| vpc\_subnets | json description of subnets to create | <pre>list(object({<br>    subnet_name       = string<br>    subnet_cidr       = string<br>    subnet_gateway_ip = string<br>  }))</pre> | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| gateway\_id | id of NAT gateway |
| network\_ids | list of IDs of the created networks |
| snat\_eip | The Public IP adress of the SNAT rule |
| subnet\_ids | list of IDs of the created subnets |
| vpc\_id | ID of the created vpc |

