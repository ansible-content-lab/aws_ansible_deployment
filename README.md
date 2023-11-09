# Ansible Collection - lab.aws_deployment

This is a collection that will deploy Ansible on AWS.  While this collection will work with any Ansible deployment on AWS, this is intended to be a starting point for customers that purchase Ansible Automation Platform subscriptions from the AWS marketplace.  Take this collection and enhance/improve/update based on the resources that you need for your AAP deployment.

## Introduction

This collection performs the following actions in the order listed.  There are variables that can be used to skip or prevent steps by setting variables, which are covered later in this document.

| Step                                 | Description                                                                                                                                                                                                                                                          |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Create a Deployment ID               | Creates a random string that will be used in tagging for correlating the resources used with a deployment of AAP.                                                                                                                                                    |
| Create VPC                           | Creates a VPC with a CIDR block that can contain the number of subnets that will be created.                                                                                                                                                                         |
| Create Subnets                       | Creates subnets for Controller, execution environments, Private Automation Hub, and EDA.                                                                                                                                                                             |
| Create IGW                           | Creates an internet gateway so that VMs have access to the internet.                                                                                                                                                                                                 |
| Create Security Group                | Creates a security group that allows AAP ports within the VPC, and HTTPS and automation mesh ports externally.                                                                                                                                                       |
| Create a Route Table                 | Creates a route table to route traffic properly.                                                                                                                                                                                                                     |
| Create Databases                     | Creates an Amazon RDS instance for Controller, Private Automation Hub, and EDA when there are one or more VMs configured to be deployed for each component.  By default, only Controller and Hub have VMs deployed, and therefor two database instances are created. |
| Create Controller VMs                | Creates the virtual machines for each AAP component.                                                                                                                                                                                                                 |
| Create Controller Load Balancer      | Creates an application load balancer when there are more than one Controller VMs created.                                                                                                                                                                            |
| Create Hub VMs                       | Creates VMs for Private Automation Hub.                                                                                                                                                                                                                              |
| Create EDA VMs                       | Creates VMs for Event Driven Ansible.                                                                                                                                                                                                                                |
| Register VMs with Red Hat            | Uses RHEL subscription manager to register each virtual machine for required RPM repos.                                                                                                                                                                              |
| Update VMs                           | Updates each VM deployed with latest kernel and packages.                                                                                                                                                                                                            |
| Setup One Controller VM as Installer | Moves the locally downloaded AAP installer to a single VMs and configures the installer inventory file based on the VMs that were created as part of this process.                                                                                                   |
| Configure SSH on installer VM        | Configures the installer VMs with a private SSH key so that it can communicate with the other VMs that are part of the installation process.                                                                                                                         |
| Run AAP Installer                    | Runs the AAP installer on the installer VM.  This process can take a long time.                                                                                                                                                                                      |
| Installation Cleanup                 | Deletes the installer and the SSH key from the the installer VM.                                                                                                                                                                                                     |

## Getting Started

These sections will describe required or recommended steps so that your Ansible Automation Platform deployment is as seamless as possible.

### This Collection

If you do not intend to make changes to the collection, then you can install directly from the `ansible-galaxy` CLI tool.  Examples in this readme will assume that you have done this.

```bash
ansible-galaxy collection install git+https://github.com/ansible-content-lab/aws_deployment.git
```

You may also download the collection from GitHub and modify to suit your needs.

### Red Hat Enterprise Linux

You will need to use a Red Hat Enterprise Linux (RHEL) Amazon Machine Image (AMI) as the foundation for your deployment.  While this collection will automatically find a public RHEL AMI available from AWS, public images bill for RHEL outside of your subscription for Ansible Automation Platform.

It is recommended that you create a custom AMI that you may then use to deploy RHEL with your subscriptions that come with Ansible Automation Platform.  [Red Hat Image Builder][image-builder] is a utility that makes creating a custom AMI easy.

### Ansible Automation Platform Installer

This collection expects that the most recent version of the [Ansible Automation Platform installer][aap-installer] is downloaded locally and can be moved to the deployment to install AAP.

This collection expects the standard "setup" file, not the "bundle".  And, it is not yet tested with the containerized installer.

### Roles

This collection includes the following roles.  Each role has default variables and required variables.  Review the default variables files to view all of the options that may be set.

| Role                                | Description                                                                         |
| ----------------------------------- | ----------------------------------------------------------------------------------- |
| `lab.aws_deployment.infrastructure` | Responsible for deploying the AWS infrastructure.                                   |
| `lab.aws_deployment.aap`            | Responsible for configuring and installing AAP once the infrastructure is deployed. |

### Variables

The following identifies the variables that you **must** set before running the deployment.  You may create a variable file that you can pass to the deployment playbook to set these variables.

| Variable                       | Description                                                                                                           |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| `infrastructure_region`        | The AWS region that the infrastructure will be deployed into.                                                         |
| `infrastructure_db_username`   | The PostgreSQL admin username that will be used for databases.                                                        |
| `infrastructure_db_password`   | The PostgreSQL admin password that will be used for databases.                                                        |
| `aap_installer_src_path`       | The path to the AAP installer file on the local machine where playbooks will be run.                                  |
| `aap_installer_path`           | The path on the installer VM where the installer file will be copied.                                                 |
| `aap_installer_inventory_path` | The path on the installer VM where the inventory file will be created.                                                |
| `aap_admin_password`           | The admin password to create for Ansible Automation Platform applications.                                            |
| `aap_installer_ssh_key`        | The name of the SSH key on the local machine that will be used to connect to the installer VM and other VMs deployed. |
| `aap_installer_ssh_key_src`    | The full path to the SSH key on the local machine.                                                                    |
| `aap_installer_ssh_key_dest`   | The path on the installer VM where the SSH key will be copied to during deployment.                                   |
| `aap_red_hat_username`         | Your Red Hat account username that will be used to register RHEL instances with subscription manager.                 |
| `aap_red_hat_password`         | Your Red Hat account password that will be used to register RHEL instances with subscription manager.                 |

