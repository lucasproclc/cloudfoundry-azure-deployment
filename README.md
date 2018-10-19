# cloudfoundry-azure-deployment

WIP

https://docs.pivotal.io/pivotalcf/2-1/customizing/azure-deploy-terraform.html#download

Creating a Cloudfoundry deployment that is specific for Azure.
This will include CF Cli and Bosh CLI - 

Pre-requisite - 
Part 1

Must have an azure subscription
Must have access to Pivotal or CF account for Marketplace
Must have a cf login with api endpoint.
Must understand Terraform 
Do not place credentials in Terraform TFVARS <--- this is just an example
It would be good to understand CM - like Bosh or Chef or Ansible (even Tower)


Part 2
Create Azure Resource with Terraform
$ terraform init
$ terraform plan -out=plan
$ terraform apply plan


