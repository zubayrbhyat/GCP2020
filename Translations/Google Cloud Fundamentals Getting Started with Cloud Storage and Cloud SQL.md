# Google Cloud Fundamentals: Getting Started with Compute Engine

## Overview
In this lab, you will create virtual machines (VMs) and connect to them. You will also create connections between the instances.

## Objectives
In this lab, you will learn how to perform the following tasks:
    * Create a Cloud Storage bucket and place an image into it.
    * Create a Cloud SQL instance and configure it.
    * Connect to the Cloud SQL instance from a web server.
    * Use the image in the Cloud Storage bucket on a web page.

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

## Task 2: Deploy a web server VM instance using Cloud Shell.

1. List the zones in the region you provided. Change the zone after **grep** as it applies to you:
    >gcloud compute zones list | grep us-central1
2. Set your default zone:
    >gcloud config set compute/zone us-central1-b
3. Create the first VM:
    >gcloud compute instances create "bloghost" \
    >--machine-type "n1-standard-1" \
    >--image-project "debian-cloud" \
    >--image "debian-9-stretch-v20190213" \
    >--subnet "default"
    >--tags http-server
    >--metadata=startup-script="apt-get update","apt-get install apache2 php php-mysql -y","service apache2 restart"
4. Run **gcloud compute addresses list** and copy the bloghost VM instance's internal and external IP addresses to a text editor for use later in this lab.

## Task 3: Create a Cloud Storage bucket using the gsutil command line

1. On the Google Cloud Platform menu, click Activate Cloud Shell. Click on Start Cloud Shell if prompted.
2. Enter your location into an environment variable
    >export LOCATION=US
    OR
    >export LOCATION=EU
    OR
    >export LOCATION=ASIA
3. Create a storage bucket using the DEVSHELL_PROJECT_ID environment variable as the chosen name.
    >gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
4. Make a copy of a banner image publically accessible from Google Cloud Storage.
    >gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
5. Copy the banner image to your newly created Cloud Storage bucket. **Note that you are specifying the name the file you are copying to your storage bucket**:
    >gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
6. Modify your ACL (Access Control List) of the object you just created so that it is readable by everyone:
    >gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

## Task 4 Create the Cloud SQL instance
1. Create the Cloud SQL instance using the following command:
    >gcloud sql instances create blog-db --root-password Complexity**1@ --database_version MYSQL_5_7
2. Find the IP addresses for your Cloud SQL instance with the following command:
    >gcloud sql instances describe blog-db
3. Create a user with a password of your choice. Where **Oranganotation(!** is typed out, you can place your own password:
    >gcloud sql users create blogdbuser --password="Orangenotation(!"
4. Authorise your network with the external IP address of your bloghost VM with the IP address followed by /32. Note that the /32 is specified to only authorise that IP address:
    >gcloud sql instances patch blog-db --authorized-networks=35.192.208.2/32

## Task 5
1. SSH into your Bloghost VM:
    >gcloud compute ssh --project DEVSHELL_PROJECT_ID bloghost
2. Navigate into the document root of the web server
    >cd /var/www/html
3. Open the root index.php file with nano using Sudo. Sudo uses elevated permissions to edit selected important files:
    >sudo nano index.php
4. Paste the following into the file:
    ><html>
    ><head><title>Welcome to my excellent blog</title></head>
    ><body>
    ><h1>Welcome to my excellent blog</h1>
    ><?php
    > $dbserver = "CLOUDSQLIP";
    >$dbuser = "blogdbuser";
    >$dbpassword = "DBPASSWORD";
    >// In a production blog, we would not store the MySQL
    >// password in the document root. Instead, we would store it in a
    >// configuration file elsewhere on the web server VM instance.
    >
    >$conn = new mysqli($dbserver, $dbuser, $dbpassword);
    >
    >if (mysqli_connect_error()) {
    >    echo ("Database connection failed: " . mysqli_connect_error());
    >} else {
    >    echo ("Database connection succeeded.");
    >}
    >?>
    ></body></html>
5. Edit the **$dbpassword** field with your password you specified earlier. Edit the password in quotes. Do not remove the quotation marks.
6. Press **CTRL+O** and then **Enter**. Press **CTRL+X** to exit nano.
7. Restart Apache:
    >sudo service apache2 restart
8. Use the curl command to test the external web server IP address. Replace the IP address below with yours:
    >curl 35.192.208.2/index.php
    You will receive the following error:
    >Database connection failed: ...
    This message occurs because you have not yet configured the PHP connection to your Cloud SQL instance.
9. Edit the index.php file with **nano**:
    >sudo nano index.php
10. Replace **CLOUDSQLIP** with the Cloud SQL Public IP address you noted above. Do not remove the quotation marks.
11. Press **CTRL+O** and then **Enter**
12. Press **CTRL+X** to edit nano
13. Restart Apache again:
    >sudo service apache2 restart
14. Use curl to access your web server again:
    >curl 35.192.208.2/index.php
15. You will receive a new message stating the database now connects:
    >Database connection succeeded.

## Task 6: Configure an application in a Compute Engine instance to use a Cloud Storage object

1. Go back to your SSH session on your **bloghost** VM.
2. Generate a public link for your image you uploaded. The format will look like this: https://storage.googleapis.com/PROJECT_ID/my-excellent-blog.png. You allowed public access to your Storage Bucket earlier, so this will automatically be created. Here is an example:
    >https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png
Note that you can find your project ID by typing **echo $DEVSHELL_PROJECT_ID** in Cloud Shell.
3. Navigate to your document root directly for Apache:
    >cd /var/www/html
4. Edit index.php with nano:
    >sudo nano index.php
5. Paste this HTML markup code below **<HTML>** and just before **<h1>**:
    ><img src='
6. Paste your public URL for the **my-excellent-blog.png** image file after the single quote:
    ><img src='https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png
7. Add a single quote and closed brackets:
    ><img src='https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png'>
8. Press **CTRL+O**, **Enter** and **CTRL+X**.
9. Restart Apache:
    >sudo service apache2 restart
10. Use curl to access your bloghost VM instance's public IP address:
    >curl 35.192.208.2/index.php
The site will now display an image, though you wil not see the image in a terminal. Use your web browser to view the image.






# Well done!
In this lab, you configured a Cloud SQL instance and connected an application in a Compute Engine instance to it. You also worked with a Cloud Storage bucket, all done via command-line. Well done!

# End your lab
You may end your lab once you are done by clicking End Lab. Qwiklabs will remove all resources and clean up the accounts and projects. Please remember to rate your experience in stars, from 1 star to 5, depending on how you felt after completing this lab.
Click submit when done.

* 1 star = Very dissatisfied
* 2 stars = Dissatisfied
* 3 stars = Neutral
* 4 stars = Satisfied
* 5 stars = Very satisfied

You can close the dialog box if you don't want to provide feedback.

For feedback, suggestions, or corrections, please use the **Support** tab.