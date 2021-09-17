# LEMP STACK IMPLEMENTATION

### Installing the NGINX Web Server

 In this project i will be using a high performance web server called 'Nginx' and this can be installed using the ***apt*** package manager to install this package , Using this in the right order, First update the apt then install by doing 
 `sudo apt update` then `sudo apt install nginx` then verify your installation was successful with `sudo systemctl status nginx` and this returns active and running. 
 Now i needed to validate that my tcp port 22 is open on the ec2 instance via ssh then i locally checked i can access my server using `curl http://localhost:80` upon verification check i now checked my server is responding as expected by going back on the browser and enter my public ip address : 80
 ### Installing mysql

 To use a the web server a database management system needs to be installed so as to be able to manage data or optimize if need be so i will be using my apt commands to acquire my packages i would need 
 `sudo apt install mysql-server`  and using `sudo mysql_secure_installation` to help me filter and remove insecure default settings that my be in the downloaded DB system plus lock down access to it as well. Run through the installation with or without password and do a `sudo mysql`

 ### Installing PHP
 For the server to process code and return dynamic content for the web server , I will need to install php and to get my additional configurations set up i need `php-fpm`and `php-mysql` effectively i can install the two at the same by doing `sudo apt install php-fpm php-mysql`
  ### Setting up Nginx to use php processor
  Just like in project 1 setting up the virtual host in apache i will do the same in this instance and i will be creating a web server blocks just to be able to manage multiple sites, So rather than altering /var/www/html am going to create a directory structure within the /var/www for the precise domain website i am working on, This will avoid duplication to be served when a client request isnt for any other sites that might be hosted on that particular server.

  First i will be creating a directory for my domain
  `sudo mkdir /var/www/lempproject`
  Next i will assign ownership of the directory that uses an environmental variable i.e it references whoever is using the system at the time 
  `sudo chown -R $USER:$USER /var/www/lempproject`
  Now i will open a new config file within Nginx **site-available` directory using VI command
  `sudo vi /etc/nginx/sites-available/lempproject`
  Then press 'i' to paste the text in the new blank file opened 

  ```
  #/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name lempproject www.lempproject;
    root /var/www/lempproject;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
Now i will be activating my configuration by linking to the config file which sits within the nginx site enabled  
`sudo  ln -s /etc/nginx/sites-available/lempproject /etc/nginx/sites-enabled/`
Then i needed to validate my configuration against syntax errors using 
`sudo nginx -t`
I need to disable the default nginx host that currently works to listen on port 80
`sudo unlink /etc/nginx/sites-enabled/default`

 Now i reloaded my nginx to reflect my changes 
 `sudo systemctl reload nginx` which i now need to create an index.html within the lempproject root directory for test purposes, To do that i used this code snippet 
 ````sudo echo 'Hello Pappizee's Lemp from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/lempproject/index.html
```` 
Saved the above and check this has been rendered on the web 
  ### Verifying PHP with Nginx

  This piece would just validate what has been installed in the previous installation steps so i would be opening nmy info.php file
  `sudo vi /var/www/lempproject/info.php` then paste 
  `<?php
   phpinfo();


  This should already be visible on the web browser `http://  iP/info.php` and as best practice , I always remove files that were created just to be sure relevant information about the php is not visible to the public
  `sudo rm /var/www/domain/info.php

