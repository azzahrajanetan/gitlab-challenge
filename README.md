# gitlab-challenege

This challenge is to automate the [installation of gitlab on microsoft auzre](https://docs.gitlab.com/ee/install/azure/#deploy-and-configure-gitlab) (acloudguru azure sandbox) using Terraform and Ansible.

## Method

Prior to this, previous learners have completed the challenge. For the ease of completing this challenge, we will `git clone` from (https://github.com/vysky/challenge-gitlab) and run the Terraform and Ansible Playbook from here.

## Getting started

### Pre-requisites

Ensure you have the following prepared and installed on your machine

1. python
2. [az cli](https://docs.microsoft.com/cli/azure/install-azure-cli)
2. [terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
3. [ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html)
4. username and password provided by acloudguru azure sandbox (or your own azure account)

### Running Terraform and Ansible

#### Git clone the repo

In your terminal:
1. `git clone https://github.com/vysky/challenge-gitlab`
2. `cd challenge-gitlab/`

#### Editing the Terraform `main.tf` file

From your browser:
1. Login to Azure portal using the username and password generated by acloudguru's Azure Sandbox
2. Copy the generated resource group name in the Azure portal.
3. **Open `main.tf` file and replace the `default` value of `variable "rg"` with the copied resource group name.
4. Save the `main.tf` file

** Note: You must change the resource group to your sandbox resource before moving on to Running Terraform.

#### Running Terraform

In your terminal (you should be in the `challenge-gitlab` folder):
1.  `az login -u <acloudguru username>`. Then copy and paste the password from acloudguru.
2. `terraform init`
3. `terraform plan`
4. `terraform apply --auto-approve`
5. When completed, refresh your Azure porta. Resources should have successfully deployed and three files (`inventory`, `fqdn`, `password`) will be generated in the working directory.

#### Running Ansible playbook `main.yml`

Ensure you have `sshpass` installed in your terminal.

- For Mac users, run `ansible-playbook main.yml`.

- For Windows users, you can run the playbook in a few ways to `ssh` into the public ip.

##### Method 1
Run `ansible-playbook --ask-pass main.yml` and key in the password from the `password` file (which should be Qwerty123456!).

##### Method 2
1. Edit the `inventory` file. 
2. Add in `ansible_ssh_password=Qwerty123456!` after the IP address.
3. Run `ansible-playbook main.yml`

If `ansible-playbook` is successful, a Gitlab 'root' and 'password' will be generated as one of the debug message from the playbook.

#### Checking if it is successful
1. Copy the `fqdn` (url) generated in the `fqdn` file and paste it in a new browser. You should be able to see the Gitlab login page.
2. Log in with the username `root`, copy and paste the password from the Ansible playbook debug message.

Congratulations! You have successfullu completed the challenge.

#### Terraform destroy and cleaning up
1. In terminal, type `terraform destroy --auto-approve` to destroy the resources created and files generated on local machine

## Further reading from Brandon (https://github.com/vysky/challenge-gitlab)

### finding the publisher, offer and sku of image

to install gitlab onto a vm, we are required to use the image provided by gitlab

in azure portal, we only need to do a search to find the gitlab image and use it to create the vm. however we are required to provide `publisher`, `offer` and `sku` of the image to automate and create the vm in terraform

there are multiple methods to find the required parameters of the gitlab image (or other images by other publishers)

#### 1. arm template and az powershell

this [fantastic article](https://vincentlauzon.com/2018/01/10/finding-a-vm-image-reference-publisher-sku/) guide you on how to find `publisher`, `offer` and `sku` through arm (azure resource manager) template and az powershell

#### 2. az cli

this [az vm image document](https://docs.microsoft.com/cli/azure/vm/image) by microsoft lists the commands to find `publisher`, `offer`, `sku` and more

example instructions for az cli to find the sku of gitlab by first finding the `publisher` , then `offer`, and finally `sku`
1. ensure you are login to az in terminal using `az login -u <username>` and enter the password when prompted
2. type `az vm image list-publishers --location eastus > publisher.txt` in terminal to output a txt file with a list publishers in eastus
3. open the publisher txt file and search for `gitlab` (there may be multiple results) and note down the `name` value (as of 17 Mar 2022, `gitlabinc1586447921813` is the correct publisher for the image we need to use)
4. type `az vm image list-offers --location eastus --publisher gitlabinc1586447921813 > offer.txt` in terminal to output a txt file with a list of offers by `gitlabinc1586447921813`
5. open the offer txt file and note down the `name` value (it should be `gitlabee`)
6. type `az vm image list-skus --location eastus --offer gitlabee --publisher gitlabinc1586447921813 > sku.txt` in terminal to output a txt file with a list of skus
7. open the sku txt file and note down the `name` value (it should be `default`)
8. add the `publisher`, `offer` and `sku` we found above to the terraform file to automate the creation of vm using the gitlab image

## work in progress

working in integrating both terraform script and ansible script so the automation is seemless (code is already present in the `main.tf` file)
