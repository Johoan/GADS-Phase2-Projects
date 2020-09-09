# Lab: Automating the Deployment of Infrastructure Using Terraform

## Objective

In this lab, you learn how to perform the following tasks:

- Create a configuration for an auto mode network

- Create a configuration for a firewall rule

- Create a module for VM instances

- Create and deploy a configuration

- Verify the deployment of a configuration


## Steps

1. Set up Terraform and Cloud Shell

   Install Terraform: Terraform is now integrated into Cloud Shell. Verify which version is installed.

   1. In the cloud shell, To confirm that Terraform is installed, run the following command:

            terraform --version

            The output should look like: Terraform v0.12.24 

    2. To create a directory for your Terraform configuration, run the following command:

            mkdir tfinfra

    3. Make tfinfra your working directory by running this command:

            cd tfinfra

    4. Create a file with name provider.tf with this command:

            touch provider.tf

    5. Open the file, provider.tf

            nano provider.tf

    6. Copy the code into it

            provider "google" {}

            press ctrl + o, enter and ctrl + x to save and return to the shell command prompt 

    7. To initialize Terraform, run the following command:

            cd tfinfra
            terraform init

2. Create mynetwork and its resources

   ## Create the auto mode network mynetwork along with its firewall rule and two VM instances (mynet_us_vm and mynet_eu_vm).

    1. To create a new file, mynetwork.tf

            touch mynetwork.tf

    2. Open the file

            nano mynetwork.tf

    3. Copy the following base code into mynetwork.tf:

            # Create the mynetwork network
            resource [RESOURCE_TYPE] "mynetwork" {
            name = [RESOURCE_NAME]
            #RESOURCE properties go here
            }

    4. In mynetwork.tf, replace [RESOURCE_TYPE] with "google_compute_network" (with the quotes).

    5. In mynetwork.tf, replace [RESOURCE_NAME] with "mynetwork" (with the quotes).

    6. Add the following property to mynetwork.tf:

            auto_create_subnetworks = "true"

    7. Verify that mynetwork.tf looks like this:

            # Create the mynetwork network
            resource "google_compute_network" "mynetwork" {
            name                    = "mynetwork"
            auto_create_subnetworks = true
            }

    8. Press ctrl + o, Enter/return key to save and continue editing

    ## To configure the firewall rule: Define a firewall rule to allow HTTP, SSH, RDP, and ICMP traffic on mynetwork.

    1. Add the following base code to mynetwork.tf:

            # Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
            resource [RESOURCE_TYPE] "mynetwork-allow-http-ssh-rdp-icmp" {
            name = [RESOURCE_NAME]
            #RESOURCE properties go here
            }  

    2. In mynetwork.tf, replace [RESOURCE_TYPE] with "google_compute_firewall" (with the quotes).

    3. In mynetwork.tf, replace [RESOURCE_NAME] with "mynetwork-allow-http-ssh-rdp-icmp" (with the quotes).

    4. Add the following property to mynetwork.tf:

            network = google_compute_network.mynetwork.self_link

    5. Add the following properties to mynetwork.tf:

            allow {
                protocol = "tcp"
                ports    = ["22", "80", "3389"]
                }
            allow {
                protocol = "icmp"
                }

    6. Verify that your additions to mynetwork.tf look like this:

            # Add a firewall rule to allow HTTP, SSH, RDP, and ICMP traffic on mynetwork
            resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
            name = "mynetwork-allow-http-ssh-rdp-icmp"
            network = google_compute_network.mynetwork.self_link
            allow {
                protocol = "tcp"
                ports    = ["22", "80", "3389"]
                }
            allow {
                protocol = "icmp"
                }
            }

    7. Press ctrl + o, Enter/return and ctrl + x keys to save and return to cloud shell prompt

    ## Configure the VM instance

    1. Create a new folder, name: instance, in tfinfra folder

            mkdir instance

    2. Make the new folder your working directory

            cd instance

    3. Name the new file main.tf, and then open it.

            touch main.tf
            nano main.tf

    4. Copy the following base code into main.tf:

            resource [RESOURCE_TYPE] "vm_instance" {
            name = [RESOURCE_NAME]
            #RESOURCE properties go here
            }

    5. In main.tf, replace [RESOURCE_TYPE] with "google_compute_instance" (with the quotes).

    6. In main.tf, replace [RESOURCE_NAME] with "${var.instance_name}" (with the quotes).

    7. Add the following properties to main.tf:

            zone         = "${var.instance_zone}"
            machine_type = "${var.instance_type}"

    8. Add the following properties to main.tf:

            boot_disk {
              initialize_params {
                image = "debian-cloud/debian-9"
                }
            }

    9. Add the following properties to main.tf:

            network_interface {
              network = "${var.instance_network}"
              access_config {
                # Allocate a one-to-one NAT IP to the instance
              }
            }

    10. Define the 4 input variables at the top of main.tf, and verify that main.tf looks like this, including brackets {}:

            variable "instance_name" {}
            variable "instance_zone" {}
            variable "instance_type" {
              default = "n1-standard-1"
              }
            variable "instance_network" {}

            resource "google_compute_instance" "vm_instance" {
              name         = "${var.instance_name}"
              zone         = "${var.instance_zone}"
              machine_type = "${var.instance_type}"
              boot_disk {
                initialize_params {
                  image = "debian-cloud/debian-9"
                  }
              }
              network_interface {
                network = "${var.instance_network}"
                access_config {
                  # Allocate a one-to-one NAT IP to the instance
                }
              }
            }

    11. Press ctrl + o, Enter/return and ctrl + x, to  save and return to the cloud shell command prompt

    12. Run this command to go back to tfinfra folder

            cd .. 

    13. Run this command to open mynetwork.tf

            nano mynetwork.tf

    14. Add the following VM instances to mynetwork.tf:

            # Create the mynet-us-vm instance
            module "mynet-us-vm" {
              source           = "./instance"
              instance_name    = "mynet-us-vm"
              instance_zone    = "us-central1-a"
              instance_network = google_compute_network.mynetwork.self_link
            }

            # Create the mynet-eu-vm" instance
            module "mynet-eu-vm" {
              source           = "./instance"
              instance_name    = "mynet-eu-vm"
              instance_zone    = "europe-west1-d"
              instance_network = google_compute_network.mynetwork.self_link
            }

    15. Press ctrl + o, Enter/Return and ctrl + x

    ## Create mynetwork and its resources

    It's time to apply the mynetwork configuration.

    1. To rewrite the Terraform configuration files to a canonical format and style, run the following command:

            terraform fmt

            the output: mynetwork.tf

    2. To initialize Terraform, run the following command:

            terraform init

        The output should look like this:

            Initializing modules...
            - mynet-eu-vm in instance
            - mynet-us-vm in instance
            ...
            * provider.google: version = "~> 3.37"

            Terraform has been successfully initialized!

    3. To create an execution plan, run the following command:

            terraform plan

        The output should look like this:

            ...
            Plan: 4 to add, 0 to change, 0 to destroy.
            ...

        Terraform determined that the following 4 resources need to be added:

    ##  Name	                            Description
        mynetwork	                        VPC network
        mynetwork-allow-http-ssh-rdp-icmp	Firewall rule to allow HTTP, SSH, RDP and ICMP
        mynet-us-vm	                        VM instance in us-central1-a
        mynet-eu-vm	                        VM instance in europe-west1-d

    4. To apply the desired changes, run the following command:

            terraform apply

    5. To confirm the planned actions, type:

            yes

        The output should lokk like this: 

            ...
            Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

3. Verify your deployment

    1.  Run this command to confirm mynetwork

            gcloud compute networks list

    2.  Run this command to view the firewall configuration for mynetwork

            gcloud compute firewall-rules list --filter="name=('mynetwork-allow-http-ssh-rdp-icmp' ...)"

    ## Verify your VM instances in the Cloud Console

    1. View the mynet-us-vm and mynet-eu-vm instances. Note the internal IP address for mynet-eu-vm

            gcloud compute instances list

    2. For mynet-us-vm, click SSH to launch a terminal and connect.

    3. To test connectivity to mynet-eu-vm's internal IP address, run the following command in the SSH terminal (replacing mynet-eu-vm's internal IP address with the value noted earlier):

            ping -c 3 <Enter mynet-eu-vm's internal IP here>

    This should work because both VM instances are on the same network, and the firewall rule allows ICMP traffic!

    ## The End Of The Lab
    
        




     







     
