# Me 

Is it possible to use yaml as variable source in terraform?

# Claude

Terraform doesn't natively support YAML as a variable source, but there are a few clean ways to achieve it depending on your use case.

---

## Option 1 — `yamldecode()` + `file()` (most common)

The simplest approach. Read a `.yaml` file and decode it directly in your config:

```hcl
locals {
  config = yamldecode(file("${path.module}/config.yaml"))
}
```

```yaml
# config.yaml
vm_id: 200
node_name: pve
cores: 2
memory: 2048
network:
  bridge: vmbr0
  address: dhcp
```

Then reference it like any other local:

```hcl
resource "proxmox_virtual_environment_container" "ct" {
  vm_id     = local.config.vm_id
  node_name = local.config.node_name
  ...
}
```

---

## Option 2 — `for_each` over a YAML list (bulk resources)

Great for spinning up multiple resources from a single YAML file:

```yaml
# containers.yaml
- name: nginx
  vm_id: 201
  memory: 512
- name: redis
  vm_id: 202
  memory: 256
```

```hcl
locals {
  containers = yamldecode(file("${path.module}/containers.yaml"))
}

resource "proxmox_virtual_environment_container" "ct" {
  for_each  = { for ct in local.containers : ct.name => ct }

  vm_id     = each.value.vm_id
  node_name = "pve"
  ...
}
```

---

## Option 3 — Multiple YAML files with `fileset()`

Useful when each resource gets its own file (e.g. one YAML per container):

```hcl
locals {
  configs = {
    for f in fileset(path.module, "containers/*.yaml") :
    f => yamldecode(file("${path.module}/${f}"))
  }
}
```

---

## Option 4 — Pass YAML via `-var` at the CLI

If you want to feed YAML content as a string variable and decode it at runtime:

```hcl
variable "raw_config" {
  type = string
}

locals {
  config = yamldecode(var.raw_config)
}
```

```bash
terraform apply -var="raw_config=$(cat config.yaml)"
```

---

## What you can't do

- You **cannot** use a `.yaml` file as a native `terraform.tfvars` equivalent — only `.tfvars` and `.tfvars.json` are auto-loaded by Terraform
- There's no `-var-file` support for YAML natively — it only accepts HCL or JSON var files

---

**Option 1 or 2** cover the vast majority of real-world use cases. Combined with the `bpg/proxmox` provider, you could define all your Proxmox containers in a single YAML manifest and have Terraform provision them all — pretty clean setup.