The following is an example of a variables file with required and optional fields that can be used to tailor a deployment.  This example uses a custom AMI that was created with [Image Builder][image-builder], it provides a certificate and values to create a load balancer in front of Automation Controller, and it will run the AAP installer once the infrastructure is configured.

You may save this as any file, but later examples will use a file called `vars.yml` as a representation of this file.

```yaml
---
infrastructure_region: us-east-1
infrastructure_keypair_name: aws_test_key

infrastructure_controller_instances: 2
infrastructure_controller_ami: ami-0d56aee8074bef5d3

infrastructure_hub_instances: 1
infrastructure_hub_ami: ami-0d56aee8074bef5d3

infrastructure_eda_instances: 0
infrastructure_eda_ami: ami-0d56aee8074bef5d3

infrastructure_db_username: ansible
infrastructure_db_password: ansible_automation_platform_password

infrastructure_create_controller_lb: true
infrastructure_cert_path: /Users/scott/Downloads/cert.pem
infrastructure_cert_key_path: /Users/scott/Downloads/key.pem
infrastructure_cert_domain_name: controller.my.custom.domain

aap_installer_src_path: ~/Downloads/ansible-automation-platform-setup-2.4-2.tar.gz
aap_installer_unarchive_folder_name: ansible-automation-platform-setup-2.4-2
aap_installer_inventory_path: "/home/ec2-user/{{ aap_installer_unarchive_folder_name }}/inventory"

aap_installer_ssh_key: aws_test_key
aap_installer_ssh_key_src: "~/.ssh/{{ aap_installer_ssh_key }}"
aap_installer_ssh_key_dest: "/home/ec2-user/.ssh/{{ aap_installer_ssh_key }}"

aap_run_installer: true
aap_remove_installer_after_install: true
```

### AWS Credentials

The AWS collections used as dependencies require an AWS access key, secret key, and sometimes a token (if using session authentication).  These variables can be set in different places, such as the variables file above or through environment variables.  The easiest, and most portable, approach will be to set the following env vars.

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_SESSION_TOKEN`

The playbooks included in this collection will need a way to connect to the virtual machines that it creates.  By default, VMs are created with public IP addresses to make this simple.  But, the collection may be modified to use private IP addresses if your local machine can route traffic to private networks.

### Inventory File

The following example of an `inventory` file configures an SSH user and a local private key that will be used when configuring the VMs and installing AAP.

```ini
localhost

[all:vars]
ansible_ssh_user=ec2-user
ansible_ssh_private_key_file=~/.ssh/scott_aws_test_key
```

## Deploying Ansible Automation Platform

This section will walk through deploying the AWS infrastructure and Ansible Automation Platform.

### Checklist

- [ ] Install this collection (or download and modify)
- [ ] A RHEL AMI (if not using hourly RHEL instances)
- [ ] A locally downloaded copy of the [AAP installer][aap-installer]
- [ ] A variables file configured with required variables
- [ ] An inventory file with the proper SSH configuration
- [ ] Ansible CLI tools installed locally (`ansible`, `ansible-navigator`)
- [ ] Configure the AWS environment variables for authentication

### Running the Playbook

Assuming that all variables are configured properly and your AWS account has permissions to deploy the resources defined in this collection, then running the playbook should be a single task.

```bash
ansible-navigator run lab.aws_deployment.deploy_aap \
-i env/inventory \
--pae false \
--mode stdout \
--lf /dev/null \
--forks 3 \
--ee false \
--penv AWS_ACCESS_KEY_ID \
--penv AWS_SECRET_ACCESS_KEY \
--penv AWS_SESSION_TOKEN \
--extra-vars "aap_red_hat_username=$RED_HAT_ACCOUNT" \
--extra-vars "aap_red_hat_password=$RED_HAT_PASSWORD" \
--extra-vars "@env/vars.yml"
```

## Uninstall

The `playbooks/destroy_aap.yml` playbook will remove RHEL subscription entitlements and deprovision the infrastructure that has been associated with a deployment id.  This will permanently remove all data, so only run this playbook if you are sure that you want to delete all traces of the deployment.

```bash
ansible-navigator run lab.aws_deployment.destroy_aap \
-i env/inventory \
--pae false \
--mode stdout \
--lf /dev/null \
--forks 3 \
--ee false \
--extra-vars "aap_red_hat_username=$RED_HAT_ACCOUNT" \
--extra-vars "aap_red_hat_password=$RED_HAT_PASSWORD" \
--extra-vars "deployment_id=<deployment_id>"
```

## Authors

Written by Scott Harwell on the Ansible team at Red Hat.

[image-builder]: https://console.redhat.com/insights/image-builder
[aap-installer]: https://access.redhat.com/downloads/content/480/ver=2.4/rhel---9/2.4/x86_64/product-software
