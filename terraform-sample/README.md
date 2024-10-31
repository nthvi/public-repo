# Terraform basic

This guideline shows a sample of Terraform module and how to provision the resource using Terraform module.

## Installation
### Terraform
Install Terraform to your workspace. Follow [documentation](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) to install terraform.
Then verify with this command

```bash
terraform -v
```

After finishing the installation, add the "<your-directory>/terraform" folder path to PATH on the system environment configuration in your local machine. 
After these steps terraform command is ready to run.

### Gcloud
In this example, we will use Google  Cloud as Terraform provider, install "gcloud" is required. 
Skip this step if gcloud has been installed in your workspace.
To install, follow [documentation](https://cloud.google.com/sdk/docs/install) or run this command in Power Shell in Window
```bash
(New-Object Net.WebClient).DownloadFile("https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe", "$env:Temp\GoogleCloudSDKInstaller.exe")

& $env:Temp\GoogleCloudSDKInstaller.exe

```
Run this command to authenticate on Google Cloud
```bash
gcloud auth application-default login
```

## Example project
This sample shows how to provision the group of resources from a specific source by using module block.

To provision all resources of PubSub which are declared in "basic-resource" folder. The "source" argument is required to specify the folder path of configuration files. The "topic_name" argument is required by "my-topic" resource as declared as a variable.

Copy this block to main.tf file in this directory.
```bash
#This block declares module block of resources declared in "basic-resource" folder.
module "pubsub-resource" {
	source = "./basic-resource"
    topic_name = var.topic_name
}
```

Next, provide a variable block for "topic_name" to allow user provide that value and forward the nested resource "my-topic" using "var.topic_name".

Copy this block to variables.tf file in this directory.

```bash
#This block declares variable for module "pubsub-resource"
variable "topic_name" {
	type = string
}
```
## Execution
In this stage, the state of the resources in configuration files will be reflected in the real infrastructure. 
The action is based on which phase will be executed. There are 3 main phases: init, plan and apply.

### Init phase
To initialize the backend and inistall plugin as provider, in this sample, to install Google provider plugin.

This execution only needs to run once until it detects the provider block is changed.

```bash
terraform init
```
### Plan phase
After this phase, the draft of the changes of all resources in this directory will be shown on the console, like the review phase.

In this phase, automatically validate all the configuration files and identify the gap between the current infrastructures and the desired infrastructures.
To execute this phase for all configuration files (.tf) placed in this directory, run this command

```bash
terraform plan
```
To execute plan command for configuration files (.tf) in specific folder, specify the target with a module name in the command.
For example, execute plan command for group of resource on "pubsub-resource" module only by running

```bash
terraform plan --target=module.pubsub-resource
```
In this sample, we declared a variable "topic_name" without any default value. Hence, the prompt will be shown up to ask for the value. Enter the value to continue.

### Apply phase
After this phase, the resources of configuration files in this directory are provisioned to real infrastructure. In this phase, automatic run the plan phase at first and the prompt is shown up to get the confirmation before the changes are actually applied.

To execute this phase for all configuration files (.tf) placed in this directory, run this command
```bash
terraform apply
```

To apply configuration files (.tf) in module block, specify the target with a module name in the command.

For example, provision group of resources on "pubsub-resource" module only by running
```bash
terraform apply --target=module.pubsub-resource
```

In this sample, we declared a variable "topic_name" without any default value. Hence, the prompt will be shown up to ask for the value. Enter the value to continue.

Review the changes and confirm to complete the action.

### Destroy
After this execution, all resources of all configuration files in this directory will be deleted from the real infrastructure.

To execute this phase for all configuration files (.tf) placed in this directory, run this command
```bash
terraform destroy
```

To destroy configuration files (.tf) in module block, specify the target with a module name in the command.

For example, delete group of resources on "pubsub-resource" module only by running
```bash
terraform destroy --target=module.pubsub-resource
```

Review the changes and confirm to complete the action.

## References

[Terraform documentation for Google Cloud provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resource)

[Terraform best practice](https://cloud.google.com/docs/terraform/best-practices/general-style-structure)

[Terraform module](https://developer.hashicorp.com/terraform/language/modules)
