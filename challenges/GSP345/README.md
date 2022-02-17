Challenge scenario
You are a junior cloud engineer intern for a new startup. For your first project, your new boss has tasked you with creating infrastructure in a quick and efficient manner and generating a mechanism to keep track of it for future reference and changes. You have been directed to use Terraform to complete the project.

For this project, you will use Terraform to create, deploy, and keep track of infrastructure on the startup's preferred provider, Google Cloud. You will also need to import some mismanaged instances into your configuration and fix them.

In this lab, you will use Terraform to create multiple VM instances, a VPC network with two subnetworks, and a firewall rule for the VPC to allow connections between the two instances. You will also create a Cloud Storage bucket to host your remote backend.

At the end of every section, plan and apply your changes to allow your work to be successfully verified.
Task 1. Create the configuration files
In Cloud Shell, create your Terraform configuration files and a directory structure that resembles the following:

main.tf
variables.tf
modules/
└── instances
    ├── instances.tf
    ├── outputs.tf
    └── variables.tf
└── storage
    ├── storage.tf
    ├── outputs.tf
    └── variables.tf

touch main.tf
touch variables.tf
mkdir modules
cd modules
mkdir instances
mkdir storage
cd instances
touch instances.tf
touch variables.tf
touch outputs.tf
cd ..
cd storage
touch instances.tf
touch variables.tf
touch outputs.tf
cd 

Fill out the variables.tf files in the root directory and within the modules. Add three variables to each file: region, zone, and project_id. For their default values, use us-central1, us-central1-a, and your Google Cloud Project ID.
You should use these variables anywhere applicable in your resource configurations.
Add the Terraform block and the Google Provider to the main.tf file.

Task 2. Import infrastructure
In the Google Cloud Console, on the Navigation menu, click Compute Engine > VM Instances. Two instances named tf-instance-1 and tf-instance-2 were already created for you.

Import the existing instances into the instances module.

To do this, you will need to:

Write the resource configuration to match the pre-existing instances.
Add the module reference into the main.tf file.
Use the terraform import command to import them into your instances module.
Hint Click on one of the instances to find its Instance ID, boot disk image, and machine type. These are all necessary for writing the configurations correctly and importing them into Terraform.
Click Check my progress to verify the objective.
Import infrastructure.

Task 3. Configure a remote backend
Create a Cloud Storage bucket resource inside the storage module. For the bucket name, use your Project ID.
You can optionally add output values inside of the outputs.tf file.
Add the module reference to the main.tf file.

Configure this storage bucket as the remote backend inside the main.tf file. Be sure to use the prefix terraform/state so it can be graded successfully.

If you've written the configuration correctly, upon apply, Terraform will ask whether you want to copy the existing state data to the new backend. Type yes at the prompt.

Click Check my progress to verify the objective.
Configure a remote backend.

Task 4. Modify and update infrastructure
Navigate to the instances module and modify the tf-instance-1 resource to use an n1-standard-2 machine type.

Modify the tf-instance-2 resource to use an n1-standard-2 machine type.

Add a third instance resource and name it tf-instance-3. For this third resource, use an n1-standard-2 machine type.

Optionally, you can add output values from these resources in the outputs.tf file within the module.
Click Check my progress to verify the objective.
Modify and update infrastructure.

Task 5. Taint and destroy resources
Taint the third instance tf-instance-3, and then plan and apply your changes to to recreate it.

Destroy the third instance tf-instance-3 by removing the resource from the configuration file.

Click Check my progress to verify the objective.
Taint and destroy resources.

Task 6. Use a module from the Registry
In the Terraform Registry, browse to the Network Module.

Add this module to your main.tf file. Use the following configurations:

Use version 2.5.0 (different versions might cause compatibility errors).

Name the VPC terraform-vpc, and use a global routing mode.

Specify 2 subnets in the us-central1 region, and name them subnet-01 and subnet-02.

You do not need any secondary ranges or routes associated with this VPC, so you can omit them from the configuration.

Navigate to the instances module and update the configuration resources to connect tf-instance-1 to subnet-01 and tf-instance-2 to subnet-02.
Hint You will need to update the network argument to terraform-vpc, and then add the subnetwork argument with the correct subnet.
Click Check my progress to verify the objective.
Use a module from the Registry.

Task 7. Configure a firewall
Create a firewall rule resource in the main.tf file, and name it tf-firewall. This firewall rule should permit the terraform-vpc network to allow ingress connections on all IP ranges (0.0.0.0/0) on TCP port 80.
Hint To retrieve the required network argument, you can inspect the state and find the ID or self_link of the google_compute_network resource you created.
Click Check my progress to verify the objective.
Configure a firewall.

Connectivity test (Optional)
After you have created a firewall rule to allow internal connections over the VPC, you can optionally run a network connectivity test.

Make sure both of your VMs are running.

Navigate to Network Intelligence > Connectivity Tests. Run a connectivity test on the two VMs to verify that they are reachable. You have now validated the connectivity between the instances!

Note Ensure that the Network Management API is successfully enabled; if it is not, click Enable.
Your configuration settings should resemble the following:

5e16c33f42d20029.png



main.tf
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}
provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}
module "instances" {

  source     = "./modules/instances"

}

variables.tf
variable "region" {
 default = "us-central1"
}

variable "zone" {
 default = "us-central1-a"
}

variable "project_id" {
 default = "<FILL IN PROJECT ID>"
}

/modules/instances/variables.tf
variable "project_id" {
  description = "The project ID to host the network in"
  default     = "qwiklabs-gcp-01-1ef729462fe2"
}
variable "region" {
  description = "The project ID to host the network in"
  default     = "us-central1"
}
variable "zone" {
  description = "The project ID to host the network in"
  default     = "us-central1-a"
}

modules/instances/instances.tf
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-1"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

terraform import module.instances.google_compute_instance.tf-instance-1 6019786458302724415
terraform import module.instances.google_compute_instance.tf-instance-2 7692116040395209023

terraform taint module.instances.google_compute_instance.tf-instance-3

terraform import module.storage.google_storage_bucket.storage-bucket qwiklabs-gcp-04-1f62bb37a0c0

gsutil mb gs://qwiklabs-gcp-04-1f62bb37a0c0


terraform import module.instances.google_compute_instance.tf-instance-1 6019786458302724415
terraform import module.instances.google_compute_instance.tf-instance-2 7692116040395209023
terraform import module.instances.google_compute_instance.tf-instance-3 2977482172733534668