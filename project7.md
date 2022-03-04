# Documentation for project 7

**In this project you will implement a solution that consists of following components:**

1. Infrastructure: AWS
2. Webserver Linux: Red Hat Enterprise Linux 8
3. Database Server: Ubuntu 20.04 + MySQL
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Serve
5. Programming Language: PHP
6. Code Repository: GitHub

** 3 tier Web Application Architecture with a single Database and an NFS Server as a shared files storage**


![3 tier](./images/3tier.png)

### STEP 1 – PREPARE NFS SERVER

1. Set up RHEL EC2 instance NFS Server

![NFS](./images/NFS%20Server.png)

2. create EBS volume and attche to newly created NFS server

![EBS](./images/EBS-attached.png)

**check for disk**

`lsblk`

![lsblk](./images/lsblk%20command.png)

**start partition**

`sudo gdisk`

![gdisk](./images/sudo%20gdisk%20command.png)

**check disk partition**

`lsblk`

![lsblk](./images/lsblk%20showing%20all%20gdisk%20partion.png)

**install lvm2 package**

`sudo yum install lvm2 -y`

![install lvm2](./images/sudo%20yum%20install%20lvm2%20-y.png)

**run a disk scan**

`sudo lvmdiskscan`

![lvmdiskscan](./images/sudo%20lvmdiskscan.png)

**create PV**

`sudo pvcreate`

![pvcreate](./images/sudo%20pvcreate.png)

**check**

`sudo pvs`

![pvs](./images/sudo%20pvs%20to%20confirm.png)

**create VG name vg-webdata**

`sudo vgcreate`

![vgcreate](./images/sudo%20vgcreate%20and%20vgs%20to%20confirm.png)

**create LV for apps. logs, opt**

`sudo lvcreate -n lv-apps -L 9G vg-webdata`

`sudo lvcreate -n lv-logs -L 9G vg-webdata`

`sudo lvcreate -n lv-opt -L 9G vg-webdata`

![lvcreate](./images/sudo%20lvcreate%20lv-apps%20lv-logs%20lv-opt%20from%20vg-webdata.png)

**check**

`sudo lvs `
`sudo vgs `
`sudo pvs `
`lsblk `

![lvs,ovs,vgs,lsblk](./images/sudo%20lvs%20pvs%20vgs%20and%20lsblk.png)

**format the disk as xfs for apps, loga, and opt**

`sudo mkfs xfs /dev/vg-webdata/lv-apps`

`sudo mkfs xfs /dev/vg-webdata/lv-logs`

`sudo mkfs xfs /dev/vg-webdata/lv-opt`

![format disk](./images/format%20with%20xfs%20.png)

**create mount points on /mnt directory for logical volumes lv-apps, lv-logs, and lv-opt and mount on /mnt/apps /mnt/logs /mnt/opt**

`sudo mkdir mnt/apps`

`sudo mkdir mnt/logs`

`sudo mkdir mnt/opt`

![mount points](./images/sudo%20mount%20on%20directory.png)

**Update your NFS Server and run the following command**

`sudo yum update -y`

![update server](./images/sudo%20yum%20update%20-y%20NFS%20server.png)

`sudo yum install nfs-utils -y`

![nfs-utils](./images/sudo%20yum%20install%20nfs-utils%20.png)


`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

![nfs status](./images/sudo%20systemctl%20start%2C%20enable%20and%20status%20nfs-server.service.png)

**Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS**

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/log`

`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/log`

`sudo chmod -R 777 /mnt/opt`

`sudo systemctl restart nfs-server.service`

![setting permission for NFS Files](./images/sudo%20chown%20and%20chmod%20for%20apps%2C%20logs.%20opts.png)

**Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 )**

`sudo vi /etc/exports`

![access to NFS](./images/sudo%20vi.png)

**exportfs**

`sudo exportfs -arv`

![export](./images/sudo%20export.png)

**Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)**

`rpcinfo -p | grep nfs`

![check ports](./images/check%20ports%20used%20by%20NFS.png)

**Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049**

![open ports](./images/edit%20NFS%20inbounds%20SG.png)

# STEP 2 — CONFIGURE THE DATABASE SERVER

**create an ubuntu server for mysql database and configure it**

`sudo apt update`

![update](./images/sudo%20apt%20update%20mysql%20server.png)

`sudo apt upgrade`

![upgrade](./images/sudo%20apt%20upgrade%20mysql%20server.png)

`sudo yum install mysql-server -y`

![install mysql](./images/sudo%20apt%20install%20mysql-server%20-y.png)

1. Create a database and name it tooling
2. Create a database user and name it webaccess
3. Grant permission to webaccess user on tooling database to do     anything only from the webservers subnet cidr

`sudo mysql`

![mysql](./images/sudo%20mysql.png)

**create database with the name 'tooling', create user with the name 'webaccess', grant psermission to user 'webaccess' on 'tooling' database and finally flush privileges**

![create database](./images/create%20databse%20tooling%2C%20user%2C%20grant%20privileges%20and%20flush%20privileges.png)

