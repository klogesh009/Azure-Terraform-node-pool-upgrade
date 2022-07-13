# aks-default-node-pool-upgrader

This Terraform Module will upgrade the Kubernetes Version of your AKS default node pool.

## Tested Versions (at Jan. 28th 2021)

* Terraform 0.14.5
* AzureRM Provider 2.45.0

## Releated Article

On [Medium](https://dennis-riemenschneider.medium.com/how-to-upgrade-the-kubernetes-version-of-your-aks-default-node-pool-with-terraform-b7311d76041b)

## Requirements

### Terraform Authentication for the AzureRM Provider

The authentication must be setup for a Service Principal with the ARM_*** environment variables.

Find the documentation [here](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_secret)

```bash
$ export ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
$ export ARM_CLIENT_SECRET="00000000-0000-0000-0000-000000000000"
$ export ARM_SUBSCRIPTION_ID="00000000-0000-0000-0000-000000000000"
$ export ARM_TENANT_ID="00000000-0000-0000-0000-000000000000"
```

### Run Terraform as a container

Depending on the used Linux distribution and its SELinux configuration

you must adjust the volume mounts and/or the user to run the container as.

```bash
# with docker
alias terraform='docker run --env-file <(env | grep -v PATH) -it --rm -v $PWD:$PWD:rw,Z -w $PWD ghcr.io/300481/terraform:0.14.5'
# with podman
alias terraform='podman run --env-file <(env | grep -v PATH) -it --rm -v $PWD:$PWD:rw,Z -w $PWD ghcr.io/300481/terraform:0.14.5'
```

## Terraform Specs

### Providers

| Name | Version |
|------|---------|
| external | n/a |
| null | n/a |

### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| default\_pool\_name | The name of the default node pool. | `string` | n/a | yes |
| kubernetes\_cluster\_name | The name of the Kubernetes Cluster. | `string` | n/a | yes |
| resource\_group\_name | The name of the Resource Group which contains the Kubernetes Cluster. | `string` | n/a | yes |
| kubernetes\_node\_version | The Kubernetes Version for the default node pool. | `string` | `"empty"` | no |

### Outputs

No output.

## Usage

```
resource "azurerm_kubernetes_cluster" "kubernetes_cluster" {
  ...
  # your cluster definition
  ...
}

module "aks-default-node-pool-upgrader" {
  source  = "300481/aks-default-node-pool-upgrader/azurerm"
  version = "0.0.5"
  default_pool_name       = "default"
  kubernetes_cluster_name = azurerm_kubernetes_cluster.kubernetes_cluster.name
  resource_group_name     = "The_RG_of_your_cluster"
  kubernetes_node_version = "1.19.3"
}
```

### Prerequisites

Your Cluster must be created before using this module.

This can be achieved using the name attribute of your cluster resource or

the depends_on argument filled with the kubernetes_cluster resource.

Following Linux packages are needed:

* jq

* curl

When using the container image `ghcr.io/300481/terraform` they are already included.
