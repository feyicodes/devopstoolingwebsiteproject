# DEVOPS TOOLING WEBSITE SOLUTION
This project aims to setup a devops tooling webssite solution using LAMP stack with remote database and NFS servers. 

The project uses a common configuration where several stateless web servers share a common database and also access the same files using Network File System (NFS) as a shared file storage. Althoug NFS server might be located on a seperate hardware, from the point of view of the web servers it seems more like a local file system from where they can serve the same files.

The diagram below shows the architecture of implemented project.

![image 1](images/img1.png)

This project consists of the following components:
* Infrastructure:   AWS
* Webserver Linux:  Red Heart Enterprise Linux 8 (3 instances)
* Database Server: Ubuntu 20.04 + MySQL
* Storage Server: Red Hat Enterprise Linux 8 + NFS Server
* Programming Language: PHP
* Code Repository: GitHub 

The three web servers, one database server and NFS server (which was accompanied by 3  EBS volumes) were launched as instances on AWS. 

![image2](images/img3.png)

![image3](images/img2.png)

For the NFS server, I mounted the 3 volumes and created the required partitionws using:
```bash
    lsblk #to check if the attached volumes
    # The gdisk command to configure the partitions and choosing the necessary options
    sudo gdisk /dev/xvdf
    sudo gdisk /dev/xvdf
    sudo gdisk /dev/xvdf
```
![image4](images/img4.png)

I confirmed that the partitions have been created.

![image5](images/img5.png)

I installed lvm2 and created the physical volumes.

![image6](images/img6.png)

![image7](images/img7.png)

Afterwards, I created the volume group named "webdata-vg" and created the three (3) logical volumes from the volume group namely:
* lv-apps
* lv-logs
* lv-opt

![image8](images/img8.png)

I checked the configurations of the volumes to confirm it met expectations using the command:
```bash
    sudo vgdisplay -v
```

![image9](images/img9.png)

I formatted the three disks using xfs system 

![image10](images/img10.png)

i created the 3 mount points required and mounted each logical volume as required.

![image11](images/img11.png)

After completing the disk configuration, I installed NFS server using the following commands:
```bash
    sudo yum -y update
    sudo yum install nfs-utils -y
    sudo systemctl start nfs-server.service
    sudo systemctl enable nfs-server.service
    sudo systemctl status nfs-server.service
```
![image12](images/img12.png)

![image13](images/img13.png)

![image14](images/img14.png)

I exported the mounts of the webservers' subnet cidr after obtaining it from AWS console.

![image15](images/img15.png)

Thereafter, I setup the necessary permissions required to allow the web servers read, write and execute files on NFS using the following commands:
```bash
    sudo chown -R nobody: /mnt/apps
    sudo chown -R nobody: /mnt/logs
    sudo chown -R nobody: /mnt/opt
    sudo chmod -R 777 /mnt/apps
    sudo chmod -R 777 /mnt/logs
    sudo chmod -R 777 /mnt/opt
    sudo systemctl restart nfs-server.service
```

I configured access to NFS for clients within the same subnet 

![image16](images/img16.png)
![image17](images/img17.png)
![image18](images/img18.png)
![image19](images/img19.png)
![image20](images/img20.png)

I also opened ports TCP 111, UDP 111, UDP 2049 to allow client access to the NFS server. 

![image21](images/img21.png)

I installed mysql on the database server and created a database named **tooling**, a database user named **webaccess** and granted permission to **webaccess** user to manipulate **tooling** database from the web server subnet cidr.

![image22](images/img22.png)

## PREPARING THE WEB SERVERS
I worked on the configuration of the web server to ensure it can serve the same content from the shared storage solutions available in NFS server and Mysql database.

Firstly, I installed the NFS client using  the command:
```bash
    sudo yum install nfs-utils nfs4-acl-tools -y
```

![image23](images/img23.png)

Afterwards, I mounted the NFS on /var/www 
```bash
    sudo mkdir /var/www
    sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
and confirmed that it was mounted in the directory and I also confirmed that a file created in the /var/www directory reflects on the NFS server in /mnt/apps .

![image24](images/img24.png)

![image25](images/img25.png)

I ensured that the mount would start on reboot by including it in /etc/fstab 

![image26](images/img26.png)

i installed Remi repository (a free repo with cutting edge latest software versions not readily avaiable in the OS repo by default), Apache and PHP.

```bash
    sudo yum install httpd -y

    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

    sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

    sudo dnf module reset php

    sudo dnf module enable php:remi-7.4

    sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

    sudo systemctl start php-fpm

    sudo systemctl enable php-fpm

    sudo setsebool -P httpd_execmem 1
```
I repeated the steps taken to configure the first webserver on the remaining two with a few snapshots of the process below:

![image27](images/img27.png)
![image28](images/img28.png)
![image29](images/img29.png)
![image30](images/img30.png)
![image31](images/img31.png)
![image32](images/img32.png)

I located the log folder for Apache on the webserver and mounted it to NFS server's export for logs as well as include it in the /etc/fstab to ensure the mount point persists after reboot.  

![image33](images/img33.png)

![image34](images/img34.png)

I installed git on the web servers using 
```bash
    sudo yum install git
```
and cloned the **tooling repo** for the project on the servers. 

![image35](images/img35.png)

Afterwards, I ensured that the html folder from the cloned repo is deployed to /var/www/html in all the three (3) web servers.

![image36](images/img36.png)

![image37](images/img37.png)

![image38](images/img38.png)

I disabled SELinux in the web servers 

![image39](images/img39.png)

![image40](images/img40.png)

I configure database server by adding tooling database.

![image42](images/img42.png)

I updated the website's configuration to connect to the database.

![image43](images/img43.png)
![image44](images/img44.png)

I logged in to the website successfully.

![image45](images/img45.png)
















