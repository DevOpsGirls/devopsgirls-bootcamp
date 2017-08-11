# Launch Configurations and Auto Scaling Groups

## Key Concepts

Before we get started, here are a couple of loose concepts:

 - *Launch Configurations are a blueprint for how you want to make your instances*. This means that you configure it in the same way that you would configure an EC2 instance, with the difference that you're not actually making one - you're making a *plan* for your instance.

 - *Autoscaling Groups dictate how many instances you want to run.* It has many mechanisms for doing so - via manual intervention or system checks, but combined with Launch Configurations, you can automate the process of making instances.

Now that's out of the way, let's get started!

## What we're going to do

In this practical session, we will:

 - Create a Launch Configuration

 - Specify UserData as we did with EC2 earlier

 - Create an Autoscaling Group

 - Configure Autoscaling group so it attaches instances to the ELB

 - Test scaling up or down
 

## Creating a Launch Configuration


### 1.) Creating your Launch Configuration

Go to Services > EC2, then look for *Launch Configurations* on the left-hand side. Click on *Create Launch Configuration*

![Image][3-1-createlcfg]


### 2.) Set your Launch configuration size, and AMI

We will configure your Launch Configuration in the same way we configure your instance - with the following parameters:

```
 - AMI: Amazon Linux AMI
 - Instance Type: t2.micro
```

![Image][3-1-2-instancetype]


### 3.) Configure Launch Configuration details

We'll then set your Launch Configuration details. Name your Launch Config *myname-launchconfig*.

![Image][3-1-3-iamrole]

Note that this IAM role only allows read-only access to the contents of the S3 bucket you've placed your Wordpress package in - and nothing else. 

### 4.) Set Advanced Details

In the same panel, click on *Advanced Details*. In the *User Data* field, paste the following in:

```
#!/bin/bash
yum install -y mysql php php-mysql httpd
aws s3 cp s3://devopsgirls-training/firstname.lastname-wordpress.tgz /var/www/wordpress.tgz --no-sign-request
tar xvfz /var/www/wordpress.tgz -C /var/www/html/
chown -R apache /var/www/html
service httpd start
```

**Make sure** you change `firstname.lastname-wordpress.tgz` to your name, or the filename of the Wordpress package you sent over. And change the S3 bucket name to your specific account.

![Image][3-1-4-userdata]


### 5.) Disable public IP addresses

In the same panel, select *Do not assign public IP*. This is an important security measure, and separates the instances from the public web - all interaction is via your Load Balancer.

Go to the next page afterwards.

### 6.) Set storage

We don't need to modify the storage, so we'll leave everything in *Add Storage* as default for now.

![Image][3-1-5-storage]


### 7.) Security Groups

We'll do the same thing that we did with your previous EC2 instance. In *Protocol*, select *HTTP* - then set *Source* to *"Custom IP"*. If you type out the name of the Load Balancer you created earlier, it should show up on the list.

![Image][3-1-6-secgroups]


The good thing about this is that for any instances we're creating, we're only ever expecting HTTP traffic *only from your Load Balancer*. This is a good thing.

### 8.) Verify your configuration, select the SSH KeyPair

We'll finish up with a review of your Launch Configuration. Check that everything is valid, then click on *Create Launch Configuration*

![Image][3-1-7-reviewlc]


As with creating EC2 instances, this will ask you to specify a Key Pair. Specify the one you created earlier.

## Creating an AutoScaling Group

### 9.) Create an Autoscaling Group for your Launch Config

Once you click *Create Launch Configuration*, you should see another button called *Create an Autoscaling Group using this Launch Configuration*. Click on it.

![Image][3-1-8-createsgh]


### 10.) Set Autoscaling Group details

In the next panel, specify the GroupPame as *myname-asg*. Set Group Size to *2* instances.

![Image][3-1-9-asgname]


### 11.) Select Network and Subnets

For this exercise, we'll use the VPC marked *default*. If you click on the *Subnet* input box, you should be given two to three subnets to choose from. Select *all of them* as destinations.

![Image][3-1-10-subnets]


Subnets essentially set which Availability Zones to deply your instances under. Think of Availability Zones as datacenters - separate locations in Sydney where your instance will live. You'll want a good spread - this protects your service from outages if say, a datacenter in one Sydney location goes down.

### 12.) Set Load Balancer

If you click on *Advanced Details* under the same pane, you should see a tickbox called *Load Balancing*. This allows you to register the server as *live* and attaches it to the Load Balancer you created earlier.

![Image][3-1-11-elb]


An additional set of options should appear. If you click on the input box for *Classic Load Balancers*, you should be able to select the Load Balancer you created earlier.

### 13.) Keep scaling policies default


We're not going to be playing with Scaling Policies for the time being - so choose *Keep this group at its initial size*. Just keep in mind that you can use policies to watch for the CPU or memory use of your instances - so for example, if the CPU ever goes over 70% usage, you can choose to add more instances automatically.

### 14.) Skip notifications

