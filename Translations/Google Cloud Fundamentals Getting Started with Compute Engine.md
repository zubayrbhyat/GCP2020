# Google Cloud Fundamentals: Getting Started with Compute Engine

## Overview
In this lab, you will create virtual machines (VMs) and connect to them. You will also create connections between the instances.

## Objectives
In this lab, you will learn how to perform the following tasks:
   * Create a Compute Engine virtual machine using the Google Cloud Platform (GCP) Console.
   * Create a Compute Engine virtual machine using the gcloud command-line interface.
   * Connect between the two instances.

## Task 1: Sign in to the Google Cloud Platform (GCP) Console

1. Make sure you are signed into Quiklabs using an incognito window
2. Not the lab's access time (for example 2:00:00 [2 hours]) and make sure you can finish in that time block. 
    >There is no pause feature. You can restart if needed, but you have to start at the beginning.
3. When ready, click **Start Lab**
4. Note your lab credentials. You will use them to sign in to Cloud Platform Console.
5. Click **Open Google Console**
6. Click **Use another account** and copy/paste credentials for **this** lab into the prompts.
    >If you use other credentials, you'll get errors or **incur charges**
7. Accept the terms and skip the recovery resource page.
    >Do not click **End Lab** unless you are finished with the lab or want to restart it. This clears your work and removes the project

## Task 2: Create a virtual machine using the Cloud Shell.

1. List the zones in the region you provided. Change the zone after **grep** as it applies to you:
    >gcloud compute zones list | grep us-central1
2. Set your default zone:
    >gcloud config set compute/zone us-central1-b
3. Create the first VM:
    >gcloud compute instances create "my-vm-1" \
    >--machine-type "n1-standard-1" \
    >--image-project "debian-cloud" \
    >--image "debian-9-stretch-v20190213" \
    >--subnet "default"
    >--tags http-server

## Task 3: Create a virtual machine using the gcloud command line
1. Create the second VM in Cloud Shell. You should still have it open after the last set of commands:
    >gcloud compute instances create "my-vm-2" \
    >--machine-type "n1-standard-1" \
    >--image-project "debian-cloud" \
    >--image "debian-9-stretch-v20190213" \
    >--subnet "default"

## Task 4: Connect between VM instances
1. Run the following command to connect my-vm-2. You can find your project name in the lab information box. Specify the zone you used earlier.
    >gcloud compute ssh --project PROJECT_ID --zone ZONE my-vm-2
2. Confirm the SSH key used and proceed with the connection to the VM. Type **yes** if prompted.
3. Enter the following command:
    >ping my-vm-1
    The ping command will reveal the fully qualified domain name of the computer you are pinging. In this case **my-vm-1** will show **my-vm-1.c.PROJECT_ID.internal**
4. Press Ctrl+C to stop the ping command.
5. Type the following command to connect to my-vm-1:
    >ssh my-vm-1
    Type **yes** if prompted.
6. Enter this command to install the Nginx web server:
    >sudo apt-get install nginx-light -y
    The -y flag here will bypass asking your confirmation to install the web server. 
    Sudo will invoke elevated permissions. 
    Apt-get install will search for the package you specify and install it if it finds it.
    *If you have any trouble with this command, remember to check your syntax (order of words used)
7. Run the following command to open up the home page in the Nano editor:
    >sudo nano /var/www/html/html/index.nginx-debian.html
For more information on using **nano** type **man nano** in a virtual machine or linux terminal
8. Search for the message "Hi from YOUR_NAME" by using the arrow keys on your keyboard. Enter your name:
    Original line:
    >Hi from YOUR_NAME
    Modified line:
    >Hi from <your name here>
9. Press **Ctrl+O** and Press **Enter** to save the edited home page file.
10. Type the following to confirm your web server is updated:
    >curl http://localhost/
    **Note that you're accessing the web server from within the VM. You can access the VM from my-vm-2 using the internal IP address, and from the internet using my-vm-1's external IP address. You may also use the fully qualified domain name of my-vm-1. Localhost is another way of access your machine via the loopback network adapter (or IP address 127.0.0.1)**
11. Type the following to exit SSH on my-vm-1:
    > exit
12. To confirm the above, run the following command on my-vm-2:
    >curl http://my-vm-1
13. Test connectivity from the internet to my-vm-1.:
    >gcloud compute addresses list
14. Take note of the external address from my-vm-1. Run the following command:
    >curl http://[my-vm-1-external-ip-address]

## Well done!
You have created VM Instances within GCP purely with command-line tools. Impressive!
