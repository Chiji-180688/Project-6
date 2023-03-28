# Documenting Project 6
Step 1: Preparing a web server
1. launch an ec2 instance that will serve as web server
2. create 3 volumes in same az as the instance
3. attach the volumes one after the other to the web server instance
4. in the terminal; 
	`lsblk`
    ![lsblk output](./Images%206/web-server%20lsblk%20output.PNG)
    `sudo gdisk /dev/xvdf`
    `sudo gdisk /dev/xvdg`
    `sudo gdisk /dev/xvdh`
    ![creating disk partitions](./Images%206/creating%20disk%20partitions.PNG)
5. installing lvm
    `sudo yum install lvm2`
    ![lvm installed](./Images%206/lvm2%20installed.PNG)
    `sudo lvmdiskscan`
     ![checking avail partitions](./Images%206/avilable%20partitions.PNG)
6. creating physical volumes
     `sudo pvcreate /dev/xvdf1`
     `sudo pvcreate /dev/xvdg1`
     `sudo pvcreate /dev/xvdh1`
     ![pvcreate xvdf1,xvdg1,xvdh1](./Images%206/physical%20volume%20creation.PNG)
     `sudo pvs`
     ![sudo pvs output](./Images%206/physical%20volume%20succesfully%20created.PNG)
7. creating volume group
     `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
     ` sudo vgs`
     ![volume group](./Images%206/volume%20group%20successfully%20created.PNG)
     `sudo lvcreate -n apps-lv -L 14G webdata-vg`
     `sudo lvcreate -n logs-lv -L 14G webdata-vg`
     ` sudo lvs`
8. creating logical group
     ![logical vol](./Images%206/logical%20volume%20successfully%20created.PNG)
     `sudo vgdisplay -v #view complete setup - VG, PV, and LV`
     `sudo lsblk`
     ![sudo lsbk](./Images%206/lsblk3%20output.PNG)
9. creating ext4 files
     `sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
     `sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`
10. creating /var/www/html directory
     `sudo mkdir -p /var/www/html`
11. mount directory on logical volume
     `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`
     `sudo rsync -av /var/log/. /home/recovery/logs/
     ![logs file backed up with rsync](./Images%206/logs%20file%20backed%20up%20with%20rsync.PNG)`
     `sudo mount /dev/webdata-vg/logs-lv /var/log`
     `sudo rsync -av /home/recovery/logs/. /var/log`
     ![var-log restored](./Images%206/var-log%20files%20restored.PNG)
     `sudo blkid`
12. updating /etc/fstab
     `sudo vi /etc/fstab`
     ![fstab update](./Images%206/fstab%20update.PNG)
     ` sudo mount -a`
     `sudo systemctl daemon-reload`
     `df -h`
Step 2: Preparing Database
1.  repeated nos 1 - 7 in step 1, but created db-lv instead of apps-lv
2.  mount directory on logical volume
     `sudo mount /dev/vg-database/db-lv /db`
Step 3: Installing wordpress on web server ec2
1.  installing php
     `sudo yum -y update`
     `sudo dnf upgrade --refresh`
     ![dnf update](./Images%206/dnf%20update.PNG)
     `sudo dnf config-manager --set-enabled crb`
     `sudo dnf install \https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm \ https://dl.fedoraproject.org/pub/epel/epel-next-release-latest-9.noarch.rpm`
     `sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y`
     ![importing centos9 remi repo](./Images%206/importing%20centos9%20remi%20repo.PNG)
     `dnf module list php`
     ![epel for centos9](./Images%206/installing%20versions%20of%20epel%20for%20centos9.PNG)
     `sudo dnf module enable php:remi-8.2 -y`
     ![enabling php remi repo](./Images%206/enabling%20php%20remi%20repo.PNG)
     `sudo dnf install php php-cli -y`
     ![installing php](./Images%206/installing%20php.PNG)
     `php -v`
     ![php -v output](./Images%206/php%20-v%20output.PNG)
     `sudo yum install php php-opcache php-gd php-curl php-mysqlnd`
     ![frequently used extensions](./Images%206/frequently%20used%20extentions%20installed.PNG)
     `sudo systemctl start php-fpm`
     `sudo systemctl enable php-fpm`
     ![systemctl enebled php](./Images%206/systemctl%20enabled%20php-fpm.PNG)
     `setsebool -P httpd_execmem 1`
2.  installing apache
     `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`
     ![installing apache](./Images%206/installing%20wget%20Apache%20and%20dependencies.PNG)
     `sudo systemctl enable httpd`
     ![apache enabled](./Images%206/apache%20enabled.PNG)
     `sudo systemctl start httpd`
     ![apache started](./Images%206/Apache%20started.PNG)
3.  Downloading wordpress and copying wordpress to var/www/html
     `mkdir wordpress`
     `cd wordpress`
     `sudo wget http://wordpress.org/latest.tar.gz`
     ![downloading wordpress](./Images%206/downloading%20wordpress.PNG)
     `sudo tar xzvf latest.tar.gz`
     ![wordpress extracted](./Images%206/wordpress%20extracted.PNG)
     `sudo rm -rf latest.tar.gz`
     `cp wp-config-sample.php wp-config.php`
     `cp -R wordpress /var/www/html/`
     ![wordpress copied into html file](./Images%206/wordpress%20copied%20into%20html%20file.PNG)
4.  configuring selinux policies
     `sudo chown -R apache:apache /var/www/html/`
     `sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R`
     `sudo setsebool -P httpd_can_network_connect=1`
Step 4: Installing mysql server on database ec2
     `sudo yum update`
     `sudo yum install mysql-server`
     ![mysql in db](./Images%206/mysql%20in%20database.PNG)
     `sudo systemctl restart mysqld`
     `sudo systemctl enable mysqld`
Step 5: Configuring DB to work with WordPress
1.  writing security script
     `sudo mysql_secure_installation`
     `sudo mysql`
     `CREATE DATABASE wordpress;`
     `CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`
     `GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%' WITH GRANT OPTIONS;`
     `FLUSH PRIVILEGES;`
     `show databases;`
     ![database created](./Images%206/user%2C%20host%20successfully%20created%20in%20database.PNG)
Step 6: Configuring WordPress to connect to remote database
1.  Installing MySQL client on web server and connecting to DB server
    Repeated steps 4 and 5 above in web server.
    `show databases;`
    ![database created in web server](./Images%206/SECURITY%20SCRIPT%20IN%20WEB-SERVER.PNG)
    Opened mysql port 3306 on database ec2 and allowed access only from web server (3.235.89.88/32)
    `sudo mysql -u wordpress -p -h 172.31.10.199`
    ![testing to confirm that client can talk to host](./Images%206/to%20ensure%20client%20can%20talk%20to%20host.PNG)
    Enabled TCP port 80 in Inbound Rules configuration for Web Server EC2 (enabled from everywhere 0.0.0.0/0)
    ![connecting to webpress through browser](./Images%206/trying%20to%20connect%20to%20wordpress.PNG)













