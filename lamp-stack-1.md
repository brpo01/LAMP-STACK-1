# **WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS**

This Project covers the implementation of developing a LAMP Stack app from start to finish and deploying to **AWS** (Amazon Web Services). The LAMP Stack is a Technology Stack. A Technology stack comprises of framework/tools that are used in building a software application. The LAMP Acronym stands for Linux, Apache, MySQL, PHP.

- Linux - It is an operating system that is free and open source, and was built from UNIX. We would be creating our AWS EC2 instance on a virtual Server.

- Apache - Apache is a very popular web server software that is used for hosting applications

- MySQL - It is a Database management system that is used for storing data in form of tables(rows, columns). It is a relational database and data must be stored in a structured format.

- PHP - PHP is a modern server side languauge that is used for building web applications.

Let's get started with implementing the LAMP Stack and deploying to AWS.

## **Create an EC2 instance  on your AWS Dashboard**
You have to create a Linux virtual server on AWS. There are several features and services that AWS offer but for this project, I'll be using the ec2 instance. If you're using windows, you have to download a CLI (Command Line Interface) such as Git, Putty, Termius, MobaXterm that is Linux-enabled so you can run basic linux commands. I watched this [video](https://www.youtube.com/watch?v=xxKuB9kJoYM&list=PLtPuNR8I4TvkwU7Zu0l0G_uwtSUXLckvh&index=7) to create a Linux virtual server, also i need to add that the Linux distro we're using is Ubuntu.

When we're done creating the server, we first have to add permissions to our private key using this command.

![1](https://user-images.githubusercontent.com/47898882/125612456-909a6286-6a8b-4492-b0c9-d8e804fa153b.JPG)

If that works out successfully, you can then connect to the server using the ssh command.

![2](https://user-images.githubusercontent.com/47898882/125612466-cfe1af2c-41a1-4ccf-aac0-6e2a88404b62.JPG)

Now that our Virtual Server is up and running we can start building our application.

## **Install Apache Web Server**
To install the apache web server, we have to use the *apt* package manager and thankfully it comes installed with linux.

Use this commands below to update your apt package manager and install apache

```
$ sudo apt upgrade
$ sudo apt install apache2
```

When that is done installing you can check the status of apache to verify if it is running as a service using:

```
$ sudo systemctl status apache2
```
When that is run, you need to get an output like this:

![3](https://user-images.githubusercontent.com/47898882/125614939-cbd30b58-9515-447e-b9d2-af6c90c38154.JPG)

In order to be able to test our apache server on the browser, we have to allow inbound connections into port 80. Port 80 handles http connections and allows you to connect to the internet over a browser. To achieve this, we have to go into our AWS console and add that configuration

![4](https://user-images.githubusercontent.com/47898882/125615842-422b1b42-fb47-41f8-891b-496470a750d4.JPG)

After that has been configured, we can then test on our browser using our public IP address or DNS name:

`
http://<Public-IP-Address>:80 or
http://DNS-name:80
`

You can check for your IP address on your aws console or better still type out this command on your terminal:

```
$ curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```
If it displays the Apache Ubuntu Default Page. You're good to go!

## **Install MySQL**
Install SQL using the *apt* command:

```
$ sudo apt install mysql-server
```
When that is done installing, you have run a security script to remove default settings and secure the server

```
$ sudo mysql_secure_installation
```
When you input this command it'll give you a prompt to create a password and after that is done you can test that mysql works fine using `sudo mysql`

![5](https://user-images.githubusercontent.com/47898882/125618012-66fb82c3-d072-4851-bb07-b23311d212fe.JPG)

## **Install PHP**
You have to install the php package on your server using the *apt* command. There are also some other php packages you'll need to install such as the `php-mysql` package which allows php to communicate with MySQL and `libapache2-mod-php` that enables apache to handle php files.

```
$ sudo apt install php php-mysql libapache2-mod-php
```

If that installs correctly you can check the php version you installed using `php -v`

At this junction, we have installed all the tools in LAMP Stack and they're all fully functional.

## **Create A Virtual Host in Apache**
In order to test our setup with a php script, we have to create a virtual host.

Create a folder in your in the /var/www/html directory using `mkdir`. This directory is the default that has been configured for serving our files.

```
$ sudo mkdir lampserver
```

Assign ownership to the user using `chown` command.
```
$ sudo chown $user:$user lampserver
```

When that is done we have to create our own configuration script in the apache configuration folder
```
$ sudo vi /etc/apache2/site-avialable/lampserver.conf
```

*vi* also known as vim is a text editor in linux for writing code. The configuration for the virtual host will be written as below. You can use *i* to insert, :*wq* to write into the file and quit.

```
<VirtualHost *:80>
    ServerName lampserver
    ServerAlias www.lampserver 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/lampserver
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
When you have saved your configuration file use `sudo a2ensite lampserver.conf` to enble to virtual host you just created. Don't forget to also disable the default configuration using `sudo dissite 000-default`

**Note** - If your configuration in lampserver.conf is not accurate, it'll throw an error when you're testing it on your browser. Make sure the DocumentRoot is correct most especially and is linking to the `/var/www/html/lampserver` folder we created. You can also test the accuracy of your configuration using `sudo apachectl configtest`

![6](https://user-images.githubusercontent.com/47898882/125623621-3e492731-aa1c-444f-9b85-d228fe26c3bd.JPG)

If your config works perfectly go back into your `/var/www/html/lampserver` folder and create a file called index.html using the *touch* command.

```
$ cd /var/www/html/lampserver
$ touch index.html 
```
Then type out the following code below:

```
$ sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```
What this piece of code will do is redirect the output of that command into the index.html. You can confirm using `cat index.html` to check the file. You can now test this on your browser using:

`
http://<Public-IP-Address>:80 or
http://DNS-name:80
`.
You should see something like this on your browser:

![7](https://user-images.githubusercontent.com/47898882/125625721-eacb7530-2e9c-41ed-8827-7ab264a54e0a.JPG)

We can also test our virtual host by writing a simple php script. In that same /var/www/html/lampserver directory, create another file called index.php and use *vi* to enter into the file. When that is done, the following script should be typed below:

```
<?php
phpinfo();
```
Before we can test, we have to change the order of preference of our files. Remember we displayed the index.html file on the the browser, but before we can display any other file format, we have to change the preference as follows:

```
$ sudo vi /etc/apache2/mods-enabled/dir.conf
```
When that is done you can change the preference.

![8](https://user-images.githubusercontent.com/47898882/125627452-87bfb1cd-b1dd-47a0-b220-474fed56342b.JPG)

After all of this is done, we can test in our browser using 
`
http://<Public-IP-Address>:80 or
http://DNS-name:80
`.

You should see an output like this:
![9](https://user-images.githubusercontent.com/47898882/125628082-c3a68cae-5086-4da6-ba60-69d315bedfd0.JPG)

So we have successfully built a LAMP stack application and deployed to our AWS ec2 instance.

**This is complete Web Stack Implemenatation for the LAMP Stack**





 




 