**show databases and use tooling**

![show databases](./images/show%20databases.png)

# Step 3 — Prepare the Web Servers

**We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.**

**You already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).**

**This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.**


*During the next steps we will do following*

- Configure NFS client
- Deploy a Tooling application to our Web Servers into a shared NFS folder
- Configure the Web Servers to work with a single MySQL database.

**Launch a new EC2 instance with RHEL 8 Operating System**

**update the instance**

`sudo yum update`

![update](./images/one%20of%20the%20web%20server%20sudo%20yum%20update.png)

**Install NFS client**

`sudo yum install nfs-utils nfs4-acl-tools -y`

![installing nfs-utils nfs4-acl-tools](./images/install%20nfs-utils%20nfs4-acl-tools.png)

**Mount /var/www/ and target the NFS server’s export for apps**

`sudo mkdir /var/www`

**insert NFS private IP Address**

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

**Verify that NFS was mounted successfully by running df -h**

`df -h`

![df -h](./images/df%20-h.png)

**Make sure that the changes will persist on Web Server after reboot**

`sudo vi /etc/fstab`

**add the following line in the executed command above inserting the Private IP Address of the NFS-Server**

*<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0*

![fsatb](./images/fstab%20.png)

# Install Remi’s repository, Apache and PHP

`sudo yum install httpd -y`

![httpd](./images/sudo%20yum%20install%20httpd.png)

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![dnf install](./images/sudo%20dnf%20install%20httpd.png)

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![remirepo](./images/sudo%20intstall%20remirepo.png)

`sudo dnf module reset php`

![dnf module](./images/sudo%20reset%20php.png)

`sudo dnf module enable php:remi-7.4`

![dnf enable](./images/sudo%20enable%20php.png)

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

![dnf install php](./images/sudo%20install%20php%20php-opacache.png)

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`sudo setsebool -P httpd_execmem 1`

![start,enable setsebool](./images/restart%2C%20enable%20and%20setsebool%20httpd.png)

**Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №4 to make sure the mount point will persist after reboot.**

*sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www*

`sudo mount -t nfs -o rw,nosuid 172.31.12.222:/mnt/apps /var/log/httpd`

![mount](./images/mount%20mnt%20to%20NFS%20server.png)

**Repeat step №4 to make sure the mount point will persist after reboot.**

`sudo vi /etc/fstab`

![edit logs](./images/edit%20logs%20httpd.png)

**Fork the tooling source code from Darey.io Github Account to your Github account.**

*but first install git in your web-server*

`sudo yum install git`

![install git](./images/sudo%20install%20git.png)

*initialize git*

`git init`

![git init](./images/git%20init%20and%20git%20clone.png)

*NB*
**How to Fork in Github**

[fork in github](https://github.com/darey-io/tooling)

![fork in github](./images/fork%20tooling%20in%20github.png)

**run the following command**

`ls`

`cd tooling/`

![files in tooling](./images/files%20in%20tooling%20directory.png)

**Still in your tooling directory Ensure that the html folder from the repository is deployed to /var/www/html**

`sudo cp -R html/. /var/www/html`

![cp -R](./images/sudo%20cp%20-R%20html%20to%20html..png)

**Update the website’s configuration to connect to the database**


`sudo /var/www/html/functions.php`

*replace local with DB private IP addrr, admin to be 'webaccess' admin to be 'password' and 'tooling'

![function.php](./images/function.php.png)

*Install mysql-client on the web-server*

`sudo yum install mysql -y`

![install mysql-client](./images/sudo%20install%20mysql%20on%20webserver.png)

*configure DB Security Group add new rule*

![DB security group](./images/edit%20inbound%20rule.png)

**change bind addresss of DB-server to 0.0.0.0**

`sudo vi /ect/mysql/mysql.conf.d/mysqld.cnf`

![change bind addrr](./images/change%20bind%20addrr%20of%20mysql%20to%200.0.0.0.png)

**Apply tooling-db.sql script to your database using this command on the webserver**

`mysql -h 172.31.9.70 -u webaccess -p tooling < tooling-db.sql`

**Go to your DB server**

`msql`

![mysql](./images/sudo%20mysql.png)

*run these commands*

`show databases`

`show tables`

`select * from table`

![DB Admin](./images/db%20admin%20.png)

**webserver showing same admin and password as DB server**

`sudo vi tooling-db.sql`

![DB admin](./images/webserver%20showing%20same%20admin.png)

**NOTE: Do not forget to open TCP port 80 on the Web Server.**

**NB: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0**

*To make this change permanent – open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabledthen restrt httpd.*

`sudo vi /etc/sysconfig/selinux`
*NB: SELINUX=disabled*

*Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php 
 make sure you can login into the websute with myuser user*

 *refersh the URL page*

 ![result](./images/result.png)

 *enter user name: admin*
 *enter password:  admin*

 ![result1](./images/result1.png)

 ![result2](./images/result2.png)

 # YAAAAY! THAT CONCLUDES THIS PROJECT
 # THANK YOU.



