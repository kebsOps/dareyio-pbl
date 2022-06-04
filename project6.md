**Create a Web Solution Using WordPress**




Launch a RedHat EC2 instance that will have a role - "Web Server"


<img width="1289" alt="Web Server" src="https://user-images.githubusercontent.com/10085348/172007295-968cfbfa-2e73-470a-aac7-c98b043f12aa.png">



Create 3 volumes in the same AZ (us-east-2a) as your Web Server EC2, each of 10 GiB and attach all three volumes one by one to your Web Server EC2 instance

<img width="1278" alt="Storage Volume" src="https://user-images.githubusercontent.com/10085348/172007534-f086f8e1-8beb-4e6c-91eb-8a594ef52d3b.png">



SSH to Web Server from your terminal app and use 'lsblk' command inspect block devices attached to Web server

<img width="391" alt="Inspect block devices attached to the server" src="https://user-images.githubusercontent.com/10085348/172007766-e45cbfa0-ea7c-4c8f-8000-b067a33a5b66.png">



Use gdisk utility to create a single partition on each of the 3 disks (xvdf, xvdg, xvdh)

<img width="945" alt="Use gdisk utility to create a single partition on each of the 3 disks" src="https://user-images.githubusercontent.com/10085348/172007919-b914d31b-1451-4cac-8410-2af257956d04.png">


Use lsblk utility to view the newly configured partition on each of the 3 disks (i.e xvdf1, xvdg1, xvdh1)

<img width="378" alt="Use lsblk utility to view the newly configured partition on each of the 3 disks" src="https://user-images.githubusercontent.com/10085348/172007999-50933c51-7f6c-4d29-8989-6cf7bbfaa537.png">



Install lvm2 package

<img width="1188" alt="Install lvm2 package" src="https://user-images.githubusercontent.com/10085348/172008137-858df802-5e97-448e-b50e-006262d2d915.png">



Verify lvm2 installation

<img width="363" alt="lvm installation location" src="https://user-images.githubusercontent.com/10085348/172008573-08836a9d-575e-4153-826e-967f0750e561.png">



Check for available partitions 

<img width="441" alt="Check for available partitions" src="https://user-images.githubusercontent.com/10085348/172008501-a4fa539b-a7fc-4b7c-9af5-cd8ea79b899a.png">



Use pvcreate utility to mark each of 3 disks as physical volumes and Use vgcreate utility to add all 3 PVs to a volume group (VG) Name the VG webdata-vg

<img width="727" alt="Use pvcreate utility to mark each of 3 disks as physical volumes and Use vgcreate utility to add all 3 PVs to a volume group (VG) Name the VG webdata-vg" src="https://user-images.githubusercontent.com/10085348/172008610-d6b829f4-7a6d-43ae-b4a5-1552555c9cbd.png">



Create two logical volumes apps-lv and logs-lv and verify with sudo lvs


<img width="699" alt="Create two logical volumes apps-lv and logs-lv and verify with sudo lvs" src="https://user-images.githubusercontent.com/10085348/172008750-818a7fb9-4cb7-4bcf-a3bb-7e220ac1c65a.png">



Verify the entire setup

<img width="868" alt="Verify entire setup 1" src="https://user-images.githubusercontent.com/10085348/172008852-785408d0-10da-4036-82ad-f4d3e9f9c660.png">
<img width="669" alt="Verify entire setup 2" src="https://user-images.githubusercontent.com/10085348/172008953-c350813c-345f-4f94-a65a-7d992bd3b2d6.png">
<img width="517" alt="verfiy entire setup 3" src="https://user-images.githubusercontent.com/10085348/172008976-4f9f2068-28fd-47c6-ab07-1563396fbde9.png">



Use mkfs ext4 to format the logical volumes with ext4 filesystem

<img width="663" alt="Use mkfs ext4 to format the logical volumes with ext4 filesystem" src="https://user-images.githubusercontent.com/10085348/172009057-ef449441-0014-4572-82ca-b3091e9a8806.png">



Create html, logs directory and use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs

<img width="679" alt="create html and logs directory and rsync utility" src="https://user-images.githubusercontent.com/10085348/172009175-677533b2-9982-4b43-ad31-da5fdc498634.png">



Mount /var/log on logs-lv logical volume and Restore log files back into /var/log directory

<img width="958" alt="Mount :var:log on logs-lv logical volume and Restore log files back into :var:log directory" src="https://user-images.githubusercontent.com/10085348/172009630-0429e67a-d479-4182-a2f8-05dcc113da8c.png">



Update the /etc/fstab file

The UUID of the device will be used to update the /etc/fstab file;

sudo blkid

<img width="1081" alt="Update the :etc:fstab file" src="https://user-images.githubusercontent.com/10085348/172010454-8e6a3f3e-d633-4c0f-9793-378b95f55f5e.png">



