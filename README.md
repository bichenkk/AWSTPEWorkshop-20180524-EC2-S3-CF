![AWS Workshop Series](http://d4r6ocegxjghf.cloudfront.net/AWSWorkShopSeries.jpg)

# AWS Workshop Series (2018-05-24) EC2-S3-Cloudfront
### Repo:AWSWorkshop-20180524-EC2-S3-CF

Today we are going to practice a very simple scenario, leverage Cloudfront for your S3 data source. 
This is a very common usage when your website serving static assets globalwised.

For this workshop, we support 3 different region: 
* Tokyo(ap-northeast-1)
* Sydney(ap-southeast-2) 
* London(eu-west-2)

We pick these region becase later we will deploy cloudfront distribution, which can obviously see the difference after CDN enabled.

### Step 1:
Switch Region on the AWS console, a drag down menu near right-up corner.

### Step 2:
* Check if you already have a EC2 Key pair in your selected region. 
* If not, select EC2 > Key Pairs > Create Key Pair. 
* Remember to download the private key(.pem) and well saved. 
* In usual, we will move it into ~/.ssh/ sub-folder in your home directory.
* To make it secure, remeber to change the privilege with command 
``` chmod 0400 XXXXX.pem ```

### Step 3:
* Create cloudformation stack: Cloudformation > Create Stack > from S3 template >
https://s3-ap-northeast-1.amazonaws.com/workshop-data-public/ec2-s3-iamrole.yaml
* 
* And wait till the stack creation ready, the status will change to `CREATE_COMPLETE`
* you can see the output sheet:
* Then you can use the command to sign into your EC2.
``` ssh ec2-user@XXX.XXX.XXX.XXX -i XXXXXX.pem ```

### Step 4:
* install httpd service on EC2, and prepare the image file for serving.
```
sudo yum install httpd
cd /var/www/html/
wget https://s3-ap-northeast-1.amazonaws.com/workshop-data-public/sydney.jpg
```
* Now you might want to test your httpd image serving status by using browser, open
``` http://XXX.XXX.XXX.XXX/sydney.jpg ```

### Step 5:
#### Copy image file from EC2 to S3, separate the static asset out of EC2.
* use awscli in your EC2 shell environment
* check the awscli column on the cloudformation output sheet. And you will see a message like this:
```   aws s3 cp sydney.jpg s3://xxxxxxxxx/ ```

* Now, open the AWS console and check the S3, you will find the target bucket is there, and the image file sydney.jpg is also there.

Click on the file and check the detail, you can find a link for this image.

Click the image link, you will find the request was failed because AccessDenied.

Now we are going into second phase, create cloudfront to distribute your static assets, and also help to speed it up globally.

### Step 6:
Manually create a Cloudfront Distribution - AWS Console > Cloudfront 
6-1 : Create Distribution > Web
Origin Domain Name > - Amazon S3 Bucket - > THE_BUCKETNAME_IN_THIS_WORKSHOP.s3.amazonaws.com
Restrict Bucket Access > Yes
Origin Access Identity > Use an Existing Identity
Your Identities > access-identity-
Grant Read Permissions on Bucket > Yes, Update Bucket Policy
And leave all the other as default > Create Distribution.

It will take about 15-20 minutes to create this distribution, we can take a break and get a coffee now.
