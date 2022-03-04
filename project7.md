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

**create mount points on /mnt directory for logical volumes lv-apps, lv-logs, and lv-opt**

