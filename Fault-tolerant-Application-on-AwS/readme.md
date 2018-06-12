![Task - fault-tolerant m-az app](https://user-images.githubusercontent.com/17884787/39775610-724b42ba-52cc-11e8-9064-fc5a5ae956f8.png)

# Fault Tolerant wordpress across AZ.

This purpose of this repo is to work you through the process of deploying a fault tolerant application on AWS. My application of choice, is a wordpress. This repo assumes, you are new to AWS platform, hence the work through will commence from security (IAM), granting privileges to other staffs, to the repository, and down to the deployment. Most of the deployment would be through AWS CLI. 
What you will need: 
- An AWS account, of which you have all priviledges

login into your AWS console, with your email and password

Click on Services top left corner of your console: under security Click on IAM. 

![click on iam](https://user-images.githubusercontent.com/17884787/39481928-234eba0c-4d3b-11e8-8fc6-1d39f12f97ea.png)

In the IAM click on users on the left then click Add user

![add user](https://user-images.githubusercontent.com/17884787/39482013-72aec934-4d3b-11e8-8ff5-949ced1898b3.png)

Add the number of users you would like to give Administrative access. Then select the type of acces you would like to grant, I select both console and programatic access. You can select autogenerate password or custom password. I chose custom password, since I am simply creating for this repo. Then you can either select users requred to reset their password or force them to use the password you create.

![user details](https://user-images.githubusercontent.com/17884787/39482804-6fa4f1de-4d3e-11e8-9b4d-7f8033b87efc.png)

Click on next permission.

Then click create group, This makes it easier you to have an idea which categories, you have users, the policies they have an the tools they have to thier disposal, for example, you can have Test and Dev, limited to Ec2, S3, RDS, ELB etc. and so on, but they will be all be classified into a group. 

![policytype for the group](https://user-images.githubusercontent.com/17884787/39483209-ad872016-4d3f-11e8-82d3-41f7e9c71942.png)
Give the group a name, say, "AWSAdminAccess" and then select Administrative access permission for the group. Then, Select the new group you created.

![setting permission](https://user-images.githubusercontent.com/17884787/39483339-071849de-4d40-11e8-9352-5dbb0c57514b.png)

Click on create users

![create users review](https://user-images.githubusercontent.com/17884787/39483556-cc1f0f42-4d40-11e8-8c21-3170fdafa12e.png)

Now we have successfully created two users, with custom passsword access, and given them admin permission.
Download the csv file, or copy them manually or send them to the users via thier email as shown in the figure below.

![access secret keys](https://user-images.githubusercontent.com/17884787/39494971-5994b06e-4d66-11e8-8c6e-0c997739d941.png)

### Adding security to users access.

- First, Install , AWS Virtual MFA, google authenticator, or any other virtual authenticator, on your mobile device. This will be needed later
Lets use one of the new users and create MFA login which is more secured, a second layer security.

- Select user Mikepractice, under users, click on security credentials

![usersummary](https://user-images.githubusercontent.com/17884787/39495215-6b6d32ba-4d67-11e8-8673-6eac5f7162fd.png)

- Then click on the pencil like icon on Assign MFA device, then select a virtual MFA device. Click next step X 2ice. 

![assign mfa device](https://user-images.githubusercontent.com/17884787/39495549-b31fae16-4d68-11e8-95d3-ac4da040d523.png)

- Scan the QR code and enter seperately 2 successive authentication codes

![mfa authentication](https://user-images.githubusercontent.com/17884787/39495834-13fc9720-4d6a-11e8-9ad5-4fd4f8381c17.png)

- Click on Activate MFA, your user now has 2 level security enabled on the AWS account. click on IAM users, Mikepractice, and Note the number beside aws:iam::1234556677, this number is your account number for AWS. note it/ copy it to clipboard

- Signout and sign in as a different user, enter the account id, then the new user created and password, in the new window, enter the codes generated from your MFA device. 

![mfa codes](https://user-images.githubusercontent.com/17884787/39496464-872ea2f4-4d6c-11e8-98ee-e777311dd193.png) 

- You may be forced to change password, depending on the admin settings, you used when creating the user. I skipped that, allowing default, Then you are loggen in.

![logged in new user](https://user-images.githubusercontent.com/17884787/39496701-9da2ddec-4d6d-11e8-9c00-5ab76fb10706.png).

### Setting up the Terminal

Go to your terminal, 


``` $ curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip" ```
``` $ unzip awscli-bundle.zip ```
This is assuming python minimum version 2.7 is installed on your system

``` $ sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws ```

after installing awscli you can test it by entering the access ID and key you got from the IAM, to setup your terminal give the profile a name for example "osetutorial" as shown below:

``` $ aws configure ```
- [x] AWS Access Key ID [None]: A............
- [x] AWS Secret Access Key [None]: F............................

chose the region thats you want to use, for example

- [x] Default region name [None]: us-east-1
- [x] Default output format [None]: 

the terminal is now setup to run your stack

#### Simple AWS-Cli test

Test the cli by creating 2 buckets to you aws s3. To create buckets called mikemediatest2018 and mikecodestest2018 ENter the following: 

``` $ aws s3 mb s3://mkmedia-back-up-app2018 ``` 

and 

 ``` $ aws s3 mb s3://mkcodetestapp2018 ```

 Test to see if the buckets are created and you have access by copying a file into one of it.

 ``` $ echo "Hello Cloud Architects" > test.txt ``` 

 and 

 ``` $ aws s3 cp test.txt s3://mkcodetestapp2018 ```

 log in to your aws console, go to s3, and check the contents of the newly created bucket "mikescodetest2018", you would see the file test.txt in it. or simply enter 
``` $ aws s3 ls s3://mkcodetestapp2018 ```

 ![awsclis3bucket](https://user-images.githubusercontent.com/17884787/39590183-2ceb5aa4-4ece-11e8-8541-a296f4c1f8e2.png)

 Now we have confirmed the cli functions:

 you can empty the content of the s3 bucket using:

 ``` $ aws s3 rm s3://mikecodestest2018 --recursive ``` this completely empties the bucket.

### Building the Fault Tolerant Application using AWS-CLI

To run the cli script and automate the build from cli, the cloud formation script (Yaml or JSON), has to be written in a way, that everything you need for the infrastructure is available.

- First create ssh-key-pair

``` aws ec2 create-key-pair --key-name mk_keypair_app_2018 ```

Copy the output and save it into your regular key store for deployment, and:

``` $ chmod 400 ~/destination/to/mk_keypair_app_2018.pem ``` 
 

- The exact instance image to run, "Important to be cognizant of the image in relation to the AZ", my availability zone is US-east-1, hence the Amazon instance image should be from this AZ.
- Create an IAM role which grants Ec2 Admin Access to S3 buckets

- The VPC which would house the whole infrastructure.  
- The Security Groups
- The Availability zone, and the subnets available to this AZ.
- Create an SSH key from Ec2 console, this key would be used to get access to EC2 instance.

The file sample.yaml has the infrastructure information written in YAML.

- ##### Parameters: Described details for RDS DB instance as well as ELB
- ##### Resources: Described details for Security groups (AppNode: for Ec2), 
    - Ec2 Instance: This has a bas scripts written to automate the insstallation process for the Wordpress: 
         - It installs apache, php, and php-mysql, and a programm to stress the app.
         - changes directory the config "conf" creates a backup of the httpd.conf, removes it, and replace it with an edited conf file stored in one of my s3 buckets, which comes in handy in case you want to make use of cloud front, "Although we wont be using cloudfront for this application, the main focus is ensuring the application is fault-tolerant accross AZs".
         - changes directory to html, echo's This app is healthy to healthy.html. Downloads wordpress latest, untar the download, and copies recursively the wordpress files into /var/www/html
         removes the wordpress folder, the tar file, for space. Elevates the priviledge, change the ownership of apache and wp-content, restart apache, as make sure it remmains on.
         - The IAM role policy is needed to grant ec2 priviledges to upload and sync files to s3. This will be essential in making the information between all instances automatically syncronize. Although, the policy grants ec2 access to all resources, you could limit it, if you want to execute a strict policy.
    - DBInstance: I made reference to details mentioned in the parameters.

#### Running the Application at first
Now we know what is written in application stack file.
To run the application, make sure you have changed directory to this application directory, cloned from github, then enter the following command

``` $ aws cloudformation create-stack --stack-name mktest3 --template-body file://$PWD/sample.yaml ```

the output should be something like this:

 $ {
    "StackId": "arn:aws:cloudformation:us-east-1:12345667769:stack/mikestest/fff8a230-4ff0-11e8-8c1c-50cdkdkb4e4fd"
}

This would automate the build. you may between 15-30 mins.

- Next, we need to have details of the ec2 instance we just launched, its public dns, as well as its public IP address. To get this details, enter the following, if you are using us-east-1, else you may need to enter the specific region for this command, and you have only one instance, else you may need to also the instace ID:

``` $ aws ec2 describe-instances --filters "Name=instance-type, Values=t2.micro" "Name=availability-zone,Values=us-east-1c"``` 

Your output would give u the details of all ec2 t2 micro instances, located in your AZ in json format, where you can get the details of your deployed Ec2 instance. I am not posting the image for security purpose, copy the following out of the output for the specific ec2 WebServer instance, Called AppNode from the Cloudformation Stack, although some of this details are already specified from the stack file:

- Image id
- Public IP
- Pubic DNS
- KeyName
- Availability Zone
- Instance ID
- Security group ID

- Next elevate the permission of the ssh key from its stored loacation. for mine I used the following:    


When the build is complete, paste the copied Public DNS into your browser, and enter it, to test and confirm the installation was properlly done.

you should see something like this:

![wordpress installed](https://user-images.githubusercontent.com/17884787/39657304-a1fabde2-4fd4-11e8-8b2f-bd804e6ce780.png)

Next enter the following, to copy the end point, of the rds-mysql database, used.

``` $ aws rds describe-db-instances ``` 

This command gives out the details we need, showing that the database is available across zones, copy the address of the end point of the instance. it should be something like 
``` td11mfly237yksmaref3.ck8yq7makec8qe.us-east-1.rds.amazonaws.com ```.


Now enter all the details for your MysQL Database instance on the page. On the Database details: paste the endpoint copied from the clip board. 
On the next page opened follow the instruction and:
IMP:: Do not click Run the installation yet, copy the information highlighted to the clipboard.

To follow the instruction we need to ssh to the ec2 instance.

on the terminal, lets ssh into the Ec2 instance.

``` $ ssh -i "~/Path-to-key/mk_keypair_app.pem" ec2-user@ec2-52-70-213-113.compute-1.amazonaws.com ```

![ec2 instance](https://user-images.githubusercontent.com/17884787/39657265-4d25c8de-4fd4-11e8-99e6-ec7ce4d27877.png)

Elevate your priviledge, ``` $ sudo su ```

``` $ nano /var/www/html/wp-config.php ```

Paste the details of the information copied from the clipboard in here. Ctrl + X + yes, save and exit. Now you can run the wordpress installation from your browser. 

Fill in the details, that you wish for your configuration...

![the installation](https://user-images.githubusercontent.com/17884787/39657922-d2ddca66-4fda-11e8-8ef2-f44a9102f126.png)

Run the complete installation, enter your login details, you should have something like this as your dashboard.

![wordpress dashboard](https://user-images.githubusercontent.com/17884787/39658409-f68db41a-4fe1-11e8-972f-2670d0c3783c.png)

Play with the word press, add pictures to the media, posting it, do as you wish.

### Making the wordpress application fault-tolerant

Now, we have installed the application, but it is prone to breakdown, if we accidentally delete it or, if the application went down, how do we make sure it is fault-tolerant, or and a disaster recovery, first within different AZ. Note: You can use the same, idea for making it fault tolerant accross regions, only you may make use of a few more tweaks, To stay on topic, we would now make use of the s3 buckets earlier created.


- First: SSH to the ec2 instance using the command above, 
then elevate your privilege to root ``` $ sudo su ``` then ```cd /var/www/html ```, then ls.

you should see something like this: 

![wordpress content](https://user-images.githubusercontent.com/17884787/39658210-09b90452-4fdf-11e8-89cd-4153ec65f93e.png)

- Create Backup: First configure the Ec2 instance for Cli access to the platform, entering your access, key, region, as well as secret key. Next, copy all files/codes into one of the s3 buckets
First check to see all s3 buckets you have, to confirm the availability again enter: 

``` # aws s3 ls ```

you should see a list of all your s3 buckets installed, including the 2 new ones created, if not you may have to configure your ec2 instance for cli. Next, backup the contents into the codetestbucket created earlier

``` # aws s3 cp --recursive /var/www/html s3://mkcodetestapp2018 ```

Use recursive to make sure it copies all the files, directories and the sub-directories.

you can confirm this action by running 

``` # aws s3 ls s3://mkcodetestapp2018 ``` 

you would have something like this:

![backedup files](https://user-images.githubusercontent.com/17884787/39658208-02456152-4fdf-11e8-909e-40aec298b991.png)

So all the codes are backed up in a bucket, which, the content can be simply copied to a new Ec2 instance in cases of failure.

Now lets check the wordpress and see its content, you may be wordering why did we create 2 s3 buckets, instead of one. If you have not uploaded any images nor any post it should look like this

``` # cd wp-content && ls ```  

![wordpress content](https://user-images.githubusercontent.com/17884787/39658434-5f925bc8-4fe2-11e8-857b-9665bf34a492.png)

Now Upload a couple of images to the wordpress dashboard media, and possibly post it. 

![uploaded files](https://user-images.githubusercontent.com/17884787/39658534-a6b5eb90-4fe3-11e8-913a-673f543f21b7.png)

Make a simple post!!

![first worpress post](https://user-images.githubusercontent.com/17884787/39658533-a489a960-4fe3-11e8-94a0-ee60cc4c7d87.png)

Now if you enter ls again from the terminal a folder uploads should have been created which has contents of all the images and files uploaded.

![uploaded mediafiles](https://user-images.githubusercontent.com/17884787/39658565-32caa4cc-4fe4-11e8-8173-fbf2fe1c8fbe.png)

click on the posted link on your wordpress app, also termed "permalink", then right-click on the image file and copy the image link address. paste in on your browser. or 

``` http://ec2-52-70-213-113.compute-1.amazonaws.com/index.php/2018/05/07/hello-world/ ``` 
 
There you have your new post.

To ensure the application is fault tolerant, We need to automate the sync process of copying newly added files into the bucket, so that the same content is shared across all the instances.

- Change directory into the etc. ``` # cd /etc ```

- ``` # nano crontab ```

This is linux version of scheduling task, similar to task scheduling in Jenkins.

You should have something like this: 

![crontab](https://user-images.githubusercontent.com/17884787/39679304-2b6fd09a-5169-11e8-9381-859ee9a9a8c8.png)

Enter the following

``` */2 * * * * root aws s3 sync --delete /var/www/html/wp-content/uploads s3://mkmedia-back-up-app2018 ```

and 

``` */2 * * * * root aws s3 sync --delete /var/www/html s3://mkcodetestapp2018 ```

This makes sure contents of the html codes, as well as media added are the same, with that of the bucket, including any deleted file. To enforce/startup this cron task, enter the following:

``` # service crond restart ``` 

This would force start the cron scheduled Job. If you add a new file to your media content, check the bucket content in 4 mins, it must have been copied into it. If its not copied, you may need to configure aws cli on your ec2 instance as well as ensure that the instance, has IAM S3-Admin access role, attached to it.

If you delete one file and add 2 new files, after waiting for a while you would see all this changes reflected in the bucket;

lets check the content of the media bucket to confirm it works.

if you check the present content it would be empty, but wait for 4 minutes. while waiting lets add a file the html content, 

``` # echo "Hello Cloud Architects, keep being awesome" > test.txt ```
``` # echo "Hello Solution Architects" > test2.txt ```
``` # echo "Hello DevOPS Engineer, keep being awesome" > text2.txt ```

 
After a while check both buckets: 

``` # aws s3 ls s3://mkcodetestapp2018 ``` and 
``` # aws s3 ls s3://mkmedia-back-up-app2018/2018/05/ ```

There you have it. All added files and changes are in the code as well as the media bucket in s3. The auto sync process is ON!!

Now we have everything on the instance, we need to create an ami image of the instance to place behind the ELB, and auto scaling group.

#### Creating an Ami Images of the application

Name the first Image, WP-AMI Server, and the Other WP-AMI Production, create the images running something like this: for each, changing the names.

``` aws ec2 create-image --instance-id i-0818ba5bd174dbccb --name "My WP-AMI Server" --description "AMI Image to lauch Auto-config of WP App ``` 

Your output should be something like this:

![ami created image](https://user-images.githubusercontent.com/17884787/39680606-99ba0adc-5170-11e8-8de8-11aa4dbc67c4.png)

Copy both image IDs, we would be needing them in the next section.

You can now terminate the Ec2 Instance since we have its image already.
 
The output would show the instance, and some of its details, that its been shutdown in json format

#### Launching Auto config, Autoscaling And Elastic Load Balancer

We could either attach this new details to a new cloud formation stack, and launch it or we make use of AWS CLI, since this is the main focus of repository.

First we create a target group,

``` aws elbv2 create-target-group --name mk-Webapp-targets --protocol TCP --port 80 --vpc-id vpc-07ae5b60 ``` referencing the WP-Prod instance, from the console.

Pointing to the instance, next we create a an Elastic load balancer whic points to this target group. 

``` # aws elb create-load-balancer --load-balancer-name mk-elb-Classic --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80" --availability-zones us-east-1c us-east-1b us-east-1f ```

I ran this creation on the console, but did this can be done on the cli as well. 

- Next, we need to put our Instances behind the Classic load balancer, and change the settings of the word press to point to the public DNS of the Load balancer. Else, if we create multiple instances, scale and have not made this changes, the site would not perform well. We do this change on the console of the word press settings. Such that when the instance is scaled up or down, customers would be directed to the load balancer, which points to its public DNS, which the applications is pointing to.

So now, we have one instance, which is behind and ELB "mk-elb-Classic" which makes use of the target group mkWebAPPTG, pointing to the instance.
And Currently, our application is pointing directly to the ELB.

- Next we create a new instance in a different AZ, using the image we created earlier, to do this we enter the following

``` aws ec2 run-instances --image-id ami-3b2aa644 --count 1 --instance-type t2.micro --key-name mk_keypair_app --subnet-id subnet-e23352ba --iam-instance-profile Name=task --tag-specifications 'ResourceType=instance,Tags=[{Key=MK-webserver,Value=production}]'```

Next, we create a lauch configuration, which makes use of the created image, next we create auto scaling group then we put the auto scaling group behind an Elastic load balancer.

We need to change the cron Job, such that, the new changes are not reflected in the s3, buckets, instead they are downloaded. 

To make this changes, we need to ssh into this instance, and chaneg the cron job settings, 

we need to get the public dns, to do this we enter 

``` aws ec2 describe-instances --filters "Name=tag:MK-webserver,Values=production" ```

ssh into the instance from another terminal, using 
``` $  ssh -i "~/Path-to-ssh-key/mk_keypair_app.pem" ec2-user@ec2-54-152-20-195.compute-1.amazonaws.com ```

This time, we run the opposite of the cron job, so It syncs , the content of the s3 buckets into the instance, and not the other way around, such, that, when one makes changes from the web server content codes, it is synced into s3 buckets, and from the s3 buckets they are synced into this instance.

``` $ sudo su ```, nano /etc/crontab

replace the two lines with the following

``` */2 * * * * root aws s3 sync s3://mkmedia-back-up-app2018 /var/www/html/wp-content/uploads --delete``` 

and 

``` */2 * * * * root aws s3 sync s3://mkcodetestapp2018 /var/www/html --delete ```


Now we can test this to see if it works, yap it works!!!

Now we need to create an image of this ami, which we would make use of for our auto scaling group. to do this we enter: 

``` aws ec2 create-image --instance-id i-0d1044ef6737c4893 --name "My WP-AMI Production" --description "AMI Image to lauch Auto-config of WP App" ```

The Ami created is "ami-61ca451e".

WE now make use of this AMI for the launch configuration as well as Auto scaling group

``` # aws autoscaling create-launch-configuration --launch-configuration-name mk-asg-launch-config --key-name mk_keypair_app --image-id ami-61ca451e --security-groups sg-a3689eeb --instance-type t2.micro --instance-monitoring Enabled=true --no-ebs-optimized --no-associate-public-ip-address --placement-tenancy dedicated --iam-instance-profile S3-Admin ```

Now we can delete both active instances, since we have our code content stored on s3 buckets. 

Next we create the auto scaling group, prefereably from the console, we need to specify the AZ, active for our VPC, This is vital, else, it will not launch.


Next we place the ASG and WP-Prod behind ELB.

``` aws elbv2 create-target-group --name mk-Webapp-targets --protocol TCP --port 80 --vpc-id vpc-07ae5b60 ``` referencing the WP-Prod instance, from the console.


We may have to go to the console to add the health-check-path to file to the instance, or we create a target group from scratch on the console, where we indicate the path to file for the instance.


- Next we create an auto-scaling group:

Here we may have to make use of the management console, on the Ec2 Platform, If your AWS platform, and roles permission is not complex, you can create it using cli, as shown below, then later add AZ's, on the console. But for ease of deployment, Its much easier with the console.

Click AutoScaling group, and click on Create Auto-scaling group, You can create it, from either 

``` # aws autoscaling create-auto-scaling-group --auto-scaling-group-name asg-auto-scaling-group --launch-configuration-name asg-launch-config --min-size 2 --max-size 5 --desired-capacity 2 --default-cooldown 600 --placement-group my-placement-group --termination-policies "OldestInstance" --availability-zones us-east-1a us-east-1d us-east-1e us-east-1b --load-balancer-names myALB-load-balancer --health-check-type ELB --health-check-grace-period 12 ```

![creating auto scaling group](https://user-images.githubusercontent.com/17884787/39739038-c86e017a-525c-11e8-9fed-3d406690fe8f.png)

- We need to lauch the AMI of the WebServer and also place it behind our ELB. So that its content is reflected across. Auto scaling group for the WP Changes. And place it behind the Load balancer, As well. 

 ```aws autoscaling create-launch-configuration --launch-configuration-name mk-WPContent-asg-LC --key-name mk_keypair_app --image-id ami-3b2aa644 --security-groups sg-a3689eeb --instance-type t2.micro --instance-monitoring Enabled=true --no-ebs-optimized --no-associate-public-ip-address --placement-tenancy dedicated --iam-instance-profile S3-admin ```
 
 we place this behind an Auto scaling group as well, which we place behind the ELB. 




Please Note: Some times The deployment environments restricts the auto scaling group. 

![elastic load balancer](https://user-images.githubusercontent.com/17884787/39739115-37233a72-525d-11e8-95cd-f4079d0432ed.png)

But Basically, on placing the 2 auto scaling groups behind the ELB, of 2 or more instances behind, an elastic load balancer, When you stress test the application, it would scale up, according to the Auto-scaling group, and When you delete instances, it would scale down as well.

The Old media contents (Pictures) are probably showing well, on the application, due to some Cache issues, because the contents are still in the s3 buckets. But when you make new changes from the content, it is reflected accross to the production via the S3.

When we run any of this instances, they would all work.


- The next is to test stress the application and force delete the application and monitor its performance. We would make use of the application downloaded earlier called stress.

ssh into both the Mk-ASG production as well as the Mk-ASG Content instances, which are behind the ELB, and run the following

``` # stress --cpu 150 ```, After 5 mins this would automate a scaling of new instances, from the auto scaling group, for both the production as well as the Content. 

we can next delete, most of the instances, leaving one behind the elb, 
The web site would still be reachable.

Lastly lets restart, the database, we would see the website can still be reach on reboot, after a failover to another AZ.

Its not perfect, but it works, its a fault tolerant application across multi-AZ, you can use any of the public DNS of any of the instances and the site would still be available.

``` http://ec2-54-208-18-18.compute-1.amazonaws.com/ ```



![assgnmt fault-tolerant m-az app](https://user-images.githubusercontent.com/17884787/39775610-724b42ba-52cc-11e8-9064-fc5a5ae956f8.png)
