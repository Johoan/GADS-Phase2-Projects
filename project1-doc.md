# LAB: App Dev: Setting up a Development Environment v1.1

## Objectives

In this lab, you will learn how to perform the following tasks:

   - Provision a Google Compute Engine instance.

   - Connect to the instance using SSH.

   - Install software on the instance.

   - Verify the software installation.

## Steps:

1. Creating a Compute Engine Virtual Machine Instance

    Name: dev-instance;  Region: us-central1; Zone: us-central1-a; Access Scopes: Allow full access to all Cloud APIs;  Firewall: Allow HTTP traffic; Others: default.

        gcloud beta compute instances create dev-instance --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --scopes=https://www.googleapis.com/auth/cloud-platform --tags=http-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=dev-instance

        gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server 

2. Install software on the VM instance
    
    1. In the SSH session, to update the Debian package list, execute the following command:

           sudo apt-get update

    2. To install Git, execute the following command:

           sudo apt-get install git

       If prompted, press Enter.    

    3. To download the Node.js setup script, execute the following command:

           curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

    4. To install npm and Node.js, execute the following command:
    
           sudo apt install nodejs   


3. Configure the VM to Run Application Software

    1. To check the version of Node.js, execute the following command:
        
           node -v

    2. To clone the class repository, execute the following command:

           git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    
    3. To change the working directory, execute the following command:

           cd ~/training-data-analyst/courses/developingapps/nodejs/devenv/

    4. To run a simple web server, execute the following command:

           sudo node server/app.js

    5. To install the Node.js library for Compute Engine, execute the following command:

           npm install

    6. To run a simple Node.js application that lists Compute Engine instances, execute the following command:

           node list-gce-instances.js

       Many details about your machine should appear in the terminal window.


## End of the lab

