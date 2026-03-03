# Terraform lab roadmap

## :point_right: Install Terraform :point_left:

```bash
sudo dnf install -y dnf-plugins-core
```
* __validate if present__
   ```bash
   sudo dnf list --available | grep -E "^terraform"
   ```
```bash
sudo dnf config-manager addrepo --from-repofile=https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
```
* __validate if present again__
  ```bash
  sudo dnf list --available | grep -E "^terraform"
  ```
```bash
sudo dnf -y install terraform
```
```bash
terraform version
```
```bash
terraform validate
```