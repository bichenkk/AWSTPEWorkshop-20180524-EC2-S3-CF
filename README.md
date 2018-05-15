![AWS Workshop Series](http://d4r6ocegxjghf.cloudfront.net/AWSWorkShopSeries.jpg)

# AWS Workshop Series (2018-05-24) EC2-S3-Cloudfront
### Repo: [https://github.com/juntinyeh/AWSWorkshop-20180524-EC2-S3-CF]

Today we are going to practice a very simple scenario, leverage Cloudfront for your S3 data source. 
This is a very common usage when your website serving static assets globalwised.

For this workshop, we support 3 different region: 
* Tokyo(ap-northeast-1)
* Sydney(ap-southeast-2) 
* London(eu-west-2)

We pick these region becase later we will deploy cloudfront distribution, which can obviously see the difference after CDN enabled.

### Step 1:
Switch Region on the AWS console, a drag down menu near right-up corner.
* Tokyo(ap-northeast-1)
* Sydney(ap-southeast-2) 
* London(eu-west-2)
``` If possible, choose the region which geographically opposite to you on this planet.```

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
* check the awscli column on the cloudformation output sheet. And you will see a message like this:
```   aws s3 cp sydney.jpg s3://xxxxxxxxx/ ```
* run this command in your EC2 shell, and you will find a simple response like:
![AWS Workshop Series - awsclicptos3](http://d4r6ocegxjghf.cloudfront.net/awsclicptoS3.png)

* Now, chekc the image file sydney.jpg is in S3 bucket.
* AWS Cosole > S3 > `YOUR_S3_BUCKET` 
![AWS Workshop Series - imageins3bucket](http://d4r6ocegxjghf.cloudfront.net/imageins3bucket.png)

Click on the file and check the detail, you can find a link for this image.
![AWS Workshop Series - imageins3bucketlink](http://d4r6ocegxjghf.cloudfront.net/imageins3bucketlink.png)

Click the image link, you will find the request was failed because AccessDenied.
![AWS Workshop Series - AccessDenied](http://d4r6ocegxjghf.cloudfront.net/AccessDenied.png)

No worry, that's what exactly we want to see: To put static assets into S3 bucket, and serve the request through cloudfront.
Now we are going into second phase, create cloudfront to distribute your static assets, and also help to speed it up globally.

### Step 6:
####Manually create a Cloudfront Distribution 

* AWS Console > Cloudfront > Create Distribution > Web
* Origin Domain Name > - Amazon S3 Bucket - > `THE_BUCKETNAME_IN_THIS_WORKSHOP`.s3.amazonaws.com
* Restrict Bucket Access > Yes
* Origin Access Identity > Use an Existing Identity
* Your Identities > access-identity-
* Grant Read Permissions on Bucket > Yes, Update Bucket Policy
* And leave all the other as default > Create Distribution.

It will take about 15-20 minutes to create this distribution, we can take a break and get a coffee now.


### Step 7:
#### Check your distribution readiness
* Find the domain name for your cloudfront distribution
* AWS Console > Cloudfront > Distributions > `YOUR_CF_DISTRIBUTION` > Domain Name
* Try to open the image through browser http://`YOUR_CF_DOAMIN`/sydney.jpg , and please check the developer tool with the network request/response, expend the response header, you could find the information like this:
![AWS Workshop Series - ImageCFCacheHit](http://d4r6ocegxjghf.cloudfront.net/imagecfcachehit.png)
And there is an header saying:  `X-Cache: Hit from cloudfront` , which telling you now the traffice started from S3, and through delivered through cloudfront distribution.

#### To verify if that really different? (Optional)
* You might already know that: S3 can also serve the web page and image, why should I circle this to use one more service? Now you can start to do an experiment: Try to access the image in S3 directly, and compare the access speed.
* AWS Cosole > S3 > `YOUR_S3_BUCKET` > Click on the image > Make Public
* And then get the link on the image detail, and try again on your browser. 
* Through the developer tool, you can compare with the speed. (If you did choose the long distance region for your source.)

## Till here, you already know how to separate your static asset onto S3 and leverage Cloudfront for global distribution. 
### What's next?
We setup two different option for workshop challenge:

a. A Wordpress website. 
(For teams you now using EC2 serving everything, you should try this first. No matter you use Wordpress, Python-Flask, Python-djongo, node-express or Apache+PHP, this challenge is help to practice again through step-4 to step-6, then you could apply the same concept back to your own workload.)

b. An On-Demand Video Streaming Page.
(For teams who plan to handle online streaming, or who already notice that we have another cloudfront distribution type: RTMP, welcome to give a try on this.)


### Track a:
You might want to know how exactly we separate the static asset on a online website, like WordPress.
1. Install the WordPress from sketch on EC2, which will require mysql-server, PHP, and also the Wordpress source code.
```
install mysql-server on your EC2
you might need:
yum install mysql mysql-server

install php related package, you might need:
yum install php php-mysql php-devel php-gd php-pspell php-xml
yum install php-common
yum install php-curl

And then, you might need to clone the wordpress onto your /var/www/html/:
git clone https://github.com/WordPress/WordPress

Create DB for Wordpress:
>mysql -u root -h localhost
mysql> create db wordpress;
mysql> create user wordpress@'localhost' identified by 'password';
mysql> grant all privileges onto wordpress.* to wordpress@'localhost';
mysql> flush privileges;
mysql> exit;
```
2. And then make sure all the configurate was aligned with /var/www/html/WordPress/wp-config.php
Now you can check with your wordpress page, see the wp-admin console.
3. Next, install a wordpress plug-in `CDN Enabler`, put `YOUR_CF_DOMAIN` into Plug-in Setting.
You can find this plug-in will redirect wp-content,wp-include to cloudfront distribution, so don't forget to dump the static assets to S3 bucket. The command is pretty much like what we did in step 5:
```
> aws s3 cp wp-content s3://YOUR_S3_BUCKET/ --recursive
```
4. Finally, you can find all the assets traffic been redirected as what we did in our workshop.


### track b:

To build up a cloudfront distribution for online streaming, you might need:
1. Create one more cloudfront distribution with RTMP option, you can either create a new bucket to serve the data or just put it into the same bucket we created for this workshop.
2. Get a mp4 file from somewhere like ```http://file-examples.com/index.php/sample-video-files/sample-mp4-files/``` or own by your self. Put it onto your S3 bucket.
3. Now, you can verify the rtmp streaming using existed streaming player. We take VLC player on mac for example:
   VLC > File > Open Network > ```rtmp://MY_S3_BUCKET_FOR_RTMP_CF```/cfx/st/```THE_MP4_FILENAME.mp4 ```
   * Please remember we need to put /cfx/st/ between distribution site and file name. You can refer to [https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Streaming_URLs.html]
4. If you want to build your own online player page, you can follow the instruction on  [https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/TutorialStreamingJWPlayer.html]
 , and then check your online player work or not. Recently we know that rtmp will require flash player, so it might get you some time to get a browser that still support flash-player.


### Note:
In particular scenario, we might face some cross origin reference (CORS) permission issue, then we need to take a look on following page:
* https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html
* https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/header-caching.html#header-caching-web-cors
* And it mentioned "Configure CloudFront to forward the following headers: Origin, Access-Control-Request-Headers, and Access-Control-Request-Method."

