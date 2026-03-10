# Me

The new proxmox ve 9.1 can now pull containers straight from docker hub. How to use this functionality from terraform? The `telmate/terraform` provider does not seem to support it yet!

# Claude

You're right that `telmate/proxmox` doesn't support this yet, but the **`bpg/proxmox`** provider does. It has a dedicated `proxmox_virtual_environment_oci_image` resource. Here's how to use it end-to-end:

---

## Switch to `bpg/proxmox`

First, update your provider block:

```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = "~> 0.89"
    }
  }
}

provider "proxmox" {
  endpoint  = "https://your-proxmox-host:8006/"
  api_token = "terraform@pve!provider=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  insecure  = true  # only if using self-signed cert
  ssh {
    agent    = true
    username = "terraform"
  }
}
```

---

## Step 1 — Pull the OCI image

The `proxmox_virtual_environment_oci_image` resource manages OCI images pulled from registries using the PVE `oci-registry-pull` API. It pulls images and stores them as `.tar` files in Proxmox datastores.

```hcl
resource "proxmox_virtual_environment_oci_image" "nginx" {
  node_name    = "pve"
  datastore_id = "local"
  reference    = "docker.io/library/nginx:alpine"

  # Optional: custom filename (stored as .tar)
  file_name = "nginx-alpine.tar"

  # Optional: increase timeout for large images (seconds)
  upload_timeout = 600

  # Optional: replace if the image size changed in the datastore
  overwrite            = true
  overwrite_unmanaged  = false
}
```

The `reference` field takes a standard OCI image path. For Docker Hub images, prefix with `docker.io/`. For GHCR it would be `ghcr.io/...`, for Quay it's `quay.io/...`, etc.

---

## Step 2 — Create the LXC container from it

```hcl
resource "proxmox_virtual_environment_container" "nginx_ct" {
  description = "Nginx from Docker Hub"
  node_name   = "pve"
  vm_id       = 200

  # OCI containers must be unprivileged (privileged is a known bug)
  unprivileged = true

  operating_system {
    template_file_id = proxmox_virtual_environment_oci_image.nginx.id
    type             = "unmanaged"
  }

  initialization {
    hostname = "nginx-ct"
    ip_config {
      ipv4 { address = "dhcp" }
    }
  }

  network_interface {
    name   = "eth0"
    bridge = "vmbr0"
  }

  disk {
    datastore_id = "local-lvm"
    size         = 8
  }

  depends_on = [proxmox_virtual_environment_oci_image.nginx]
}
```

---

## Key things to know

- **`unprivileged = true` is required.** Creating privileged LXC containers from OCI images fails with an "Invalid argument" error, likely due to UID/GID mapping issues during the `vzcreate` process — this is a known upstream Proxmox limitation, not a provider bug.

- **`type = "unmanaged"`** is the correct OS type for OCI-sourced containers, since Proxmox doesn't know the distro ahead of time.

- **Updates aren't seamless.** Because all layers are squashed into one rootfs upon container creation, you can't update a container by swapping in a newer image — you need to destroy and recreate it, ideally with data stored in a bind mount.

- **This is still a tech preview** in Proxmox VE itself — for use cases requiring container orchestration or live migration, it's still recommended to run them inside a Proxmox QEMU VM.