We'll choose not to send notifications at this time. Think of this as a way of notifying someone if an event occurs - if say, an instance is launched or deleted.

### 15.) Configure tags

As with anything that we do, it's iportant to tag our to-be-created instances as soon as they're created. Set a Key of *Name*, and set the Value to your name.

![Image][3-1-12-tags]


### 16.) Review, and finish up!

You should be met with a section where you can review the changes that have occurred. Click *Create Auto Scaling Group* once you're ready, and it should kick things off.

![Image][3-1-13-review]



## Testing your Autoscaling Group: Replacing Instances

Now, everything should be ready! We have our *blueprint* for creating instances, and we have a *count* of instances to create, so let's see if it's working!

### 17.) Check your Load Balancer again

Go to *Services > EC2 > Load Balancers*. Select the Load Balancer you created previously (*myname-elb*), then look for the description.

![Image][3-1-14-elbinstances]


### 18.) Use the DNS Name to access your Load Balancer

Copy the DNS name (similar to below), and paste it on another tab. You're effectively looking at any of your four instances.

![Image][3-1-15-accesslb]


### 19.) Kill your instances

Go to *Services > EC2 > Instances*. In the search box above, you should be able to type your name - allowing you to search any instances you tagged with your name.

For all of your four instances, *Right-Click > Instance State > Terminate*.

![Image][3-1-16-killec2]


### 20.) Check your ELB

Check the browser tab where you pasted your Load Balancer's DNS name. It should be return with a problem shortly.


### 21.) Watch new instances get created

In the AWS console, check under *Services > EC2 > Instances* every minute or so. What you should see is that eventually, new instances with your name should be created.

![Image][3-1-17-instancereplace]



## Testing your Autoscaling Group: Resizing the number of instances

Now that we know that your instances are getting automatically replaced, we can see what happens when we change the number of instances in your Autoscaling Group.

### 22.) Modify your Autoscaling Group

Go to *Services > EC2 > Autoscaling Group*. Look for the Autoscaling Group you created.

Once you highlight the Autoscaling Group you created, go to the bottom panel and click on *Edit*. Set your *Desired Instances* to 3 and your *Max Instances* to 3. Click on *Save*.

![Image][3-1-18-setasg]


### 23.) Watch new instances get created

In the AWS console, check under *Services > EC2 > Instances* every minute or so. What you should see is that eventually, new instances with your name should be created.


Congratulations! You have an automated scaling service!

## Making everything using Cloudformation

### 24.) Download the Cloudformation template

Download the cloudformation template here. You can open it with your text editor (Notepad, or something similar).

### 25.) Read through the template

What you're going to notice is that everything you've made is basically declared line by line if you scroll down. I know it looks intimidating, but don't worry! This is just a declaration of what things you want build.

### 26.) Go to the Cloudformation section

Go to **Services > Cloudformation**. Click on **Create Stack**.

This will point you to a page where you can upload your template file. Click on **Next**.

![Image][3-1-19-upload]


### 27.) Set the Stack Name and Parameters

Set the `DevopsGirlsUser` parameter. Use your `firstname.lastname` format for this.

Change `WordpressS3Bucket` to the name of your S3 bucket name - for example `devopsgirls-training-2` or `devopsgirls-training-3` depending on which account you were setup in.

### 28.) Review, and click on IAM resources

Click through the rest of the dialogs until you get to the **Review** section. On the text box at the bottom that says "*I acknowledge that AWS CloudFormation might create IAM resources*", tick the boxes.

![Image][3-1-20-iam]

Click on *Create*.

### 29.) See your stack get created

Go to **Services > Cloudformation** and look for the stack name you set earlier. On the dialog box at the bottom, click on *Events*. Refresh it every now and then - and watch as your environment gets created!


Congratulations! Now you've deployed things with code!


[3-1-10-subnets]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-10-subnets.png
[3-1-11-elb]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-11-elb.png
[3-1-12-tags]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-12-tags.png
[3-1-13-review]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-13-review.png
[3-1-14-elbinstances]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-14-elbinstances.png
[3-1-15-accesslb]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-15-accesslb.png
[3-1-16-killec2]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-16-killec2.png
[3-1-17-instancereplace]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-17-instancereplace.png
[3-1-18-setasg]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-18-setasg.png
[3-1-19-upload]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-19-upload.png
[3-1-20-iam]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-20-iam.png
[3-1-2-instancetype]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-2-instancetype.png
[3-1-3-iamrole]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-3-iamrole.png
[3-1-4-userdata]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-4-userdata.png
[3-1-5-storage]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-5-storage.png
[3-1-6-secgroups]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-6-secgroups.png
[3-1-7-reviewlc]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-7-reviewlc.png
[3-1-8-createsgh]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-8-createsgh.png
[3-1-9-asgname]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-9-asgname.png
[3-1-ASG]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-ASG.png
[3-1-createlcfg]: https://raw.githubusercontent.com/DevOpsGirls/devopsgirls-bootcamp/master/images/3-1-ASG/3-1-createlcfg.png
