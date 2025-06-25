# AWS-wordpress-two-tier-app using Ubuntu EC2 instance

Use Case:
MercyReads is an organization that publish books in large quantities. To improve customer engagement and streamline operations, they decides to launch a new website using wordpress as their content management system (CMS). The goal is to create a secure, reliable and scalable site that will support their growing organization and serve as the main platform for all departments.

Two-Tier Architecture:
A two-tier architecture separates the application into two layers: the web server and the database server. This setup improves ease of management, scalability and security .

Web Tier: The web tier consists of an EC2 instance running Apache and wordpress. It serves web pages and handles user interactions.
Database Tier: The database tier uses an RDS instance with MySQL to store and manage application data in a secure way.
Setting up Network
1. Create a Virtual Cloud Network (VPC)
Go to the VPC dashboard and select “Create VPC” the in US-East-1 region
Choose “VPC only” for manual setup
Use the CIDR block ‘10.0.0.0/16’ for IPv4
Name the VPC and complete the setup
Subnets are created after vpc has been established.
2. Create Subnets
Create three subnets with the VPC earlier created: one public and one private
Public Subnet: AZ us-east-1a, CIDR block 10.0.0.0/24
Private Subnet1: AZ us-east-1a, CIDR block 10.0.1.0/24
Private Subnet2: AZ us-east-1b, CIDR block 10.0.2.0/24
3. Internet Gateway and Route tables
Internet traffic will be needed, so we’ll create an Internet Gateway (IGW)
Create an IGW and attach it to the VPC
Create a private route table and a public route table and associate them with the private subnets and public subnet respectively
Create Internet Gateway
Select the initial VPC created and click on the attach internet gateway button.
Create the public route table:
Edit the public subnet route table to make it exactly public by adding a route the grants public access pointing to the internet gateway attached to the VPC and save changes
Private route tables created here:
Associate the route tables (private and public route table with their respective subnets accordingly (private route table with private subnet, public route table with public subnet).
Navigate to the route table section, click on the public route table, then navigate to subnet association edit and associate the public route table with the public subnet then save the association.
Do the same for private subnet and private route table.
4. Create security group for the webserver
By default, you get a default security group when you create any VPC, you should create your own security group
The description should be allow http and ssh access.
Select the VPC
Allow two inbound rules; one is http with source as anywhere and the second is ssh to access EC2 for administrative purpose with the source as My IP address or company network address.
Then click create security group button
WordPress Database Creation with AWS RDS
1. Create a subnet group
Login to AWS management console, type rds in the search bar, navigate to subnet group and follow the prompt to create the subnet group.
Choose the two private subnet with different availability zones on creating your subnet group
After creating the subnet group, navigate to databases
create databases by choosing all necessary aspect (ensure you choose the free-tier if you are using this for a demo)
2. Setup security group for database
Create an inbound rule that has type as MySQL/Aurora, source as custom and its attached to the webserver security group
Launching and Configuring a Linux EC2 Instance on the webserver where we will launch MercyReads Organization application
1. Create an EC2 Instance
Create an Ubuntu EC2 instance in the public subnet
Choose the instance type-t2 micro
Create a key pair
Assign the correct VPC and Public Subnet
Edit your network settings:
select your VPC and the public subnet you want to put your server in.
enable auto assign public IP
select the security group created (i.e existing security group)
configure your storage and launch your instance.
Connect to your EC2 instance using any of the following:
EC2 instance connect, Session Manager and SSH Client.
I will use EC2 instance connect:
2. Launching, configuring and installing Wordpress on a Linux EC2 instance at MercyReads
Run the following commands one step at a time
sudo apt update
sudo apt install apache2 (to install apache web server)
sudo systemctl start apache2 (to start the web server)
sudo systemctl status apache2 (to check the web server status)
sudo systemctl enable apache2 (to start automatically on boot)
you can access the default apache page by visiting http://your_server_ip in your web browser
Verify if your webserver is up by copying your instance ip (public ipv4 address) to paste on a browser.
3. Wordpress Setup on EC2 Instance
sudo apt install mysql-server (to install mysql)
sudo mysql_secure_installation
sudo apt install php libapache2-mod-php php-mysql
Create a new Apache configuration file for your Wordpress site
sudo nano /etc/apache2/sites-available/wordpress.conf
Add the configuration below
<VirtualHost *:80>
ServerName your_domain_or_ip
DocumentRoot /var/www/wordpress
<Directory /var/www/wordpress>
Options Indexes FollowSymLinks MultiViews
AllowOverride All
Require all granted
</Directory>
</VirtualHost>
Replace your_domain_or_ip with your domain name or your EC2 instance public ip address
sudo a2ensite wordpress.conf (to enable the new configuration)
sudo systemctl restart apache2
If you encounter password issues, you can use (sudo -i systemctl restart apache2)
sudo mysql -u root -p (to log in to Mysql)
CREATE DATABASE wordpress; (to create a new database)
CREATE USER ‘user’@’%’ IDENTIFIED BY ‘password’; (replace user and password with your preferred username and password — to create a new mysql user.
GRANT ALL PRIVILEGES ON wordpress.* TO ‘user’@’%’; (to grant privileges)
FLUSH PRIVILEGES (to flush privileges)
EXIT; (to exit mysql)
sudo wget https://wordpress.org/latest.tar.gz -P /tmp (to download wordpress)
cd /tmp
sudo tar -xvf /tmp/latest.tar.gz -C /var/www/ (to extract wordpress)
sudo chown -R www-data:www-data /var/www/wordpress (to change ownership)
cd /var/www/wordpress
sudo mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php
sudo nano /var/www/wordpress/wp-config.php
scroll down
Update the database settings:
define(‘DB_NAME’, ‘wordpress’);
define(‘DB_USER’, ‘wordpressuser’);
define(‘DB_PASSWORD’, ‘password’);
define(‘DB_HOST’, ‘localhost’);
Replace:
- wordpress with your database name (if different)
- user with your database username
- password with your database password
- localhost with your database end point
After configuring wp-config.php, you can access your WordPress site by visiting http://your_ec2_instance_public_dns/wordpress in your web browser and follow the WordPress installation wizard.
https://api.wordpress.org/secret-key/1.1/salt/ (this is copied and used as credentials to replace the keys below)
3. Wordpress Setup on EC2 Instance
sudo apt install mysql-server (to install mysql)
sudo mysql_secure_installation
sudo apt install php libapache2-mod-php php-mysql
Create a new Apache configuration file for your Wordpress site
sudo nano /etc/apache2/sites-available/wordpress.conf
Add the configuration below
<VirtualHost *:80>
ServerName your_domain_or_ip
DocumentRoot /var/www/wordpress
<Directory /var/www/wordpress>
Options Indexes FollowSymLinks MultiViews
AllowOverride All
Require all granted
</Directory>
</VirtualHost>
Replace your_domain_or_ip with your domain name or your EC2 instance public ip address
sudo a2ensite wordpress.conf (to enable the new configuration)
sudo systemctl restart apache2
If you encounter password issues, you can use (sudo -i systemctl restart apache2)
sudo mysql -u root -p (to log in to Mysql)
CREATE DATABASE wordpress; (to create a new database)
CREATE USER ‘user’@’%’ IDENTIFIED BY ‘password’; (replace user and password with your preferred username and password — to create a new mysql user.
GRANT ALL PRIVILEGES ON wordpress.* TO ‘user’@’%’; (to grant privileges)
FLUSH PRIVILEGES (to flush privileges)
EXIT; (to exit mysql)
sudo wget https://wordpress.org/latest.tar.gz -P /tmp (to download wordpress)
cd /tmp
sudo tar -xvf /tmp/latest.tar.gz -C /var/www/ (to extract wordpress)
sudo chown -R www-data:www-data /var/www/wordpress (to change ownership)
cd /var/www/wordpress
sudo mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php
sudo nano /var/www/wordpress/wp-config.php
scroll down
Update the database settings:
define(‘DB_NAME’, ‘wordpress’);
define(‘DB_USER’, ‘wordpressuser’);
define(‘DB_PASSWORD’, ‘password’);
define(‘DB_HOST’, ‘localhost’);
Replace:
- wordpress with your database name (if different)
- user with your database username
- password with your database password
- localhost with your database end point
After configuring wp-config.php, you can access your WordPress site by visiting http://your_ec2_instance_public_dns/wordpress in your web browser and follow the WordPress installation wizard.
https://api.wordpress.org/secret-key/1.1/salt/ (this is copied and used as credentials to replace the keys in the wp-config.php)
To allow W3TC plugin write the configuration data into the database and paste this command — define( ‘W3TC_CONFIG_DATABASE’, true ) ; at the end of the script.
3. Install wordpress through the browser:
open your webserver, enter your ec2 instance’s public IP address/wp-admin or domain name (http://ec2 instance’s public IP address/wp-admin)
follow the wordpress setup wizard
Summary
Successfully created a two-tier architecture to ensure scalability, security, and efficient operation for the MercyReads website.
Created a Virtual Private Cloud (VPC) with public and private subnets for network organization
Configured an Internet Gateway for secure internet access
Deployed an Ubuntu EC2 instance in the public subnet to host the web server
Set up a MySQL RDS instance in the private subnet for database management
Installed Wordpress on the EC2 instance, linking it with the RDS database
Wordpress accessible





