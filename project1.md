# LAMP STACK IMPLEMENTATION

## Installing apache and updating the firewall
#### I did a spin up on my new AWS ec2 instance (Ubuntu server )
#### i did an update on my server (sudo apt update)
#### I installed apache server ( sudo apt install apache 2)
#### I checked my apache status if its running using (sudo systemctel status apache2) and it was active.
#### I proceeded to installed mysql and tested its installed properly by using ( sudo mysql), this then returns the 
#### mysql console confirm its connection ID
##  INSTALLING PHP
. As apache and mysql has been installed, I need to install the component of the set which will then process code to display dynamic content to render the expected user experience to end users.

⋅⋅⋅To achieve this i have identified the package that will be needed as  php-mysql and libapache2-mod-php .

+ Using this command line below i was able to install all the packages :
  sudo apt install php libapache2-mod-php php-mysql

.then validate the installation using this command php -v


## CREATING A VIRTUAL HOST FOR YOUR WEBSITE USING APACHE
. i will be setting up a domain using the following setup.
+  sudo mkdir /var/www/lampproject
. then assign ownership
+  sudo chown -R $USER:$USER /var/www/lampproject
. now set up the configuration 
+ sudo vi /etc/apache2/sites-available/lampproject.conf
... (<VirtualHost *:80>
    ServerName lampproject
    ServerAlias www.lampproject 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/lampproject
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>)
 ... then save with :wq!
 ...  verify the new file created with sudo ls /etc/apache2/sites-available
 ... Now enable virtual host with sudo apache2ctl configtest
 ... I now created an index.html file within the /var/www/lampproject and then test the virtual host works as expected
 ...(sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/lampproject/index.html
)
.... When i navigate to my url to see what i have just echoed


## Enabling PHP on the Website 
....I am going to modify a file /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:
.(sudo vim /etc/apache2/mods-enabled/dir.conf)
*(<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>)
* Save file and reload apache with sudo systemctl reload apache2
.... Now am going to create an index file (vim /var/www/lampproject/index.php)
.... then paste this file inside this new opened file <?php
.... phpinfo();
* Now i should be able to see my installation has been successful

![apache status](https://github.com/pappizee/mydevopsJourney/blob/9ba221e8ae99f32f82dc225d292ecca4faea68d8/images/project1_apache_status.png)

![mysql console](https://github.com/pappizee/mydevopsJourney/blob/3558605dfea578fa0e460656deda684ac344654d/images/mysql%20console.png)

![virtual host ](https://github.com/pappizee/mydevopsJourney/blob/86cc4e96928dcae42bbf92efcc4484fe86579d10/images/virtualhost.png)