---
title: WordPress on EC2
author: Will Garbutt

date: 2022-08-11

---
## WordPress on EC2

I originally had this blog hosted on Azure with a borrowed MSDN subscription credit with the idea that I would eventually get around to document what I had learnt. Unfortunately the MSDN account I was using expired before I realised.  
Silver lining is I get to do it all over again, this time in AWS, and im documenting it right away so I dont fall for the same mistake again



## Infrastructure
![](/Images/WordPressonEC2/image1.png)


The networking side of the infrastructure consists of a new VPC with a single AZ Subnet attached. With that I deployed a new Internet Gateway and new route table. I have a security group allowing SSH (Port22) and HTTPS (Port 443)

I deployed a new EC2 instance and attached the networking above, configured Session Manager and attached an existing keypair.  
This was deployed via CloudFormation (Github [here][1])

## Installing WordPress and SQL DB

I am currently going through the AWS Certified Solutions Architect &#8211; Associate training course by Adrian Cantrill (Website [here][2]) and coincidently am up to a video where they do just this process. I have modified Adrian&#8217;s supplied script slightly.

```bash
# DBName=database name for wordpress
# DBUser=mariadb user for wordpress
# DBPassword=password for the mariadb user for wordpress
# DBRootPassword = root password for mariadb
```
### STEP 1 - Configure Authentication Variables which are used below
```bash
DBName='stuffaboutcloudwp'
DBUser='stuffaboutcloudwp'
DBPassword='REPLACEME'
DBRootPassword='REPLACEME'
```
### STEP 2 - Install system software - including Web and DB
```bash
sudo yum install -y mariadb-server httpd wget
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
```


### STEP 3 - Web and DB Servers Online - and set to startup
```bash
sudo systemctl enable httpd
sudo systemctl enable mariadb
sudo systemctl start httpd
sudo systemctl start mariadb
```

### STEP 4 - Set Mariadb Root Password
```bash
mysqladmin -u root password $DBRootPassword
```

### STEP 5 - Install WordPress
```bash
sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
cd /var/www/html
sudo tar -zxvf latest.tar.gz
sudo cp -rvf wordpress/* .
sudo rm -R wordpress
sudo rm latest.tar.gz
```


### STEP 6 - Configure WordPress
```bash
sudo cp ./wp-config-sample.php ./wp-config.php
sudo sed -i "s/'database_name_here'/'$DBName'/g" wp-config.php
sudo sed -i "s/'username_here'/'$DBUser'/g" wp-config.php
sudo sed -i "s/'password_here'/'$DBPassword'/g" wp-config.php   
sudo chown apache:apache * -R
```

### STEP 7 Create WordPress DB
```bash
echo "CREATE DATABASE $DBName;" &gt;&gt; /tmp/db.setup
echo "CREATE USER '$DBUser'@'localhost' IDENTIFIED BY '$DBPassword';" &gt;&gt; /tmp/db.setup
echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" &gt;&gt; /tmp/db.setup
echo "FLUSH PRIVILEGES;" &gt;&gt; /tmp/db.setup
mysql -u root --password=$DBRootPassword &lt; /tmp/db.setup
sudo rm /tmp/db.setup
```

### STEP 8 - Browse to http://your_instance_public_ipv4_ip

</pre>
</div>

### Configuring HTTPS

First step was to point my domain at my EC2 instances public IP address, this is important as the tools we use below needs to verify I own my domain name.

Next I followed [this][3] post made by AWS on how to configure Lets Encrypt on an EC2 instance, near the last section of the post you will see a heading named **Certificate automation: Let&#8217;s Encrypt with Certbot on Amazon Linux 2**

This will install all the components needed to use the free Lets Encrypt service and setup a Crontab to enable automatic cert renewal. Very cool

### Updating PHP from 7.2 to 7.4

Some where in my deployment steps above I managed to install an older version of PHP. I was warned of this when I first logged on to WordPress, stating I was using an unsecure version of PHP and that I needed to update it as soon as possible.  
Another blog to the rescue! I found [this][4] handy blog post that detailed the steps needed to upgrade PHP

### Restoring WordPress from backup

Luckily before my Azure Subscription ran dry I had the wisdom to install a plugin named [Updraft][5] that has a free option to back up your entire WordPress deployment to cloud storage such as AWS S3. I simply had to redownload the plugin on my new deployment, point it towards my existing S3 bucket and restore!
![](/Images/WordPressonEC2/image2.png)

### Cost

Its early days so far but I estimate this will cost me around $10 a month. This is running on a very small t2.micro EC2 which seems to be running things a lot quicker than my Marketplace deployment of WordPress in Azure, which coincidently drew down my monthly credit by $60 a month.

 [1]: https://github.com/wgarbutt/stuffaboutcloud/blob/main/template.yaml
 [2]: https://learn.cantrill.io/
 [3]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/SSL-on-amazon-linux-2.html
 [4]: https://greggborodaty.com/amazon-linux-2-upgrading-from-php-7-2-to-php-7-4/
 [5]: https://updraftplus.com/