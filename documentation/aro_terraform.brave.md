Deploy ARO with Terraform

**Deploying Azure Red Hat OpenShift (ARO) using Terraform** is fully supported and widely adopted for enterprise-grade, automated infrastructure provisioning.

### Prerequisites
Before deploying ARO with Terraform, ensure you have:
- **Terraform CLI** (version 1.0 or later)
- **Azure CLI** installed and authenticated (`az login`)
- **Red Hat pull secret** from the [Red Hat Customer Portal](https://console.redhat.com/openshift/install/azure/installer-provisioned)
- An **Azure subscription** with contributor rights
- Required **resource providers** registered:
  ```bash
  az provider register --namespace Microsoft.RedHatOpenShift --wait
  az provider register --namespace Microsoft.Compute --wait
  az provider register --namespace Microsoft.Storage --wait
  az provider register --namespace Microsoft.Authorization --wait
  ```

### Terraform Configuration
Use the `azurerm` provider to manage ARO resources. A typical configuration includes:
- **Resource Group**
- **Virtual Network** with dedicated subnets for master and worker nodes
- **Network Security Groups (NSG)** to allow inbound access on ports 6443 (API) and 443 (Ingress)
- **Service Principal** for ARO cluster identity
- **Azure Red Hat OpenShift Cluster** resource with cluster, network, and node profile settings

Example snippet:
```hcl
resource "azurerm_redhat_openshift_cluster" "aro_cluster" {
  name                = "cluster-1"
  resource_group_name = "aro1"
  location            = "northeurope"
  cluster_profile {
    domain           = "mydomain.com"
    version          = "4.17.27"
    pull_secret      = "<YOUR_RED_HAT_ARO_PULL_SECRET>"
    managed_resource_group_name = "aro1-vms"
  }
  network_profile {
    pod_cidr    = "10.128.0.0/14"
    service_cidr = "172.30.0.0/16"
  }
  master_profile {
    vm_size = "Standard_D8s_v3"
    subnet_id = azurerm_subnet.control_plane_subnet.id
  }
  worker_profile {
    vm_size           = "Standard_D4s_v3"
    disk_size_gb      = 128
    node_count        = 3
    subnet_id         = azurerm_subnet.worker_subnet.id
  }
  service_principal {
    client_id     = azuread_application.my_aro_app.client_id
    client_secret = azuread_service_principal_password.sp_aro_app_pwd.value
  }
}
```

### Best Practices
- **Use Terraform Cloud or GitHub Actions** for CI/CD pipelines to enforce code quality and secure deployments.
- **Apply policies via Azure Policy as Code** to enforce compliance across environments.
- **Avoid direct `main` branch pushes**; use feature branches and pull requests with automated validation.
- **Use `azapi` provider** for newer ARO APIs (e.g., `Microsoft.RedHatOpenShift/openShiftClusters@2022-04-01`).

### References

- [Red Hat Developer: Deploy ARO with Terraform](https://developers.redhat.com/articles/2025/09/04/how-deploy-azure-red-hat-openshift-using-terraform)
- [Redhat blog: OpenShift 4.1 UPI environment deployment on Microsoft Azure Cloud](https://www.redhat.com/en/blog/openshift-4-1-upi-environment-deployment-on-microsoft-azure-cloud)
- [Redhat blog: Terraforming ARO with GitHub Actions](https://www.redhat.com/en/blog/terraforming-azure-red-hat-openshift-aro-with-github-actions)
- [Github: Deploy Azure Red Hat OpenShift via Terraform azurerm](https://gist.github.com/heoelri/de952e3bdf05eaa13f30e5554e4f82a5)
- [GitHub: Azure-Samples/aro-azapi-terraform](https://github.com/Azure-Samples/aro-azapi-terraform)
- [Github: luckylittle terraform-azure-aro](https://github.com/luckylittle/terraform-azure-aro)

> ✅ **Yes, you can create an Azure Red Hat OpenShift cluster in Terraform — and it's recommended for production deployments.**