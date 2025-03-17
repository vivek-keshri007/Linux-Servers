# FTP (File Transfer Protocol) Guide for Debian 12

## What is FTP, and Why Use It?
FTP (File Transfer Protocol) is a standard network protocol used to transfer files between a client and server over a TCP/IP network. It’s an essential tool for webmasters, developers, system administrators, and businesses needing to share large files across different systems.

Some benefits of using FTP include:

Ease of use: FTP clients are widely available, making it easy to connect to FTP servers.
Security: With proper configuration, FTP servers can be secured using SSL/TLS encryption (FTPS).
Efficiency: FTP allows for large file transfers without losing data integrity.

## Prerequisites

A fresh Debian 12 installation: This guide assumes that Debian 12 is already installed and updated.
Root or sudo privileges: You need superuser privileges to install software and configure the server.
A basic understanding of the Linux command line: Though beginners can follow along, familiarity with terminal commands will make the process smoother.

## Step 1: Update Your System
```sh
apt update
```

## Step 2: Install the FTP Server Software (vsftpd)
```sh
apt search vsftpd 
apt install vsftpd -y
```

## Step 3: Start and Enable the FTP Service
```sh
systemctl start vsftpd.service
systemctl enable vsftpd.service
```

## Step 4: Allow the ftp service or port in the firewall
```sh
ufw allow ftp
```
    or
```sh
ufw allow 20/tcp
ufw allow 21/tcp
```

## Step 5: Configure the FTP Server (vsftpd)
The configuration file of vsftpd is located at `/etc/vsftpd.conf`.
```sh
vim /etc/vsftpd.conf
```
### ADD PASSIVE MODE FOR ALL THE CONFIGURATION
```sh
pasv_enabl=YES
pasv_min_port=55000
pasv_min_port=55999
```

### Allow Anonymous Access:
By default, vsftpd disallows the anonymous access for the security reasons if you want to allow the anonymous users to log in without credentials (for public sharing) set:
```sh
anonymous_enable=YES
systemctl restart vsftpd.service
```
The default sharing directory is `/srv/ftp`. Place the public data here and can access through the client. You can use file explorer, any software, or an FTP client in Linux to access. Just provide the username from above and any password.

Allowed usernames: `ftp` / `anonymous` / `anon`
```sh
ftp ip_address
```
**Note**: The anonymous users are given the power of get (read) only. They cannot write or place content on the server.

To Allow the Anonymous users to be able to write then:
Create a folder at `/srv/ftp/sharedFolder`, and provide 777 permission to the folder only.
```sh
mkdir -p /srv/ftp/sharedFolder
chmod 777 /srv/ftp/sharedFolder
```
Add some configurations to the config file.
```sh
anon_root=/srv/ftp/sharedFolder    # root directory for the anonymous users
write_enable=YES                   # Permission to be able to write
anon_upload_enable=YES             # Permission to upload the folder
systemctl restart vsftpd.service
```
**NOTE**: It is better to avoid using and enabling the anonymous user because an anonymous user with write permission can lead to a security breach.

## 2. Local User Access:
To allow the users of the server to be able to log in, we need to add some configuration to the file.

The default directory for the local users will be `/home/user` but needs to be set first, you can set any directory as after login dir.
```sh
mkdir -p /backup/ftpfolder
chmod 777 /backup/ftpfolder
```
```sh
local_enable=YES                   # allow the local users
write_enable=YES                   # allow the write access
local_root=/backup/folder          # same directory for all users
allow_writeable_user               # to be able to login the user in chroot (this can be risky to enable)
chroot_local_user=YES              # user will not be able to change the directory
```
You can specify the home directory as well but need to provide a folder available in all the user directories with 777 permission to able to write.
```sh
user_sub_token=$USER
local_root=/home/$USER/folder-name
systemctl restart vsftpd.service
```

## 3. Enable the root login:
FTP server does not allow the root to log in, to enable the root login we need to check out the file `/etc/ftpusers`. This file contains the list of users whose login access is restricted.
```sh
vim /etc/ftpusers
```
Comment the root user to enable the FTP login through root as well.
```sh
systemctl restart vsftpd.service
```

## 4. Deny And Allow List
### Deny List:
This can be done with a file `/etc/ftpusers`, this file is used for deny FTP login. Just create an entry of the user whom to block or deny.
```sh
vim /etc/ftpusers
```
```sh
user-name
systemctl restart vsftpd.service
```

### Allow List:
To create the allow list, there are some configurations need to be added. We need to create a file at `/etc/file-name`
```sh
vim /etc/allow_list
```
Make the entry of the user in the file
```sh
local_root=/backup/ftpfolder
chroot_local_user=YES
allow_writeable_chroot=YES
userlist_enable=YES
userlist_file=/etc/allow_list
userlist_deny=NO              # allow the users from the list
systemctl restart vsftpd.service
```

## Step 5: Test the service through FTP client
FTP client provides various commands to handle the operations
```sh
ftp ip_address
```
Check out the commands available in the FTP client
After login, use `?` to check the commands available
```sh
?
```
To get the data from the server:
```sh
get file-name
```
### TO GET MULTIPLE FILES AT ONCE
```sh
mget file1 file2 file3
```
To put the data onto the server:
```sh
put file-name
```
### TO PUT MULTIPLE FILES AT ONCE
```sh
mput file1 file2 file3
