# CFN-TEMPLATES

## Description:

This is a Infrastructure creation application that uses AWS Cloudformation to create infrasture on your AWS account like VPC, EC2, etc.

It gives you a simple and easy to use CLI to update/create Stacks on Cloudformation. Few stacks are already present here for usage. But you can also create your own stacks or update existing ones.

## Prerequisite
- AWS IAM user with Admin privilege/Cloudformation Admin access or Root user.
- Configure AWS cli on your local system. [Access keys, Region]
  - `aws configure` and fill details on prompt.


## Setup:

- `bundle`

- List all avaialable commands: `rake -T`
- List all stacks: `rake list_stacks`
- List all templates: `rake list_templates`

## Usage:

  To update staging VPC(already there in repo):
  - Update parameters in staging-vpc.json as per your requirement. You can make changes in vpc.yaml template file if needed.
  - `rake -T` to list all avialable commands
  - rake staging-vpc:update

  To create your own cfn stack:
  - If it is not multi-environment template:
    - Create a yaml file in templates folder (e.g. `akash.yaml`)
    - then rake will detect your stack as: `akash`
  - If you want to make it multi-environment:
    - Create a yaml file in templates folder (e.g. `akash.yaml`)
    - Create json file in templates-parameters folder like this (`staging-akash.json` & `production-akash.json`)
    - then rake will detect two stacks for you: (`production-akash` & `staging-akash`)

## Definitions and Guidelines:
- Template: Stores details of infrastructure you want to create. What resource to create/update (e.g. VPC, EC2, Subnet, RDS, etc)
- Parameters: Stores configuration that is passed into a template file. (e.g. IP address to use for a VPC etc.)
- It strictly accepts templates in only yaml format. And parameters in Json format.
- By default, it will start stack creation/updation on the AWS Region that you've configured using AWS CLI
- You don't have to specify your AWS Account, it will automatically detect AWS account after you've configured AWS CLI & will perform stack creation/updation only on that account.
- [Warning]: Make sure to check you've correct AWS account configured in your local machine otherwise it will start creating resources in other AWS account.
