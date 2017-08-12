# Load Balancing and High Availability.

## Key Concepts

Before we get started, here's a couple of loose concepts:

 - Load Balancers forward your traffic to one or more service providers. What this essentially means is that you can have multiple EC2 instances serving the same kind of content, being represented as one service.

 - High Availability means making it so that your system remains operational for a desirably long length of time. This means making it more durable to system failures - having failovers, in this case.

 - Service Separation refers to making sure that if you have multiple systems dependent on each other, that they exist separately and share fewer failure modes.

 - Artifacts are tangible byproducts of our systems development. In this case, our artifact will be the modified Wordpress package that's special to only you.

Now that that's out of the way, let's get started!

## What we're going to do

In this practical session, we will:

 - Create an Elastic Load Balancer
 - Reinstall Wordpress, but using an external database
 - Create an Artifact of your modified Wordpress, then upload it to S3
 - Create another instance, using the artifact as a base


## Creating an Elastic Load Balancer

### 1.) Create your load balancer

Go to Services > EC2, then look for "Load Balancers". Click on "Create Load Balancer"

![Image][2-1-1-create]


### 2.) Select the type of load balancer

In the next page, make sure you select *Classic Load Balancer*.

![Image][2-1-2-classic]


### 3.) Name your ELB

Put *myname*-elb under *Load Balancer Name*. Under *Create ELB Inside*, select *My Default VPC*.

![Image][2-1-3-lbname]


### 4.) Set your load balancing protocol

We'll leave the listener configuration intact for the time being - but do click on "Load Balancer Protocol". You should see a couple of options - load balancers are powerful!

![Image][2-1-4-listeners]


### 5.) Create security groups for your ELB

On the next page, choose the option "Create a new Security Group". Take note that it's similar to the security groups you made for your instance earlier - similar to a "firewall" where you can say what kind of traffic is allowed.

![Image][2-1-5-secgroups]

We'll leave things default for now, except for "Source" - for which we'll choose "My IP".

### 6.) Skip the next page, as we have no SSL settings.

### 7.) Configure your health checks

On the next page, we'll be configuring our *health checks*. Health checks essentially verify if the instance is "healthy" - if it's accepting traffic, or it's not receiving errors. Have a look at the tooltips for each of the options.

![Image][2-1-6-healthchecks]


For now, we'll choose *TCP* as the Ping Protocol, using *80* as the Ping Port. What this is saying, is that the Load Balancer will check port 80 of any attached instances, and mark it as "healthy" if it responds without errors.



### 8.) Set the instances to forward to

On the instances page, we'll select the instance that you created in the previous module. This essentially means that we'll be forwarding any traffic that hits the load balancer to your instance.

![Image][2-1-7-instances]


### 9.) Add tags

As usual, tags. We'll put a Key of *Name* and a Value of *myname*-elb.

![Image][2-1-8-tags]


### 10.) Finish up!

Review all the settings you've had, then click *Create*

![Image][2-1-9-review]


### 11.) See your ELB details

Click on the link that goes to your ELB. As with the instance, you'll see a pane that details the properties of your ELB. Make note of the *DNS Name*, marked below.

![Image][2-1-10-details]



## Reinstall Wordpress using RDS

Because Wordpress keeps its URL data in the database, we'll need to reset your Wordpress installation. If your SSH session is still open then you're good to go - otherwise, go to the SSH instructions here.

### 12.) Reset Wordpress Config

Using your Terminal or Putty, go to the Wordpress directory, then remove the wp-config.php file:

```
cd /var/www/html/
sudo rm wp-config.php
```

### 13.) Reconfigure Wordpress database

Now, open your browser and paste the *DNS Name* that you had in Step 11. This should show you an installation page. Proceed with the installation, but when you get to the panel that asks you for your database details, put in the following:

```
Database Name: devopsgirlsdb
Database User: devopsgirls
Database Password: devopsgirlsrds
Database Host: rds.devopsgirls.internal
Database Prefix: firstname_
```

It should look like this:

![Image][2-1-11-wpsql]


Make sure you replace 'firstname_' with your first name. For example, 'Banana Smith' would have the database prefix of `banana`.

### 14.) Finish the installation

Finish the install, until it forwards you to the Blog Post page. Now that things are ready, we can start rolling our Wordpress artifact.

## Creating the artifact

### 15.) Create a copy of your Wordpress directory

If the installation went well, then you're going to want to create a copy of your installed Wordpress directory, so you won't have to run the install again. We do this via the Terminal or Putty application. We're going to use the following commands:

```
cd /var/www/html/
sudo tar cvfz ~/firstname.lastname-wordpress.tgz .
```

With this command, changing directories to */var/www/html* - where Wordpress is installed. Then, we create a compressed tar file - firstname.lastname-wordpress.tgz - containing the contents of our current directory (.)

### 16.) Upload the file to S3

S3 is an object store - essentially allowing you to upload files to a directory that you can share either within the account, or to the world. We do this by running the following commands - one to set the maximum size of the upload, and one to *copy* the file to S3 ( *s3 cp* ).


