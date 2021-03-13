# A High-Performance Wordpress Installation
This guide will demonstrate how to setup a very fast Wordpress installation on a server with [OpenLiteSpeed](https://openlitespeed.org/), which typically performs better than Apache or NGinx. 

The OS used is Ubuntu 20.04.

## Server Requirements
This can be installed on a VM with most cloud providers (AWS, Azure, IBM, etc). This guide was tested on a VM in Oracle Cloud.

You should specify the CPU and RAM to match your expected load, but these are settings for a small/medium VM that have worked for me:
- 2 CPUs
- 8 GB RAM
- Fixed IP address


Please open ports `80` and `443` for HTTP and HTTPS, and also `7080` for the OpenLiteSpeed dashboard.

### Database
For this guide I used an external managed MySQL database on the same virtual network, although you are welcome to host a database on the same VM as Wordpress. By hosting the database separately you will typically have better performance as resources are not shared, and also a backup if the VM fails. If however you wish to install a local database you can with `sudo apt install mariadb-server`.

Assuming you are using an external managed database, be sure that it is visible to the VM. This is done by hosting on the same subnet, eg. if your VM has an IP of 10.0.0.1, the database could be on 10.0.0.2. 

You will also need to open ports internally on the subnet for `3306` and `33060`, but restricting access to instances on the internal subnet only.

Be sure to remember the username and password!

## Setup the VM
Once provisioned, SSH into the VM using keys or password to get started.

### Update your system
```sudo apt-get update```

```sudo apt-get upgrade -y```

### Open your firewall (optional)
If required, as with Oracle Cloud, open the necessary ports in the firewall:

```sudo apt install firewalld```

```sudo firewall-cmd --zone=public --permanent --add-port=80/tcp```

```sudo firewall-cmd --zone=public --permanent --add-port=443/tcp```

```sudo firewall-cmd --zone=public --permanent --add-port=7080/tcp```

Then reload the firewall

```sudo firewall-cmd --reload ```

### Create a database
Create the MySQL client on the VM:

```cd /tmp```

```curl -OL https://dev.mysql.com/get/mysql-apt-config_0.8.15-1_all.deb```

and add to your system's repository list and begin installation:

```sudo dpkg -i mysql-apt-config*```

During installation you will be be asked which product to configure. Just use the default by selecting `OK`. Then run `sudo apt update
` to refresh your apt cache.

Now install MySQL client and you're set:

```sudo apt install mysql-shell```

Login to the external datatbase using the correct IP address and user:

```$ mysqlsh --sql root@10.0.0.2:3306```

Once in the MySQL client:

```mysql-js> CREATE DATABASE wordpress;```

```mysql-js> CREATE USER wp IDENTIFIED BY 'MyCrazyP3assword!';```

```mysql-js> GRANT ALL PRIVILEGES ON wordpress.* TO wp;```

### Install PHP 7.4

You will need PHP 7.4 or higher:

```sudo apt-get install lsphp74```

```sudo apt install lsphp74-common lsphp74-curl lsphp74-imap lsphp74-json lsphp74-mysql lsphp74-opcache lsphp74-imagick lsphp74-memcached lsphp74-redis```

### Install OpenLiteSpeed

```wget -O - http://rpms.litespeedtech.com/debian/enable_lst_debian_repo.sh | sudo bash```

```sudo apt-get install openlitespeed```

### Setup Wordpress

Navigate to the virtual host:

```cd /usr/local/lsws/Example/html/```

Get latest Wordpress

```wget https://wordpress.org/latest.tar.gz```

Extract:

```tar xvfz latest.tar.gz```

You now have a directory called `wordpress` inside `/usr/local/lsws/Example/html`

Setup the files ownership and permissions:

```sudo chown -R nobody:nogroup /usr/local/lsws/Example/html/wordpress```

```sudo find /usr/local/lsws/Example/html/wordpress/ -type d -exec chmod 750 {} \;```

```sudo find /usr/local/lsws/Example/html/wordpress/ -type f -exec chmod 640 {} \;```

Change the wp-config.php file to point wordpress to our external datatabse.

```sudo nano /usr/local/lsws/Example/html/wordpress/wp-config.php```

Where you find `** MySQL settings...` copy in the following using your own values:
```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wp' );

/** MySQL database password */
define( 'DB_PASSWORD', 'MyRootPassword' );

/** MySQL hostname */
define( 'DB_HOST', '10.0.0.2' );
```
Below this you should paste in unique Salt Keys, which can be generated [at this site](https://api.wordpress.org/secret-key/1.1/salt/)

Finally you are ready to install Wordpress :)






