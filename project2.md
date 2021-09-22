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
 ````sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/lempproject/index.html
```` 
Saved the above and check this has been rendered on the web 
  ### Verifying PHP with Nginx

  This piece would just validate what has been installed in the previous installation steps so i would be opening nmy info.php file
  `sudo vim /var/www/lempproject/info.php` then paste 
  `<?php
   phpinfo();
 `

  This should already be visible on the web browser `http://172.31.24.154/info.php` and as best practice , I always remove files that were created just to be sure relevant information about the php is not visible to the public
  `sudo rm /var/www/lempproject/info.php`

  ### Retriving Data from the mySql Database using PHP
   On the backend, I would like to create a db that is linked to a todo list application and i can configure access to it, whereby my nginx website would be able to query data from the set DB and display its output.

The first thing that was done was to create a new user with the `mysql_native_password` command method so as to be able to connect to my db via the php, Below are my steps to achieve this :

Connecting to mysql console
`sudo mysql`

Create a new database 
`mysql> CREATE DATABASE` `pappizee_database`;

After creating the database, I then created a user and granted full privilege to the newly created database by doing the below :
ezzee_user with the mysql_native_password been the default auth method, Password as "Welcome01"
`mysql>  CREATE USER 'ezzee_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Welcome01';`
then giving the person to the created user over the created database
`mysql> GRANT ALL ON pappizee_database.* TO 'ezzee_user'@'%';`
The logic above gives my created user full privilege over the created database BUT is limited to modifying or creating other database on the server.
`mysql> exit` to exit the sql console

**validating my new user has the set permission**
`mysql -u ezzee_user -p` , at the password prompt i entered my set password when creating my user and i was successfully logged in the mysql database . Then to view my database content 
`mysql> SHOW DATABASES;`

Once verified,I created a table called *todo_things* by using the below command :
```
CREATE TABLE pappizee_database.todo_things (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```
I then proceeded to inserting a rows in my todo content by updating my insert statement with new values
`mysql> INSERT INTO pappizee_database.todo_things (content) VALUES ("My project 2 in progress");`
`mysql> INSERT INTO pappizee_database.todo_things (content) VALUES ("My project 3 in the works");`
`mysql> INSERT INTO pappizee_database.todo_things (content) VALUES ("Can't wait to start project 4");`
Now i need to verify my inserted values 
`mysql>  SELECT * FROM pappizee_database.todo_list;`

After validation i exited the console 
`mysql> exit`

`vim /var/www/lempproject/todo_things.php`

***Lastly*** I created a php script that will connect to mySQL db and queries for the content of the todo_things table which would then render the result on the screen and catch any exception provided there is any error with database connection or called service.
 `todo_things.php`
 ```
 <?php
$user = "ezzee_user";
$password = "Welcome01";
$database = "pappizee_database";
$table = "todo_things";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
When i navigate to my domain then `/todo_things.php` my page displays as i would expect.