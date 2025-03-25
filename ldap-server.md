# LDAP Server and Client Configuration on Debian 12

Lightweight Directory Access Protocol (LDAP) is a powerful tool for centralized authentication and user management. This guide will walk you through setting up an **OpenLDAP server** on **Debian 12** and configuring a client system to authenticate users via LDAP.

## Step 1: Install OpenLDAP Server on Debian 12

### Update the package and install the OpenLDAP package

```bash
sudo apt update
sudo apt install slapd ldap-utils

```

### During the installation, you will be asked for the configuration of LDAP. If you skip the configuration during installation, you can configure it later by running:
```bash
sudo dpkg-reconfigure slapd
```

### Configuration options during the setup:
* Omit OpenLDAP server configuration? → No
* Domain Name → example.com (creates dc=example,dc=com)
* Organization Name → (Your Organization Name)
* Admin Password → (Choose a secure Password)
* Database backend → MDB
* Remove database when slapd is purged? → No
* Move old database? → Yes


### Verify the status of the OpenLDAP service
```bash
systemctl status slapd.service
```

### Confirm LDAP Directory Structure
```bash
ldapsearch -x -H ldap://localhost -b dc=example,dc=com
```

### Step 2: Create LDAP Organizational Units
Create a file named base_config.ldif to define organizational units:

```bash
vim base.ldif
```
Add the following configuration in the file:
```bash
dn: ou=users,dc=example,dc=com
objectClass: organizationalUnit
ou: users

dn: ou=groups,dc=example,dc=com
objectClass: organizationalUnit
ou: groups
```

Load the configurations into LDAP. You will be asked for the admin password you set while installing OpenLDAP.

```bash
ldapadd -x -D "cn=admin,dc=example,dc=com" -W -f base_config.ldif
```

### Step 3: Add LDAP Groups
Create a file named addgroup.ldif:

```bash
vim addgroup.ldif
```

Add the following configuration in the file:
```bash
dn: cn=ldapusers,ou=groups,dc=example,dc=com
objectClass: posixGroup
cn: ldapusers
gidNumber: 1001
memberUid: johndoe
```

Load the group into LDAP
```bash
ldapadd -x -D "cn=admin,dc=example,dc=com" -W -f addgroup.ldif
```

### Step 4: Add an LDAP User
Create a file named addgroup.ldif:

```bash
vim addguser.ldif
```

Add the following content in the file:
```bash
dn: uid=johndoe,ou=users,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: John Doe
sn: Doe
uid: johndoe
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/johndoe
loginShell: /bin/bash
userPassword: {CRYPT}$6$saltsalt$hashedpasswordhere
```

Load the user into LDAP:
```bash
ldapadd -x -D "cn=admin,dc=example,dc=com" -W -f adduser.ldif
```

Manually set the password for the user:
```bash
ldappasswd -s securepassword -D "cn=admin,dc=example,dc=com" -W "uid=johndoe,ou=users,dc=example,dc=com"
```

### Step 5: LDAP Client Configuration on Debian 12
On the client system, install the LDAP authentication packages:
```bash
sudo apt update
sudo apt install libnss-ldapd libpam-ldapd ldap-utils nscd
```

### During the installation, provide the following details:
* LDAP URI: ldap://<server-ip>
* Search base: dc=example,dc=com
* LDAP version: 3

If needed, reconfigure LDAP settings:
```bash
sudo dpkg-reconfigure libnss-ldapd
```

Edit the /etc/nsswitch.conf file:
```bash
sudo vim /etc/nsswitch.conf
```

Update the following lines to ensure that system authentication checks LDAP after local files:
```bash
passwd:         files ldap
group:          files ldap
shadow:         files ldap
```

## Enable Home Directory Creation for LDAP Users
Edit the PAM session configuration file:
```bash
sudo nano /etc/pam.d/common-session
```

Add the following line at the end to ensure that a home directory is created automatically for LDAP users upon their first login:
```bash
session required pam_mkhomedir.so skel=/etc/skel umask=077
```

Restart the caching service to apply the changes:
```bash
sudo systemctl restart nscd
```

## Step 6: Verify LDAP User Authentication
Check if the LDAP user is recognized by the system
```bash
getent passwd johndoe
```

Expected output for the command:
```bash
johndoe:x:1001:1001:John Doe:/home/johndoe:/bin/bash
```

To test logging in as the LDAP user:
```bash
su - johndoe
```

Alternatively, try using SSH:
```bash
ssh johndoe@<client-ip>
```

## Note: If everything is set up correctly, the user should log in successfully, and their home directory will be created automatically.

