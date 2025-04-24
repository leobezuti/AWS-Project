# AWS WordPress Deployment with EC2, RDS, and Load Balancer




## 📋 Project Overview
This project demonstrates a complete WordPress deployment on AWS infrastructure using:
- **EC2** as web servers (Amazon Linux 2)
- **RDS MySQL** for database
- **Application Load Balancer** for traffic distribution
- Proper security group configuration

## 🚀 Deployment Steps

### 1. EC2 Instance Setup
**Configuration:**
- **AMI**: Amazon Linux 2
- **Instance Type**: t2.micro (Free Tier eligible)
- **Security Groups**:
  - Inbound Rules: SSH (22), HTTP (80), HTTPS (443)
  - Outbound Rules: All traffic

**Connection:**
ssh -i "chave.pem" ec2-user@ip-publico-ec2



### 2. Web Server Installation
# Update system and install packages
sudo yum update -y

# Install and run Apache
sudo yum install -y httpd
sudo service httpd start

# Install PHP and Database
sudo amazon-linux-extras install php8.2 -y
sudo amazon-linux-extras install mariadb -y
sudo systemctl start mariadb



### 3. RDS Database Configuration
**Configuration:**
- **Engine**: MySQL
- **Instance Type**: db.t3.micro (Free Tier eligible)
- **Allocated storage**: 15 GB
- **Security Groups**: 
   - Inbound Rules: MYSQL/Aurora (3306) to the EC2 security group (to allow connection from the instance)



### 4. WordPress Installation and Configuration
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Add ec2-user to apache group
sudo usermod -a -G apache ec2-user

# Set permissions
sudo chown -R ec2-user:apache /var/www/
sudo chmod 27755 /var/www/ && find /var/www/ -type d -exec sudo chmod 2775 {} \;
find /var/www -type f -exec sudo chmod 0664 {} \;

# Database Configuration (wp-config.php)
define('DB_NAME', 'wordpress_db');
define('DB_USER', 'admin');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'rds-endpoint.rds.amazonaws.com');

# Allow WordPress to use PermaLinks in Apache
sudo vim /etc/httpd/conf/httpd.conf
Locate the <Directory "/var/www/html"> section and change to:
    AllowOverride None > All
sudo systemctl restart httpd

# Access the website via Public IPV4 address
Finish the configuration of the Wordpress
Customize the site the way that you want with themes, plugins and posts

![alt text](image-1.png)

### 5. Application Load Balancer Creation
Create a load balancer and select 3 availability zones
Create a security group for the load balancer:
  - **Inbound Rules**: HTTP (80), HTTPS (443)
  - **Outbound Rules**: All traffic
Create a target group and put the WordPress instance as a target

# Restricting EC2 Acess to ALB Only:
Navigate to EC2 -> Security Groups
Copy the Security Group ID of your ALB
Edit inbound rules for your EC2 instance's Security Group:
  - Remove existing HTTP/HTTPS rules allowing 0.0.0.0/0
  - Add HTTP/HTTPs and set the source as ALB Security Group

![alt text](image.png)


### 6. Adding a custom domain
You can use Amazon Route 53 to manage your domain, or can also use a custom domain managed by a third-party
In this project the domain **bezuti.cloud** was purchased for Hostinger to use as an example 

# Create New Hosted Zone

# Configure Domain Provider
Copy all 4 NS records from Route 53
Log in to your domain registrar 
Update nameservers to use AWS nameservers

# Create DNS Records
Create a root domain record and choose route traffic to the ALB
Choose the region where the ALB has been created

# Create WWW redirection record
Record name: www
Record type: A - Routes to IPv4 address
Route traffic to:
- Alias to another record in this hosted zone
- Select your root domain record



### 7. SSL/TLS Configuration
Open ACM Console
Click "Request certificate"
  - Select "Request public certificate"
  - Configure domains:
  - Primary domain: yourdomain.com
  - Additional names: *.yourdomain.com 

# Configure load balancer
Navigate to EC2 -> Load Balancers
Select your ALB -> Click "Add listener"
- Protocol: HTTPS
- Port: 443
- Default actions: Forward to [created target group]
Edit HTTP listener
- Action type: Redirect to URL
- Protocol: HTTPS
- Port: 443
- Status code: HTTP_301 (permanent redirect)

# WordPress HTTPS Configuration
Edit wp-config.php file to force HTTPS

cd /var/www/html/
sudo vim wp-config.php

Paste this command:

// Force HTTPS behind load balancer
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}

// Update site URLs
define('WP_HOME', 'https://yourdomain.com');
define('WP_SITEURL', 'https://yourdomain.com');


![alt text](image-2.png)



🚀 Now the blog is running smoothly and securely using AWS. The architecture utilizes an AWS Application Load Balancer (ALB) with Route 53 DNS routing and ACM-provisioned SSL/TLS encryption.
💰 All services used are eligible for AWS Free tier. The only money spended was the amount of R$42,97 for the domain.

Key Notes:
    Free Tier limits apply for 12 months after AWS sign-up.
    Monitor usage to avoid unexpected charges.
    Domain renewal fees will apply annually.