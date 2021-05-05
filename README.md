# A High-Performance Wordpress Installation

### Why

Performance is already important for modern Wordpress sites as users don't want to wait for loading. 

However, Google Core Web Vitals update this year will make performance more important page ranking will be impacted significantly. This is measured using tools such as Pagespeed (show image).

### What Can I do about this for my Wordpress site?


1. Get faster hosting.
2. Get a caching plugin.
3. Get away from Wordpress page builders, and use Gutenberg with a light theme.


### But I already have Siteground, Kinsta or WP Engine

Using Siteground, WP Engine or Kinsta are all great options for the average user.

However with the solution I am going to describe, you can improve your Pagespeed, in my tests, by 10 points (do a Kinsta comparison in next video)

### and Monthly Cost ?

For this example I will use Google Cloud + Cloudflare. You can get $350 credits if you sign up using the link below, $50 more than usual.

[Kinsta](https://kinsta.com/plans/?plan=visits-pro&interval=month) is $30 for 1 WP install, 25k visits, 10gb disk, SSL + CDN

This solution is [free (VM), $7.67 (Mysql)] for many WP installs, unlimited visits, 10gb disk, SSL +CDN  with fast Litespeed caching

## Let's get started.


---

## 1: Faster Hosting

## Setup a VM Instance

We will use the (**f1-micro**) which costs nothing... for ever! . You can upgrade this to a better CPU if needed later.

- choose location
- open ports: HTTP, HTTPS
- Boot disk: Ubuntu 20.04 LTS
- Networking apply tag: `wordpressvm` ,
- use a Bootstrap script (supplied below) to install OpenLitespeed, Wordpress, PHP

```jsx
#!/bin/bash
apt update -y
apt install firewalld -y
firewall-cmd --zone=public --permanent --add-port=80/tcp
firewall-cmd --zone=public --permanent --add-port=443/tcp
firewall-cmd --zone=public --permanent --add-port=7080/tcp
firewall-cmd --reload
wget -O - http://rpms.litespeedtech.com/debian/enable_lst_debian_repo.sh | sudo bash
apt-get install lsphp74 -y
apt install lsphp74-common lsphp74-curl lsphp74-imap lsphp74-json lsphp74-mysql lsphp74-opcache lsphp74-imagick lsphp74-memcached lsphp74-redis -y
apt-get install openlitespeed -y
/usr/local/lsws/bin/lswsctrl start
cd /tmp
git clone https://github.com/andygrillo/litespeed-wordpress-vm.git
cp litespeed-wordpress-vm**/**httpd_config.conf /usr/local/lsws/conf/
cp litespeed-wordpress-vm/vhconf.conf /usr/local/lsws/conf/vhosts/Example/
/usr/local/lsws/bin/lswsctrl restart
apt install redis -y
systemctl start redis-server
systemctl enable redis-server
cd /usr/local/lsws/Example/html/
wget https://wordpress.org/latest.tar.gz
tar xvfz latest.tar.gz
chown -R nobody:nogroup /usr/local/lsws/Example/html/wordpress
find /usr/local/lsws/Example/html/wordpress/ -type d -exec chmod 750 {} \;
find /usr/local/lsws/Example/html/wordpress/ -type f -exec chmod 640 {} \;
chown -R nobody:nogroup /usr/local/lsws/Example/html/wordpress
```

- Networking>VPC Networks>Firewall
- add 7080 firewall rule with IP range: 0.0.0.0/0
- add 3306, 33060 firewall rule with IP range: 10.0.0.0/0

- SSH connect to create password for OpenLitespeed dashboard:

```bash
sudo /usr/local/lsws/admin/misc/admpass.sh
```

- go to dashboard:

```bash
http://102.021.03.2:7080
```

- Add ip address or domain  and do Soft Restart

```jsx
>Listeners>Default>Virtual Host Mappings>Domains
```

- Take down the Primary Internal IP `10.128.0.2`

## Setup a separate MySQL database

We will then use a separate managed MySQL database (**db.t2.micro**)

- Machine type: Shared core
- Same location as VM
- Add root password
- Storage: 10GB
- Connections: Private IP, `default` automatic IP range.
- Connections: no Public
- Connect to add IP of VM: `10.128.0.2`
- Click on instance to get IP of database
- Create database `wordpress`
