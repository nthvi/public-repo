# Terraform basic

This guideline shows an example of Terraform structure and how to provision the resource using Terraform .

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
In this sample, we will use Google  Cloud as Terraform provider, gcloud CLI is required. 
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
This sample creates PubSub topic and subscription on Google Cloud platform.

Firstly, provide a block to declare Google provider as PubSub resource from Google cloud platform. 

Copy this block to versions.tf file in this directory.

```bash
#This block declares google provider
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = ">= 4.0.0"
    }
  }
}
```

Secondly, provide blocks to declare resources of Pubsub topic and subscription. This sample allows the dynamic topic name by placing variables instead of a hard-coded value.

Copy this block to main.tf file in this directory.

```bash
#This block declares a local variable for the topic name which is called "local.prefix_name" in "my-topic" resource.
locals {
	prefix_name = "devday"
}

#This block declares google topic resource
resource "google_pubsub_topic" "my-topic" { 
	project = "klara-nonprod"
	name = "${local.prefix_name}-${var.topic_name}"
}

#This block declares google subscription resource consume the topic by defining the topic id using "google_pubsub_topic.my-topic.id" from "my-topic" resource
resource "google_pubsub_subscription" "my-subscription" { 
	project = "klara-nonprod"
	name  = "${local.prefix_name}-${var.topic_name}-subscription"
	topic = google_pubsub_topic.my-topic.id
}
```
Lastly, declare a variable to dynamic topic name which is called "var.topic_name" in "my-topic" resource.

This variable type is text and no default value is specified. Hence, this value should be inputted when the file is run.

Copy this block to variables.tf in this directory.

```bash
variable "topic_name" {
	type = string
}
```

This is an optional step, to expose the data about our resource in the end. This data is only available when the apply phase is executed, it means the real infrastructure has been provisioned.

To create the output data, declare an output block to the resource property want to be returned.

In this sample, due to the topic name is declaring dynamically, fetching the topic name after the resource is created is necessary.

Copy this block to output.tf in this directory. 
```bash
output "pubsub_topic_name" {
	description = "PubSub topic name created"
    value       = google_pubsub_topic.my-topic.name
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
To execute this phase, run the command

```bash
terraform plan
```
In this sample, we declared a variable "topic_name" without any default value. Hence, the prompt will be shown up to ask for the value. Enter the value to continue.

### Apply phase
After this phase, the resources of configuration files in this directory are provisioned to real infrastructure. In this phase, automatic run the plan phase at first and the prompt is shown up to get the confirmation before the changes are actually applied.
To execute this phase, run the command

```bash
terraform apply
```
In this sample, we declared a variable "topic_name" without any default value. Hence, the prompt will be shown up to ask for the value. Enter the value to continue.

Review the changes and confirm to complete the action.

### Destroy
After this execution, all resources of all configuration files in this directory will be deleted from the real infrastructure.
To execute this phase, run the command 

```bash
terraform destroy
```
Review the changes and confirm to complete the action.

### Output
After this execution, the console will show all the output data declared in this directory.

Or to show the output of specific data based on output block. As this sample, can replace the "<output-name>" to "pubsub_topic_name" to print out only the created topic name.

```bash
terraform output <output-name>
```
## References

[Terraform documentation for Google Cloud provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resource)

[Terraform best practice](https://cloud.google.com/docs/terraform/best-practices/general-style-structure)