```
aws configure set default.s3.multipart_threshold 64MB
aws s3 cp ~/firstname.lastname-wordpress.tgz s3://devopsgirls-training/firstname.lastname-wordpress.tgz --no-sign-request
```

Depending on the AWS login that you used ( `devopsgirls`, `devopsgirls-2`, or `devopsgirls-3`, you may need to change the S3 bucket to upload to. `devopsgirls` accounts need to use `devopsgirls-training`, `devopsgirls-2` accounts need to use `devopsgirls-training-2`, and `devopsgirls-3` accounts need to use `devopsgirls-training-3`. For example, for a `devopsgirls-2` account:

```
aws configure set default.s3.multipart_threshold 64MB
aws s3 cp ~/firstname.lastname-wordpress.tgz s3://devopsgirls-training-2/firstname.lastname-wordpress.tgz --no-sign-request
```

### 17.) Confirm the file exists using the web console:

Go to *Services > S3*. Click on the bucket called *devopsgirls-training*. If you uploaded your file correctly, then it should be there!

![Image][2-1-12-s3]


## Create a second instance

So: now we have a Load Balancer, and an Artifact. What we can do now is make it so that you have multiple instances, so that your service can still be up if one of the instances is down.

### 18.) Create a second instance

Go to *Services > EC2* in the web console. As with the first module, we're creating another instance. Click on Launch Instance.

![Image][2-1-13-launchinstance]


### 19.) Set instance details

On Step 1, choose the *Amazon Linux AMI*. On the Instance Type, select *t2.micro*.

![Image][2-1-14-ami]


### 20.) Set User Data

You can think of *User Data* as scripts that you can run for your EC2 instance when it launches. We can use User Data to declare what we want to do - in this case, we're declaring the same commands that we used to install Wordpress the first time - except you don't have to login and do it manually anymore.

On the *Advanced Details* tab of *Step 3: Configure Instance Details*, paste the following into the *User Data* box:

```
#!/bin/bash
yum install -y mysql php php-mysql httpd
aws configure set default.s3.multipart_threshold 64MB
aws s3 cp s3://devopsgirls-training/firstname.lastname-wordpress.tgz /var/www/wordpress.tgz --no-sign-request
tar xvfz /var/www/wordpress.tgz -C /var/www/html/
chown -R apache /var/www/html/
service httpd start
```

Again, make sure that you change the S3 bucket name (`devopsgirls-training`, `devopsgirls-training-2`, or `devopsgirls-training-3`). Your configuration should look like this:

![Image][2-1-15-userdata]


### 21.) Continue the instance configuration

For the rest of the instance configuration, specify the following:

```
 - Storage: Defaults
 - Tags:
     Key:Name
     Value: myname-wordpress2
```

![Image][2-1-16-tags]


### 22.) Change the security group configuration

Because your instance is only supposed to be accepting traffic from your Load Balancer, you can now specify your load balancer as a source. This means that you're making your instances more secure by narrowing down the kind of traffic it can receive.

Additionally, you no longer need to specify SSH access - because all the configuration is done via your User Data. For your security group configuration, choose *Create a new security group* and specify the following:

```
 -  Type: HTTP
 -  Source: your ELB
```

It should look like this:

![Image][2-1-17-elbsg]


### 23.) Launch your instance and go to Load Balancers

Click on *Launch instance* to launch your instance. Now, go back to the Load Balancer section ( *Services > EC2 > Load Balancers* ). Select the Load Balancer you created prior, then click on *Instances* on the lower pane.

![Image][2-1-18-instances]


### 24.) Add your new instance to the Load Balancer

Click on *Edit Instances*,  then add the new instance you just created (myname-wordpress-2). Click on *Save*.

![Image][2-1-19-addinstance]


### 25.) Check that everything is working perfectly

Go to the *Description* tab of your load balancer. If everything worked well, it should say "*2 of 2 instances in service*". You can test this by:

 - Refresh your browser with the Wordpress window open

 - In the AWS console (Services > EC2), right click on one of your instances and select *Stop Instance*.

 - If everything worked well, then your site should still be up even if you refresh your browser.

### 26.) And, done!

![Image][2-1-20-blog]


Congratulations! You now have a more durable service configuration.





[2-1-1-create]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-1-create.png
[2-1-10-details]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-10-details.png
[2-1-11-wpsql]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-11-wpsql.png
[2-1-12-s3]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-12-s3.png
[2-1-13-launchinstance]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-13-launchinstance.png
[2-1-14-ami]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-14-ami.png
[2-1-15-userdata]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-15-userdata.png
[2-1-16-tags]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-16-tags.png
[2-1-17-elbsg]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-17-elbsg.png
[2-1-18-instances]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-18-instances.png
[2-1-19-addinstance]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-19-addinstance.png
[2-1-2-classic]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-2-classic.png
[2-1-20-blog]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-20-blog.png
[2-1-3-lbname]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-3-lbname.png
[2-1-4-listeners]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-4-listeners.png
[2-1-5-secgroups]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-5-secgroups.png
[2-1-6-healthchecks]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-6-healthchecks.png
[2-1-7-instances]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-7-instances.png
[2-1-8-tags]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-8-tags.png
[2-1-9-review]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/2-1-ELB/2-1-9-review.png
