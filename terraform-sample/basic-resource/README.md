# Terraform basic

This guideline shows an example of Terraform structure and how to provision the resource using Terraform.

## Installation
### Terraform
Install Terraform to your workspace. Follow [documentation](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) to install terraform.
Then verify with this command

```bash
terraform -v
```
### Gcloud
In this sample, we will use Google  Cloud as Terraform provider, gcloud CLI is required. 
Skip this step if gcloud has been installed in your workspace.
To install, follow [documentation](https://cloud.google.com/sdk/docs/install) or run this command in Power Shell in Window
```bash
(New-Object Net.WebClient).DownloadFile("https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe", "$env:Temp\GoogleCloudSDKInstaller.exe")

& $env:Temp\GoogleCloudSDKInstaller.exe

```

## Example project
This sample creates PubSub topic and subscription on Google Cloud platform.

Firstly, provide a block to declare Google provider as PubSub resource from Google cloud platform. 

Copy this block to provider.tf file in this directory.

```bash
#This block declares provider, google provider is set as default if no provided any provider.
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

#This block declares topic resource
resource "google_pubsub_topic" "my-topic" { 
	project = "klara-nonprod"
	name = "${local.prefix_name}-${var.topic_name}"
	message_retention_duration = "3600s" 
}

#This block declares subscription resource consume  the topic by referencing the topic id using "google_pubsub_topic.my-topic.id" from "my-topic" resource
resource "google_pubsub_subscription" "my-subscription" { 
	project = "klara-nonprod"
	name  = "${local.prefix_name}-${var.topic_name}-subscription"
	topic = google_pubsub_topic.my-topic.id

	# 30 minutes
	message_retention_duration = "1800s"
	retain_acked_messages      = true

	ack_deadline_seconds = 20

	expiration_policy {
		ttl = "300000.5s"
	}
	retry_policy {
		minimum_backoff = "10s"
	}

	enable_message_ordering    = false 
}
```
Lastly, declare a variable to dynamic topic name which is called "var.topic_name" in "my-topic" resource.

This variable type is text and no default value is specified. Hence, this value should be inputted when the file is run.

Copy this block to variable.tf in this directory.

```bash
variable "topic_name" {
	type = string
}
```
## Execution
To provision the resources in configuration files, follow the steps as Terraform phases
### Init phase
To initialize the backend, in this sample, to install Google provider plugin and module block.
```bash
terraform init
```

### Plan phase
To validate the change and identify the gap between the current infrastructures and the desired infrastructures.
This phase will return the draft of the new state, like the review phase.
```bash
terraform plan
```
In this sample, we declared a variable "topic_name" without any default value. Hence, the prompt will be shown up to ask for the value. Enter the value to continue.

### Apply phase
To create or update the resource to real infrastructure. This phase executed the plan phase as default at first and the prompt is shown up to get the confirmation before the changes are applied to real infrastructures.
```bash
terraform apply
```
In this sample, we declared a variable "topic_name" without any default value. Hence, the prompt will be shown up to ask for the value. Enter the value to continue.

Review the changes and confirm to complete the action.

### Destroy
Delete the resource from real infrastructure.
By removing or comment the desired block of resources in the configuration file.
Run the command 
```bash
terraform destroy
```
Review the changes and confirm to complete the action.
## References

[Terraform documentation for Google Cloud provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resource)

[Terraform best practice](https://cloud.google.com/docs/terraform/best-practices/general-style-structure)