Reload daemon and verify setup df -h 

<img width="680" alt="reload daemon and verify setup df -h " src="https://user-images.githubusercontent.com/10085348/172010137-a14a95dc-b7d4-4670-af0a-27b414928369.png">



Launch a second RedHat EC2 instance that will have a role – ‘DB Server’

<img width="1239" alt="DB Server Instance" src="https://user-images.githubusercontent.com/10085348/172010485-a1395587-22e7-449b-bd9a-76c9434da8f9.png">



Create 3 volumes in the same AZ (us-east-2c) as your DB Server EC2, each of 10 GiB and attach all three volumes one by one to your DB Server EC2 instance

<img width="463" alt="DB Server storage volumes" src="https://user-images.githubusercontent.com/10085348/172010718-00d587c6-ef58-456d-8ffa-0bb062122efa.png">



Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/



DB Server Logical and Physical volumes and Volume group

<img width="761" alt="DB Server Logical and Physical volumes and Volume group" src="https://user-images.githubusercontent.com/10085348/172011166-ad21baa2-ad53-4f33-98bb-1431a24ab782.png">



DB Server verify logical volume and create db directory and make filesystem ext4

<img width="761" alt="DB Server verify logical volume and create db directory and make filesystem ext4" src="https://user-images.githubusercontent.com/10085348/172011174-5247e5c6-b96a-4a5f-a90b-abd33229c649.png">



Mount /var/log on db-lv logical volume

<img width="572" alt="DB Server Mount db lv" src="https://user-images.githubusercontent.com/10085348/172011230-db50e660-5825-4dd0-8eff-6bdd7619e6ca.png">



Make db-lv mount persistent
with the command "sudo mount -a"

<img width="787" alt="DB Server db mount persistent" src="https://user-images.githubusercontent.com/10085348/172011363-5427f941-79a4-481c-a593-207871e2b565.png">



Install wget, Apache and dependencies on Web Server EC2

<img width="1071" alt="Install wget, Apache and dependencies" src="https://user-images.githubusercontent.com/10085348/172011486-74eff123-8a8d-40b2-9354-9664f6bb6245.png">



Start Apache

<img width="857" alt="Start Apache" src="https://user-images.githubusercontent.com/10085348/172011654-8b16461d-eb6c-41c3-bfe0-b12f0e9d1b5a.png">



**Install PHP and it’s depemdencies using the commands below:**

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo yum module list php

sudo yum module reset php

sudo yum module enable php:remi-7.4

sudo yum install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1


**Restart Apache**

sudo systemctl restart httpd


**Install mysql and verify status on DB Server**

<img width="897" alt="DB Server Mysql staus" src="https://user-images.githubusercontent.com/10085348/172011836-f85fef16-1e0c-4904-8ebd-0755a88435a5.png">



open MySQL port 3306 on DB Server EC2 to allow inbound connections

<img width="1233" alt="open MySQL port 3306 on DB Server EC2" src="https://user-images.githubusercontent.com/10085348/172011813-3fbb575c-d37f-43d6-a715-d204a643fcae.png">



Bind Web Server IP to enable secure DB access to MySQL database DB Server

<img width="544" alt="mysql bind address" src="https://user-images.githubusercontent.com/10085348/172011898-a46986ed-205e-4877-82c7-0dbd3791ed7e.png">



**Download wordpress and copy wordpress to var/www/html**
  
  mkdir wordpress
  
  cd   wordpress
  
  sudo wget http://wordpress.org/latest.tar.gz
  
  sudo tar xzvf latest.tar.gz
  
  sudo rm -rf latest.tar.gz
  
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  
  cp -R wordpress /var/www/html/

**Configure SELinux Policies**

  sudo chown -R apache:apache /var/www/html/wordpress

  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R

  sudo setsebool -P httpd_can_network_connect=1


**Configure DB to work with WordPress**

<img width="869" alt=" Configure DB to work with WordPress" src="https://user-images.githubusercontent.com/10085348/172013218-85a1931a-b28b-4ae6-8a46-7bfba248a764.png">




Verify that you can connect from your Web Server to your DB server by using mysql-clientand  execute SHOW DATABASES; command to see a list of existing databases.



<img width="643" alt="test that you can connect from your Web Server to your DB server by using mysql-client" src="https://user-images.githubusercontent.com/10085348/172012185-b726ffdb-e66f-4ca6-b1b1-0daac640b940.png">



Try to access from your browser the link to your WordPress 

http://18.117.233.241/wordpress/

<img width="1205" alt="WordPress Installation page" src="https://user-images.githubusercontent.com/10085348/172013263-da7c21b7-5161-4087-aabf-2bd24a2a08f5.png">


<img width="1194" alt="wordpress installation complete" src="https://user-images.githubusercontent.com/10085348/172013339-f834d92f-0e0d-4c80-ae5a-3c4e955e44ac.png">
