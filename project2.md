
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
  
  































































