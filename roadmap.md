# Terraform lab roadmap

## :point_right: Install Terraform on Fedora 43 (WSL) :point_left:

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

## Start here :point_down:

```bash
mkdir -p workstation/lab1 logs
```
```bash
cd workstation/lab1
```
```bash
touch main.tf providers.tf variables.tf
```
:point_right: [Populate the above files from here](terraform/Hands-On%20Labs/Section%2002%20-%20Understand%20IAC%20Concepts/02%20-%20Benefits_of_Infrastructure_as_Code.md#task-7-prepare-files-and-credentials-for-using-terraform-to-deploy-cloud-resources)

```bash
export AWS_ACCESS_KEY_ID=<?>
export AWS_SECRET_ACCESS_KEY=<?>
export AWS_DEFAULT_REGION=ca-central-1
export TF_LOG=DEBUG
export TF_LOG_PATH="$(git rev-parse --show-toplevel)/logs/run_$(date +%Y%m%d_%H%M%S).log"
```

```bash
tf init
tf validate
tf plan
```

```bash
tf apply
```

```bash
tf state list
```

```bash
tf plan -destroy
```

```bash
tf destroy
```

```bash
tf plan -out my_plan
tf show my_plan
tf apply my_plan
```

:point_right: [Variables precedence](terraform/Hands-On%20Labs/Section%2004%20-%20Understand%20Terraform%20Basics/06%20-%20Intro_to_the_Input_Variables_Block.md)

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```