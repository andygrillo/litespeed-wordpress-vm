# A High-Performance Wordpress Installation
This guide will demonstrate how to setup a very fast Wordpress installation on a server with [OpenLiteSpeed](https://openlitespeed.org/), which typically performs better than Apache or NGinx. 

The OS used is Ubuntu 20.04.

These steps allowed me to achieve a 100 score in Google PageSpeed and also GTMetrix whilst using a Gutenberg theme (Kadence).

## Server Requirements
This can be installed on a VM with most cloud providers (AWS, Azure, IBM, etc). This guide was tested on a VM in Oracle Cloud.

You should specify the CPU and RAM to match your expected load, but these are settings for a small/medium VM that have worked for me:
- 2 CPUs
- 8 GB RAM
- 15 GB Drive

Make sure you have a fixed IP address.

Open ports `80` and `443` for HTTP and HTTPS, and also `7080` for the OpenLiteSpeed dashboard.

### Database
For this guide I used an external managed MySQL database on the same virtual network, although you are welcome to host a database on the same VM as Wordpress. By hosting the database separately you will typically have better performance as resources are not shared, and also a backup if the VM fails. If however you wish to install a local database you can with `sudo apt install mariadb-server`.

Assuming you are using an external managed database, be sure that it is visible to the VM. This is done by hosting on the same subnet, eg. if your VM has an IP of 10.0.0.1, the database could be on 10.0.0.2. 

You will also need to open ports internally on the subnet for `3306` and `33060` so that the database is not reachable by external devices on the internet.

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

## Install and Configure OpenLiteSpeed
Next, you need to configure the OpenLiteSpeed server to host your WordPress site. It requires you to set the right version of PHP processor, enable the rewrite module and several other features.

### Install OpenLiteSpeed

```wget -O - http://rpms.litespeedtech.com/debian/enable_lst_debian_repo.sh | sudo bash```

```sudo apt-get install openlitespeed```

### Configure OpenLiteSpeed in the Dashboard
Go to `Server Configuration > External App` and click on the edit button:
![image](https://user-images.githubusercontent.com/6279965/111042627-e6fc6100-8403-11eb-94fc-72c6de6127e4.png)

Here, you can configure your server to use any specific PHP processor. For this tutorial, we will use lsphp74.

- Replace `lsphp` with `lsphp74`
- Replace `uds://tmp/lshttpd/lsphp.sock` with `uds://tmp/lshttpd/lsphp74.sock`
- Replace `lsphp73/bin/lsphp` with `$SERVER_ROOT/lsphp74/bin/lsphp`

![image](https://user-images.githubusercontent.com/6279965/111042645-ff6c7b80-8403-11eb-8a23-62e07e04a7b1.png)

Once changes are made, click the save icon in the upper right corner as shown above. 

Moving next to configure the rewrite module which is an essential requirement for WordPress feature. Go to the `Virtual Hosts` and click on the view icon.

![image](https://user-images.githubusercontent.com/6279965/111042666-1f03a400-8404-11eb-976d-26c341926d97.png)

Click on the `General` tab and edit the `General options` with the edit icon at the top right corner.

![image](https://user-images.githubusercontent.com/6279965/111042670-2460ee80-8404-11eb-806d-a8ed6233ae3a.png)

In the `Document Root` field, type `$VH_ROOT/html/wordpress` and click the save button at the top right corner.

![image](https://user-images.githubusercontent.com/6279965/111042717-56725080-8404-11eb-8042-3170c0fa0f4b.png)

Then again on the `General` tab of `Virtual Hosts` configuration, click the edit icon next to `Index Files` section.

![image](https://user-images.githubusercontent.com/6279965/111042727-638f3f80-8404-11eb-808b-83398cb9ff05.png)

In the `Index Files` field, add `index.php` at the beginning of the section. Then click the save button at the top right corner.

![image](https://user-images.githubusercontent.com/6279965/111042739-7275f200-8404-11eb-8b15-58a027781c0e.png)

Next, go to the `Rewrite` tab of the `Virtual Hosts` configuration view and edit the `Rewrite Control` options.

![image](https://user-images.githubusercontent.com/6279965/111042748-81f53b00-8404-11eb-8db6-06d7c1d3b75c.png)

Set `Enable Rewrite` and `Auto Load` from `.htaccess` to Yes and click save icon at the top right corner.

![image](https://user-images.githubusercontent.com/6279965/111042761-90435700-8404-11eb-8987-7d8436ea34a4.png)

Once you’ve configured the OpenLiteSpeed server, Click the gracefully restart icon to apply the changes.

![image](https://user-images.githubusercontent.com/6279965/111042770-99342880-8404-11eb-8182-065d14284e29.png)


## Install Wordpress

Navigate to the virtual host:

```cd /usr/local/lsws/Example/html/```

Get latest Wordpress

```wget https://wordpress.org/latest.tar.gz```

Extract:

```tar xvfz latest.tar.gz```

```sudo chown -R nobody:nogroup /usr/local/lsws/Example/html/wordpress```

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
Below this you should paste in unique Salt Keys, which can be generated [at this site](https://api.wordpress.org/secret-key/1.1/salt/).

Save and close the editor

## Configure Wordpress
If you navigate to the VM's IP address, you should now be presented with the Wordpress welcome screen. Create your user and enter.

### Configure LiteSpeed plugin
To ensure you get the best performance install the `LiteSpeed Cache` plugin. These settings work for me:

#### Cache Section
In `General` Tab:
![image](https://user-images.githubusercontent.com/6279965/111042969-84a46000-8405-11eb-8d31-a84973515fb9.png)

In `Cache` turn everything to `ON` except `Cache Mobile`.

In `Object` turn on `Object Cache` and choose Redis or Memcached. To install Redis you can follow this [guide](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04). You should also turn on `Persistent Connection` and `Cache Wp-Admin`.

In `Browser` turn on the `Browser Cache`

#### CDN Section
Leave everything off but turn on Cloudflare API (if using Cloudflare). Here add your credentials and ensure the IP address has been added to the Cloudflare DNS. 

#### Image Optimization Section
In the settings I leave everything off apart from `Image WebP Replacement`. I then use the `Imagify` Plugin to do compression and webp generation. In Imagify I select `Use <picture> tags` but do not specify a URL.

#### Page Optimization Section
- CSS: Leave everything on apart from `Load CSS Asynchronously`
- JS: Leave everything on and choose `Default` Load Inline JS 
- Optimization: Leave everything on apart from `Remove Query Strings`, `Remove Google Fonts` and `Remove Noscript Tag`



### Select Theme
With the advent of Wordpress 5.7 it is now recommended to use Gutenberg, the built-in page builder. This has significant advantages over 3rd party builders such as Elimentor or WPBakery.

I found the Kadence site to be very fast and also free !


## Credits
Configuration of the OpenLiteSpeed server was based on this [guide on Upcloud.com](https://upcloud.com/community/tutorials/install-wordpress-openlitespeed/).




