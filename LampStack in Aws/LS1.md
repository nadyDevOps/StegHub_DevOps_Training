## Introduction:

_The LAMP stack is a popular open-source web development platform that consists of four main components: Linux, Apache, MySQL, and PHP (or sometimes Perl or Python). This documentation ouatlines the setup, configuration, and usage of the LAMP stack.__

## Step 0: Prerequisites

__1.__ Launched an EC2 Instance of t2.micro type and Ubuntu 24.04 LTS (HVM) in the ap-south-1 region using the AWS console.



__2.__ Created SSH key pair named __nadyDevOpsKP__ to access the instance. Installed OpenSSH server used for connecting to the instance.

__3.__ The key pair was located and then connected to the instance by running,
```
cd Downloads

ssh -i "nadyDevOpsKP.pem" ubuntu@3.108.60.92
```

# Step 1 - Install Apache and Update the Firewall

__1.__ __ Ran commands to update and upgrade list of packages in Ubuntu's package manager__

```
sudo apt update
sudo apt upgrade -y
```

__2.__ __Ran apache2 package installation__
```
sudo apt install apache2 
```


__3.__ __Enable and verify that apache is running using the following command.__
```
sudo systemctl enable apache2
sudo systemctl status apache2
```
If in the response in terminal, it shows green and running, then apache2 is correctly installed


__4.__ __The server is running and can be accessed locally in the ubuntu shell by running the command below:__

```
curl http://localhost:80
OR
curl http://127.0.0.1:80
```


__5.__ __Test with the public IP address if the Apache HTTP server can respond to request from the internet using the url on a browser.__

```
http//:<Public-IP-Address>:80
http://3.108.60.92:80
```

If in the browser it shows a page titled Apache2 Ubuntu Default Page, it means that the web server is correctly installed and it is accessible throuhg the firewall.

## Step 2 - Install MySQL

__1.__ __Install a database management system(DBMS)__

MySQL is a popular relational database management system used within PHP environments. The following command will successfully install MySQL.
```
sudo apt install mysql-server
```

This connects to the MySQL server as the administrative database user __root__ infered by the use of __sudo__ when running the command.

__2.__ __Set a password for root user using mysql_native_password as default authentication method.__

Here, the user's password is "Admin0001$", which can be set as required.
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin0001$';
```

After this, must exit the MySQL shell, for the next command.
```
exit
```
__3.__ __Ran a security script to secure MySQL__

The security script comes pre-installed with mysql. This script removes insecure settings and locks down access to the database system for safety.
```
sudo mysql_secure_installation
```
This prompts for VALIDATE PASSWORD PLUGIN.This is for safety of password. If enabled, then must follow strict rules for password strength else, will get rejected. I left this disabled. Then there is prompt for password strength. I selected 0. After this, set the password for MySql root user. This is for previleges of databas esystem. Then presses Y for all the next options, change the root password, remove anonymous users and test database, disable remote logins,load these new rules. This is for removing general access into database.

__6.__ __After changing root user password, log in to MySQL console.__

With the following command, there is prompt for password, and I suuccessfully logged into MySQL with the password created in the previous step.
```
sudo mysql -p
```
## Step 3 - Install PHP

Apache is installed to serve web content and MySQL is installed to store and manage our data.
PHP processes the code to display dynamic content to the end user.

The following needs to be installed:
- php package
- php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.
- libapache2-mod-php, to enable Apache to handle PHP files.
```
sudo apt install php libapache2-mod-php php-mysql
```

Confirm the PHP version
```
php -v
```
Now LAMP stack is completely installed and fully operational.

To test the set up with a PHP script, it's best to set up a proper Apache Virtual Host to hold the website files and folders. Virtual host allows to have multiple websites located on a single machine and it won't be noticed by the website users.

## Step 4 - Create a virtual host for the website using Apache

__1.__ __Created the directory for domain projectlamp.__

Apache by default  serves documents from /var/www/html directory.Created a new directory for domian projectlamp using "mkdir" command.
```
sudo mkdir /var/www/projectlamp
```

__Assigned directory ownership with $USER environment variable which references the current system user.__
```
sudo chown -R $USER:$USER /var/www/projectlamp
```


__2.__ __Created and opened a new configuration file in apache’s “sites-available” directory using vim.__
```
sudo vim /etc/apache2/sites-available/projectlamp.conf
```
Then pressed i to enter insert mode.
And pasted in the bare-bones configuration below:
```
<VirtualHost *:80>
  ServerName projectlamp
  ServerAlias www.projectlamp
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/projectlamp
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Then pressed esc button, :wq and enter to save and close the file.


__3.__ __To see the new file in sites-available__
```
sudo ls /etc/apache2/sites-available
```
```
Output:
000-default.conf default-ssl.conf projectlamp.conf
```


With the VirtualHost configuration, Apache will serve projectlamp using /var/www/projectlamp as its web root directory.

__4.__ __Enable the new virtual host__
```
sudo a2ensite projectlamp
```

__5.__ __Disable apache’s default website.__

This is because Apache’s default configuration will overwrite the virtual host if not disabled. This is required if a custom domain is not being used.
```
sudo a2dissite 000-default
```

__6.__ __Ensure the configuration does not contain syntax error__

The command below was used:
```
sudo apache2ctl configtest
```

__7.__ __Reload apache to save the changes.__
```
sudo systemctl reload apache2
```

__8.__ __The new website is now active but the web root /var/www/projectlamp is still empty. Create an index.html file in this location so to test the virtual host work as expected.__
```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```

__9.__ __Open the website on a browser using the public IP address.__
```
http://3.108.60.92:80
```

__10.__ Open the website with public dns name (port is optional)
```
http://<public-DNS-name>:80
```

This file can be left in place as a temporary landing page for the application until an index.php file is set up to replace it. Once this is done, the index.html file should be renamed or removed from the document root as it will take precedence over index.php file by default.

## Step 5 - Enable PHP on the website

With the default DirectoryIndex setting on Apache, index.html file will always take precedence over index.php file. This is useful for setting up maintenance page in PHP applications, by creating a temporary index.html file containing an informative message for visitors. The index.html then becomes the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root bringing back the regular application page.
If the behaviour needs to be changed, /etc/apache2/mods-enabled/dir.conf file should be edited and the order in which the index.php file is listed within the DirectoryIndex directive should be changed.

__1.__ __Open the dir.conf file with vim to change the behaviour__
```
sudo vim /etc/apache2/mods-enabled/dir.conf
```

```
<IfModule mod_dir.c>
  # Change this:
  # DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
__2.__ __Reload Apache__

Apache is reloaded so the changes takes effect.
```
sudo systemctl reload apache2
```

__3.__ __Created a php test script to confirm that Apache is able to handle and process requests for PHP files.__

A new index.php file was created inside the custom web root folder.

```
vim /var/www/projectlamp/index.php
```

__Added the text below in the index.php file__
```
<?php
phpinfo();
```


__4.__ __Now refresh the page__

This page provides information about the server from the perspective of PHP. It is useful for debugging and to ensure the settings are being applied correctly.

After checking the relevant information about the server through this page, It’s best to remove the file created as it contains sensitive information about the PHP environment and the ubuntu server. It can always be recreated if the information is needed later.
```
sudo rm /var/www/projectlamp/index.php
```

## CONCLUSION <br>
Lamp stack is a popular open-source software bundle used for web development. it consist of four main components:
_Linux: The operating system where the web sewrver runs_<br>

_Apache: The web server software that serves web pages to users._<br>

_MySQL: The relational database management system(RDBMS) used to store and manage data._<br>

 _PHP: The server-side scripting language used to generate dynamic web content._<br>
