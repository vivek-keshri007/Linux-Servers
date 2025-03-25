# APACHE SERVER IN CENTOS9

## Install the httpd
```bash
yum install httpd
```

## start and enable the service
```bash
systemctl restart httpd.service
```

## check the service
```bash
netstat -nltup
```

## allow the port or service in firewall
```bash
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent
```

## check the firewall allowed services
```bash
firewall-cmd --list-all
```

## restart firewall
```bash
systemctl restart firewall-cmd
```

## check if the default page running or not
```bash
http://IP-ADDRESS
```

## create a Folder inside the /var/www/html
```bash
mkdir site1
```

## put the content and inside the folder and change the ownership of folder
```bash
chown -Rv apache:apache /var/www/html/site1
```

## bind the website( put the binding configuration inside the /etc/httpd/conf/httpd.conf)
binding using IP-Address and Ports

```bash
# this will run the same website on all the ip addresses

<Virtualhost *>
        DocumentRoot /var/www/html/site1/
        DirectoryIndex index.html
</Virtualhost>


#this binding will run on all the ip's on port 80

<Virtualhost 0.0.0.0:80>
        DocumentRoot /var/www/html/Site-Folder/
        DirectoryIndex index.html
</Virtualhost> 


# configure the site for an ip on port 80

<Virtualhost 192.168.43.51:80>
        DocumentRoot /var/www/html/site1/
        DirectoryIndex index.html	
</Virtualhost>


# bind using the port
Listen 8080

<Virtualhost 192.168.43.51:8080>
        DocumentRoot /var/www/html/site1/
        DirectoryIndex index.html
</Virtualhost>
```


## want to bind multiple websites using the ip address (OPTIONAL):
```bash
# configure the site for an ip on port 80

<Virtualhost 192.168.43.51:80>
        DocumentRoot /var/www/html/site1/
        DirectoryIndex index.html
</Virtualhost>

# binding of the site for the second ip
<Virtualhost 192.168.43.52:80>
        DocumentRoot /var/www/html/site2/
        DirectoryIndex index.html
</Virtualhost>
```


## you can create separate binding file for every site (OPTIONAL)
create a file at /etc/httpd/conf.d/filename.conf and put the binding conf in it.
```bash
vim /etc/httpd/conf.d/filename.conf
```


## Binding the website using the name
```bash
<Virtualhost 192.168.43.51:80>
	DocumentRoot /var/www/html/site1
	DirectoryIndex index.html
</Virtualhost>

<Virtualhost 192.168.43.51:80>
	ServerName phoenix.local
	DocumentRoot /var/www/html/site1
        DirectoryIndex index.html
	ServerAlias www.armour.local
</Virtualhost>

```


## Bind the website using the ssl 

```bash
yum install mod_ssl	
```

Generating the Self Signed Certificate and binding it with the site
```bash
cd /opt/ssl
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
```

Copy the certificates to the main folder
```bash
cp -v /opt/ssl/cert.pem /etc/pki/tls/certs/cert.pem
cp -v /opt/ssl/key.pem /etc/pki/tls/private/key.pem
```
## Restart the service
This will ask the same password entered while generating the certificate
```bash
systemctl restart httpd.service
```

## binding configuration for the ssl
```bash
# binding using the self signed certificate

<Virtualhost 192.168.43.51:443>
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/certs/cert.pem
        SSLCertificateKeyFile /etc/pki/tls/private/key.pem
        ServerName armour.local
        DocumentRoot /var/www/html/site1/
        DirectoryIndex index.html
        ServerAlias www.armour.local
</Virtualhost>
```
