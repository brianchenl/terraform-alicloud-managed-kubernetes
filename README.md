Alibaba Cloud Managed Kubernetes Cluster Module  
terraform-alicloud-managed-kubernetes
======================================================

A terraform module used to launch a managed kubernetes cluster on Alibaba Cloud.

These types of the module resource are supported:

- [VPC](https://www.terraform.io/docs/providers/alicloud/r/vpc.html)
- [VSwitch](https://www.terraform.io/docs/providers/alicloud/r/vswitch.html)
- [EIP](https://www.terraform.io/docs/providers/alicloud/r/eip.html)
- [Nat Gateway](https://www.terraform.io/docs/providers/alicloud/r/nat_gateway.html)
- [Snat](https://www.terraform.io/docs/providers/alicloud/r/snat.html)
- [SLS Project](https://www.terraform.io/docs/providers/alicloud/r/log_project.html)
- [Managed Kubernetes](https://www.terraform.io/docs/providers/alicloud/r/cs_managed_kubernetes.html)

Usage
-----

This module used to create a managed kubernetes and it can meet several scenarios by specifying different parameters.

**NOTE:** This module using AccessKey and SecretKey are from `profile` and `shared_credentials_file`.
If you have not set them yet, please install [aliyun-cli](https://github.com/aliyun/aliyun-cli#installation) and configure it.

### 1 Create a new vpc, several new vswitches and a new nat gateway for the cluster.
```hcl
module "managed-k8s" {
  source             = "terraform-alicloud-modules/managed-kubernetes/alicloud"
  
  k8s_name_prefix = "my-managed-k8s-with-new-vpc"
  new_vpc         = true
  vpc_cidr        = "192.168.0.0/16"
  vswitch_cidrs   = [
    "192.168.1.0/24",
    "192.168.2.0/24",
    "192.168.3.0/24",
    "192.168.4.0/24",
  ]
}
```
In this scenario, the module will create a new vpc with `vpc_cidr`, several vswitches with `vswitch_cidrs`, a new nat gateway, 
a new EIP with `new_eip_bandwidth` and several snat entries for vswitches.
  
### 2 Using existing vpc and vswitches by specifying `vswitch_ids`. Setting `new_nat_gateway=true` to add a new nat gateway in the vswitches' vpc.
```hcl
module "managed-k8s" {
  source             = "terraform-alicloud-modules/managed-kubernetes/alicloud"
  
  k8s_name_prefix = "my-managed-k8s-with-new-vpc"
  new_vpc         = false
  vswitch_ids     = [
    "vsw-12345678",
    "vsw-09876537"
  ]
  new_nat_gateway = true
}
```
In this scenario, if setting `new_nat_gateway=false`, you should ensure the specified vswitches can access internet.
In other words, the specified vpc has a nat gateway and there are several snat entries to bind the vswitches and a EIP.

## Conditional creation

This moudle can set [sls project](https://www.terraform.io/docs/providers/alicloud/r/log_project.html) config for this module

1. Create a new sls project with `cluster_addons`:  

    ```hcl
     cluster_addons = [
       {
         name   = "flannel",
         config = "",
       },
       {
         name   = "flexvolume",
         config = "",
       },
       {
         name   = "alicloud-disk-controller",
         config = "",
       },
       {
         name   = "logtail-ds",
         config = "{\"IngressDashboardEnabled\":\"true\"}",
       },
       {
         name   = "nginx-ingress-controller",
         config = "{\"IngressSlbNetworkType\":\"internet\"}",
       },
     ]
    ```

If you want to store kube config and other certificates after the cluster created, you can set the following parameters: 

1. Store kube config with `kube_config_path`:  

    ```hcl
    kube_config_path = "/home/xxx/.kube/config"
    ```

1. Store more certificates:  

    ```hcl-terraform
    client_cert_path     = "/home/xxx/.kube/client-cert.pem"
    client_key_path      = "/home/xxx/.kube/client-key.pem"
    cluster_ca_cert_path = "/home/xxx/.kube/cluster-ca-cert.pem"
    ```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| new_vpc  | Create a new vpc for this module  | string  | false | no  |
| vpc_cidr  | The cidr block used to launch a new vpc | string  | "192.168.0.0/16"  | no  |
| vswitch_ids  | List Ids of existing vswitch  | string  | []  | yes  |
| vswitch_cidrs  | List cidr blocks used to create several new vswitches when 'new_vpc' is true | string  | ["192.168.1.0/24"]  | yes  |
| availability_zones | List available zone ids used to create several new vswitches when 'vswitch_ids' is not specified. If not set, data source `alicloud_zones` will return one automatically. | list | [] | no |
| new_eip_bandwidth | The bandwidth used to create a new EIP when 'new_vpc' is true | int | 50 | no |
| new_nat_gateway | Seting it to true can create a new nat gateway automatically in a existing VPC. If 'new_vpc' is true, it will be ignored | bool | false|
| cpu_core_count | CPU core count is used to fetch instance types | int | 1 | no |
| memory_size | Memory size used to fetch instance types | int | 2 | no |
| worker_instance_types | The ecs instance type used to launch worker nodes. If not set, data source `alicloud_instance_types` will return one based on `cpu_core_count` and `memory_size` | list | ["ecs.n4.xlarge"] | no |
| worker_disk_category | The system disk category used to launch one or more worker nodes| string | "cloud_efficiency" | no |
| worker_disk_size | The system disk size used to launch one or more worker nodes| int | 40 |no |
| ecs_password | The password of work nodes | string | "Abc12345" | no |
| worker_number | The number of kubernetes cluster work nodes | int | 2 | no |
| k8s_name_prefix | The name prefix used to create managed kubernetes cluster | string | "terraform-alicloud-managed-kubernetes" | no |
| k8s_pod_cidr | The kubernetes pod cidr block. It cannot be equals to vpc's or vswitch's and cannot be in them. If vpc's cidr block is `172.16.XX.XX/XX`, it had better to `192.168.XX.XX/XX` or `10.XX.XX.XX/XX` | string | "172.20.0.0/16" | no |
| k8s_service_cidr | The kubernetes service cidr block. It cannot be equals to vpc's or vswitch's or pod's and cannot be in them. Its setting rule is same as `k8s_pod_cidr` | string | "172.21.0.0/20" | no |
| cluster_network_type | Network type, valid options are `flannel` and `terway` | string | "flannel" | no |
| new_sls_project | Create a new sls project for this module | bool | false | no |
| sls_project_name | Specify a existing sls project for this module | string | "" | no |
| kube_config_path | The path of kube config, like ~/.kube/config | string | "" | no |
| client_cert_path | The path of client certificate, like ~/.kube/client-cert.pem | string | "" | no |
| client_key_path | The path of client key, like ~/.kube/client-key.pem | string | "" | no |
| cluster_ca_cert_path | The path of cluster ca certificate, like ~/.kube/cluster-ca-cert.pem | string | "" | no |

## Outputs

| Name | Description |
|------|-------------|
| this_k8s_id | The ID of managed kubernetes cluster |
| this_k8s_name | The name of managed kubernetes cluster |
| this_k8s_nodes | List worker nodes of managed kubernetes cluster |
| this_vpc_id  | The ID of VPC  |
| this_vswitch_ids  | List Ids of vswitches  |
| this_security_group_id  | ID of the Security Group used to deploy kubernetes cluster  |
| this_sls_project_name  | The sls project name used to configure cluster |

Terraform version
-----------------
| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.12.0 |
| <a name="requirement_alicloud"></a> [alicloud](#requirement\_alicloud) | >= 1.77.0 |

## Notes
From the version v1.5.0, the module has removed the following `provider` setting:

```hcl
provider "alicloud" {
   profile = var.profile != "" ? var.profile : null
   shared_credentials_file = var.shared_credentials_file != "" ? var.shared_credentials_file : null
   region = var.region != "" ? var.region : null
   skip_region_validation = var.skip_region_validation
   configuration_source = "terraform-alicloud-modules/managed-kubernetes"
} 
```

If you still want to use the `provider` setting to apply this module, you can specify a supported version, like 1.4.0:

```hcl
module "managed-kubernetes" {
   source = "terraform-alicloud-modules/managed-kubernetes/alicloud"
   version     = "1.4.0"
   region      = "cn-hangzhou"
   profile     = "Your-Profile-Name"
   k8s_name_prefix = "my-managed-k8s-with-new-vpc"
   new_vpc         = true
   vpc_cidr        = "192.168.0.0/16"
}
```

If you want to upgrade the module to 1.5.0 or higher in-place, you can define a provider which same region with
previous region:

```hcl
provider "alicloud" {
   region  = "cn-hangzhou"
   profile = "Your-Profile-Name"
}
module "managed-kubernetes" {
  source = "terraform-alicloud-modules/managed-kubernetes/alicloud"
  k8s_name_prefix = "my-managed-k8s-with-new-vpc"
  new_vpc         = true
  vpc_cidr        = "192.168.0.0/16"
}
```
or specify an alias provider with a defined region to the module using `providers`:

```hcl
provider "alicloud" {
  region  = "cn-hangzhou"
  profile = "Your-Profile-Name"
  alias   = "hz"
}
module "managed-kubernetes" {
  source  = "terraform-alicloud-modules/managed-kubernetes/alicloud"
  providers = {
    alicloud = alicloud.hz
  }
   k8s_name_prefix = "my-managed-k8s-with-new-vpc"
   new_vpc         = true
   vpc_cidr        = "192.168.0.0/16"
}
```

and then run `terraform init` and `terraform apply` to make the defined provider effect to the existing module state.
More details see [How to use provider in the module](https://www.terraform.io/docs/language/modules/develop/providers.html#passing-providers-explicitly)

Authors
-------
Created and maintained by Alibaba Cloud Terraform Team(terraform@alibabacloud.com)

License
-------
Apache 2 Licensed. See LICENSE for full details.

Reference
---------
* [Terraform-Provider-Alicloud Github](https://github.com/terraform-providers/terraform-provider-alicloud)
* [Terraform-Provider-Alicloud Release](https://releases.hashicorp.com/terraform-provider-alicloud/)
* [Terraform-Provider-Alicloud Docs](https://www.terraform.io/docs/providers/alicloud/)
