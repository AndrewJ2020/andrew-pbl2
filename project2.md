
# Step 0: Preparing Pre-requisites

**In my AWS account I created a new EC2 Instance of t2.nano family with Ubuntu Server 22.04 LTS (HVM) image**

![prj2](https://user-images.githubusercontent.com/110178748/182403774-5115eb3f-0399-4734-b632-228b3598f4b9.png)

**I created a new instance for project 2 and ssh into the instance**

![prj3](https://user-images.githubusercontent.com/110178748/182404606-a3f1a629-0cf2-4463-af8d-9796df943ec6.png)

# Step 1: Installing the **NGNIX** server 

In order to display web pages to my site visitors, I will employ employ Nginx, a high-performance web server. 
I will use the apt package manager to install this package.

command: sudo apt update

![prj4](https://user-images.githubusercontent.com/110178748/182405397-87774f4d-5312-4ab1-9e52-4937a3566447.png)

command: sudo apt install nginx

![prj5](https://user-images.githubusercontent.com/110178748/182405740-344d54d6-56b5-4131-9ded-56fcedfd3446.png)

To verify that nginx was successfully installed and is running as a service in Ubuntu, I run: sudo systemctl status nginx

![prj6](https://user-images.githubusercontent.com/110178748/182406446-72135802-f887-4825-90c0-ca45671a2b6a.png)

I need to **open TCP port 80**. As we know **TCP port 22** open by default on my EC2 machine to access it via SSH, so I need to 
add a rule to EC2 configuration to open inbound connection through **port 80**

![prj7](https://user-images.githubusercontent.com/110178748/182410493-22e3823a-be72-4e7b-879a-fcb0cc894050.png)

‘curl’ command to request our Nginx on port 80
access it locally in our Ubuntu shell via **DNS name**, run: curl http://localhost:80 
 
![prj8](https://user-images.githubusercontent.com/110178748/182410987-2d43106d-e874-4c8f-a71a-18310cafd039.png)

access it locally in our Ubuntu shell via **IP address**, run: curl http://127.0.0.1:80

![prj9](https://user-images.githubusercontent.com/110178748/182411230-61232533-2d99-45a0-ac9a-6f430171b441.png)

Now it is time to test how Nginx server can respond to requests from the Internet.
I open my chrome web browser and access with follwing url http://54.226.102.116:80

![prj10](https://user-images.githubusercontent.com/110178748/182412443-8c95a77b-7575-4313-b549-3acb697243e5.png)

Another way to retrieve your Public IP address, other than to check it in AWS Web console, is to use following command:

curl -s http://169.254.169.254/latest/meta-data/public-ipv4

![prj11](https://user-images.githubusercontent.com/110178748/182412851-eb601706-b52f-472f-a043-f4fc71b77ed6.png)


#Step 2: Installing **MYSQL**

I use ‘apt’ to acquire and install this software
I run command: sudo apt install mysql-server

![prj12](https://user-images.githubusercontent.com/110178748/182413761-21328a64-a55d-438b-b54b-36eddc963c29.png)

When the installation is finished, I log in to the MySQL console, this will connect to the 
MySQL server as the administrative database user **root**, I type: sudo mysql

![prj13](https://user-images.githubusercontent.com/110178748/182414107-103dfb16-bbb5-41ec-beb5-6f8c688b3df5.png)

I run a security script that comes pre-installed with MySQL.This script will remove some insecure default settings and lock down 
access to my database system. Before running the script I will set a password for the root user, using mysql_native_password 
as default authentication method. We’re defining this user’s password as **PassWord.1**.

Exit the MySQL shell with:  mysql> exit

![prj14](https://user-images.githubusercontent.com/110178748/182415064-dcea7873-e9f7-459d-9354-1875e5cd3577.png)

Start the interactive script by running: sudo mysql_secure_installation and i created a password as specifed on Darey.io but did not 
change the password and folowed instructions as detailed on darey: 
"If you enabled password validation, you’ll be shown the password strength for the root password you just entered and your server will ask if you want to continue with that password. If you are happy with your current password, enter Y for “yes” at the prompt:
Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
For the rest of the questions, press Y and hit the ENTER key at each prompt. This will prompt you to change the root password, remove some anonymous users and the test database, disable remote root logins, and load these new rules so that MySQL immediately respects the changes you have made."

![prj15](https://user-images.githubusercontent.com/110178748/182419055-19b5fb34-baeb-4bfc-8c25-6ab0099c04cc.png)

To test if I am  able to log in to the MySQL console I type: sudo mysql -p
** there is -p flag in the above command, which will prompt me for the password set by Darey**

![prj16](https://user-images.githubusercontent.com/110178748/182419378-cc2caafe-8cba-4548-830b-bbca0f63b9fa.png)

To exit the MySQL console, I type:  mysql> **exit** 

![prj17](https://user-images.githubusercontent.com/110178748/182419949-f26456ac-41a8-4a1b-bc11-febcfca85c7d.png)

my MySQL server is now installed and secured

# Step 3: Installing **PHP**

 I have Nginx installed to serve my content and MySQL installed to store and manage my data. Now I can install PHP to process code 
 and generate dynamic content for the web server.
 
Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. 
This allows for a better overall performance in most PHP-based websites, but it requires additional configuration.

*I will need to install php-fpm, which stands for **“PHP fastCGI process manager”**, and tell Nginx to pass PHP requests to this software 
for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages
will automatically be installed as dependencies.*

To install these 2 packages at once, I run: sudo apt install php-fpm php-mysql

![prj18](https://user-images.githubusercontent.com/110178748/182421673-933981d0-65c8-47bd-883b-ca0cb61bec6e.png)

I now have your PHP components installed. I run:$  php -v

![prj19](https://user-images.githubusercontent.com/110178748/182422049-b828f112-3dfa-42e6-b896-13bcf88b592e.png)

Next, I will configure Nginx to use PHP fastCGI process manager and php-mysql module to communicate to Mysql-based databases. 

# Step 4: Configuring **Nginx** to use **PHP processor**

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. I will use projectLEMP as an example domain name.

On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if I am hosting multiple sites. Instead of modifying /var/www/html, I will create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.


Create the root web directory for your_domain as follows: sudo mkdir /var/www/projectLEMP

Create the root web directory for your_domain as follows: sudo mkdir /var/www/projectLEMP

Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user: sudo chown -R $USER:$USER/var/www/projectLEMP

![prj19 5](https://user-images.githubusercontent.com/110178748/182423882-d319aa5e-a0e7-4538-9799-dd1ef0f26014.png)

Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:

sudo nano /etc/nginx/sites-available/projectLEMP

This will create a new blank file. Paste in the following bare-bones configuration:

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

**see image below**

![prj20](https://user-images.githubusercontent.com/110178748/182423964-5c7db935-41e3-4cfb-b801-8086734ea4c5.png)

And When im done editing, save and close the file. I'm using nano, so i  can do so by typing CTRL+X and then y and ENTER to confirm. 


I next Activate my configuration by linking to the config file from Nginx’s sites-enabled directory: 
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

![prj21](https://user-images.githubusercontent.com/110178748/182424777-170dfc99-6d20-4f46-a334-291b99ea27e1.png)

This will tell Nginx to use the configuration next time it is reloaded. I test your configuration for syntax errors by typing: sudo nginx -t

![prj22](https://user-images.githubusercontent.com/110178748/182425018-99ad6a63-b8ce-414e-a9ce-716da7783314.png)

I also need to disable default Nginx host that is currently configured to listen on port 80, for this I run: sudo unlink /etc/nginx/sites-enabled/default

![prj23](https://user-images.githubusercontent.com/110178748/182425187-28855991-68da-41a6-983d-5941493408e8.png)

I reload Nginx to apply the changes: sudo systemctl reload nginx

![prj24](https://user-images.githubusercontent.com/110178748/182425341-fdb1b712-eda9-4399-a112-c45726861744.png)


My new website is now active, but the web root /var/www/projectLEMP is still empty. I will Create an index.html file in that location so that 
I can test that my new server block works as expected: 
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html 


![prj25](https://user-images.githubusercontent.com/110178748/182425907-fb578278-406e-4953-9063-85fb5dc79fe3.png)


Now I go to my browser of choice , chrome  and try to open your website URL using IP address: http://<Public-IP-Address>:80

http://54.226.102.116:80

![prj26](https://user-images.githubusercontent.com/110178748/182426372-7751d3df-7ea9-43f6-a21e-b21454346b09.png)
 
My LEMP stack is now fully configured. In the next step, I will  create a PHP script to test that Nginx is in fact able to handle .php files within my
newly configured website.
  

# Step 5: Testing **PHP** with **NGINX** 
 
 My LEMP stack should now be completely set up.

At this point, my LAMP stack is completely installed and fully operational.

I can test it to validate that Nginx can correctly hand .php files off to my PHP processor.

I can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:
sudo nano /var/www/projectLEMP/info.php


 ![prj27](https://user-images.githubusercontent.com/110178748/182427264-195242a0-f615-4abe-b8f0-e459f7996530.png)


 
 Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:
<?php
phpinfo();



![prj28](https://user-images.githubusercontent.com/110178748/182427450-0fbd8877-4647-4d7f-b3b0-c477f565a6f3.png)



And When im done editing, save and close the file. I'm using nano, so i  can do so by typing CTRL+X and then y and ENTER to confirm. 



You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:

http://`server_domain_or_IP`/info.php

http://54.226.102.116/info.php



![prj29](https://user-images.githubusercontent.com/110178748/182427964-f52cc2a9-4171-4708-a678-86f5a4a7be21.png)




After checking the relevant information about my PHP server through that page, I remove the 
file I created as it contains sensitive information about my PHP environment and my Ubuntu server. I use rm to remove that file:

sudo rm /var/www/your_domain/info.php

in my case 

sudo rm /var/www/projectLEMP/info.php



![prj30](https://user-images.githubusercontent.com/110178748/182428623-b95c37fd-cc80-4fb2-aef6-e1e501aaae43.png)



therefore:



![prj31](https://user-images.githubusercontent.com/110178748/182428815-a4411c45-c24d-4ef5-9034-ab70de34aa78.png)




#Step 6: RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)

In this step I will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website
would be able to query data from the DB and display it.
I create a database named example_database and a user named example_user, but I can replace these names with different values.



First I connect to the MySQL console using the root account with password flag -p : sudo mysql -p



![prj32](https://user-images.githubusercontent.com/110178748/182429750-a886f4d9-731c-4515-b43c-ca8e418cb508.png)



I ceate a new database, run the following command from your MySQL console: mysql> CREATE DATABASE `example_database`;



![prj34](https://user-images.githubusercontent.com/110178748/182430327-e0dccdf2-c71f-4714-9b22-5a026470aaf8.png)



Now I can create a new user and grant him full privileges on the database I have just created.



The following command creates a new user named example_user, using mysql_native_password 
as default authentication method. I am defining this user’s password as password, but you 
should replace this value with a secure password of your own choosing. **i decided to define users password as PassWord.1**. 

mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';



![prj36](https://user-images.githubusercontent.com/110178748/182431167-e294d144-bf39-4fad-82e5-f9203109ff04.png)



Now we need to give this user permission over the example_database database:



mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';

![prj38](https://user-images.githubusercontent.com/110178748/182432149-85d8df04-c6f5-44b7-a11a-f871357672ca.png)




I can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:



After logging in to the MySQL console, confirm that I have access to the example_database database:



![prj39](https://user-images.githubusercontent.com/110178748/182432725-964466b8-2823-42cc-b409-ce8a84657383.png)



Next, I create a test table named todo_list. From the MySQL console, I run the following statement:
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );



![prk40](https://user-images.githubusercontent.com/110178748/182433386-8c7b099a-487b-448a-9021-f3aa7941563d.png)


I Insert a few rows of content in the test table. I  repeat the next command a few times, using different VALUES:


mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");  

**i inserted 4 rows** 


![prj41](https://user-images.githubusercontent.com/110178748/182433874-c216e1ee-e0f2-4f03-8ba6-51585fe97436.png)


To confirm that the data was successfully saved to my  table, I  run:
mysql>  SELECT * FROM example_database.todo_list;


![prj42](https://user-images.githubusercontent.com/110178748/182434085-5d2f9921-9601-4362-953e-9c5ec446d47f.png)



Now I can create a PHP script that will connect to MySQL and query for my content. 
I Create a new PHP file in my custom web root directory using a preferred editor. I will  use **vi**  for that:

nano /var/www/projectLEMP/todo_list.php



I will then Copy this content into my todo_list.php script:

<?php
$user = "example_user";
$password = "PassWord.1";
$database = "example_database";
$table = "todo_list";

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



![prj44](https://user-images.githubusercontent.com/110178748/182435282-949d0c8e-44da-4418-a786-cfe8a8c55178.png)


I can now access this page in my Chrome web browser by visiting the domain name or 
public IP address configured for my website, followed by /todo_list.php:

http://<Public_domain_or_IP>/todo_list.php
in my case
http://54.226.102.116/todo_list.php



![prj46](https://user-images.githubusercontent.com/110178748/182435598-151505e3-f84e-4bdc-ba9a-db2d3aed5d38.png)



My  PHP environment is ready to connect and interact with my MySQL server.

END PROJECT 2 



























































