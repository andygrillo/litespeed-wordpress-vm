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

## Setup the VM
Once provisioned, SSH into the VM using keys or password to get started.

#### Update your system
```sudo apt-get update```

```sudo apt-get upgrade -y```

#### Open your firewall (optional)
If required, as with Oracle Cloud, open the necessary ports in the firewall:

```sudo apt install firewalld```

```sudo firewall-cmd --zone=public --permanent --add-port=80/tcp```

```sudo firewall-cmd --zone=public --permanent --add-port=443/tcp```

```sudo firewall-cmd --zone=public --permanent --add-port=7080/tcp```

Then reload the firewall

```sudo firewall-cmd --reload ```

#### Install OpenLiteSpeed

```wget -O - http://rpms.litespeedtech.com/debian/enable_lst_debian_repo.sh | sudo bash```

```sudo apt-get install openlitespeed```











