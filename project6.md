# WEB SOLUTION WITH WORDPRESS

In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

Project 6 consists of two parts:

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.

2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

As a DevOps engineer, your deep understanding of core components of web solutions and ability to troubleshoot them will play essential role in your further progress and development.

## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”
### Step 1 — Prepare a Web Server
 - Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

### Added EBS Volumes
![Screenshot (86)](https://user-images.githubusercontent.com/52970510/164915233-355b0ff6-744b-4066-bde2-4a6b15853ffc.png)

- Attach all three volumes one by one to your Web Server EC2 instance

### Attaching EBS Volumes to my instance
![Screenshot (88)](https://user-images.githubusercontent.com/52970510/164915388-607719ae-98ff-4433-99d0-d8efcd8d7cfe.png)

- Open up the Linux terminal to begin configuration
- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.
### block devices attached to the server
![Screenshot (89)](https://user-images.githubusercontent.com/52970510/164915761-47fd15b9-b5aa-4baf-a9f0-4499570687a9.png)

- Use df -h command to see all mounts and free space on your server

- Use gdisk utility to create a single partition on each of the 3 disks

sudo gdisk /dev/xvdf
### partition creation for xvdf
![Screenshot (91)](https://user-images.githubusercontent.com/52970510/164915888-c04f3414-e9ae-4db8-a0b9-766f54651b91.png)

sudo gdisk /dev/xvdg
### partition creation for xvdg
![Screenshot (92)](https://user-images.githubusercontent.com/52970510/164915914-a1777b02-2b69-42a8-a23a-a2468d446c04.png)

sudo gdisk /dev/xvdh
### partition creation for xvdh
![Screenshot (93)](https://user-images.githubusercontent.com/52970510/164915965-d3462b27-332a-4e0c-9a30-08ba09b66083.png)

- Use lsblk utility to view the newly configured partition on each of the 3 disks.
![Screenshot (94)](https://user-images.githubusercontent.com/52970510/164916062-696a8301-acb1-4c33-aaa8-e963988abd94.png)

- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```javascript
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

- Verify that your Physical volume has been created successfully by running sudo pvs
![Screenshot (95)](https://user-images.githubusercontent.com/52970510/164916163-b17bb11c-1494-4f4d-86e8-359980f2b242.png)

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```javascript
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```

- Verify that your VG has been created successfully by running sudo vgs
![Screenshot (96)](https://user-images.githubusercontent.com/52970510/164916230-ee43db7b-7d88-4961-9b6d-07485bc40cf6.png)

- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
```javascript
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
- Verify that your Logical Volume has been created successfully by running sudo lvs
![Screenshot (97)](https://user-images.githubusercontent.com/52970510/164916297-b15befc0-a8e2-4551-8e43-ea33325291d9.png)

- Verify the entire setup
```javascript
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
![Screenshot (98)](https://user-images.githubusercontent.com/52970510/164916368-91d19a4d-81db-4a83-885b-9c2a7a0ce7d2.png)

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem
```javascript
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
- Create /var/www/html directory to store website files
```javascript
sudo mkdir -p /var/www/html
```
- Create /home/recovery/logs to store backup of log data
```javascript
sudo mkdir -p /home/recovery/logs
```
- Mount /var/www/html on apps-lv logical volume
```javascript
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
```javascript
sudo rsync -av /var/log/. /home/recovery/logs/
```
- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)
```javascript
sudo mount /dev/webdata-vg/logs-lv /var/log
```
- Restore log files back into /var/log directory
```javascript
sudo rsync -av /home/recovery/logs/. /var/log
```
- Update /etc/fstab file so that the mount configuration will persist after restart of the server.

## UPDATE THE `/ETC/FSTAB` FILE
The UUID of the device will be used to update the /etc/fstab file;

sudo blkid

sudo vi /etc/fstab

- Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
![Screenshot (99)](https://user-images.githubusercontent.com/52970510/164916868-cb507348-67f5-40a3-85d2-7c01c09aafd6.png)

- Test the configuration and reload the daemon
```javascript
sudo mount -a
sudo systemctl daemon-reload
```
- Verify your setup by running df -h, output must look like this:
![Screenshot (100)](https://user-images.githubusercontent.com/52970510/164916933-1cd5b8f2-b227-4ebb-8594-4bfad232e2f5.png)

### Step 2 — Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

### Step 3 — Install WordPress on your Web Server EC2
- Update the repository
```javascript
sudo yum -y update
```
- Install wget, Apache and it’s dependencies
```javascript
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

- Start Apache
```javascript
sudo systemctl enable httpd
sudo systemctl start httpd
```

- To install PHP and it’s depemdencies
```javascript
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

- Restart Apache
```javascript
sudo systemctl restart httpd
```

- Download wordpress and copy wordpress to var/www/html
```javascript
 mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
 ```

- Configure SELinux Policies
```javascript
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```

### Step 4 — Install MySQL on your DB Server EC2
```javascript
sudo yum update
sudo yum install mysql-server
```
Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
```javascript
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
### Step 5 — Configure DB to work with WordPress
```javascript
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
### Step 6 — Configure WordPress to connect to remote database.
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

### mysql port 3306 open on DB server (EC2 INSTANCE)
![Screenshot (101)](https://user-images.githubusercontent.com/52970510/164917518-db2765ed-f540-4bf0-a7ac-d461f7064b04.png)

- Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
```javascript
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
- Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

- Change permissions and configuration so Apache could use WordPress:

- Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

- Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/


  ![Screenshot (102)](https://user-images.githubusercontent.com/52970510/164924987-347dcb1b-f837-48fa-b7b3-d21962e784bd.png)

  ![Screenshot (103)](https://user-images.githubusercontent.com/52970510/164925439-90c88c07-a9b5-4ff4-a55a-c4b3c163dd3f.png)

  ![Screenshot (104)](https://user-images.githubusercontent.com/52970510/164926083-48472e84-5afd-45b0-bef4-01564810f3ec.png)

  
  
