# Apache Web Server Setup and Configuration

## Apache is an open-source web server software developed and maintained by the **Apache Software Foundation**. It is one of the most widely used web servers in the world and is known for its reliability, flexibility, and strong community support.

### **Key Features of Apache Web Server:**
1. **Cross-Platform Support** – Runs on Linux, Windows, macOS, and other operating systems.
2. **Modular Architecture** – Supports modules like `mod_rewrite`, `mod_ssl`, and `mod_proxy` for extended functionalities.
3. **Virtual Hosting** – Allows hosting multiple websites on a single server.
4. **Security Features** – Supports SSL/TLS encryption, authentication, and access control.
5. **Custom Configurations** – Configurable through `.htaccess` and `httpd.conf` files.
6. **Compatibility** – Works with PHP, Python, Perl, and other server-side scripting languages.

### **Common Use Cases of Apache:**
- Hosting static and dynamic websites
- Running web applications (WordPress, Joomla, Drupal, etc.)
- Reverse proxy and load balancing
- Secure communication via HTTPS

---

# Apache Web Server Setup and Configuration

## 1. Check if Apache is Installed
Use the following command to check if Apache (`httpd`) is already installed:
```bash
rpm -qa | grep httpd
```

## 2. Install Apache Web Server
Install Apache using `yum`:
```bash
yum install httpd
```
Install all related Apache packages:
```bash
yum install httpd*
```

## 3. Verify Apache Installation
Check package information:
```bash
rpm -qi httpd
```
List installed files:
```bash
rpm -ql httpd
```
List configuration files:
```bash
rpm -qc httpd
```
List documentation files:
```bash
rpm -qd httpd
```

## 4. Apache Configuration
### View Apache Configuration
Check the main configuration file:
```bash
tail /etc/httpd/conf/httpd.conf
```
Find the `conf.modules.d` directive:
```bash
grep conf\.modules\.d /etc/httpd/conf/httpd.conf
```
Find the `conf.d` directive:
```bash
grep conf\.d /etc/httpd/conf/httpd.conf
```

## 5. Manage Apache Service
### Check Apache Status
Check if the Apache service is running:
```bash
systemctl status httpd.service
```
### Start Apache Service
Start the Apache service:
```bash
systemctl start httpd.service
```
### Enable Apache to Start on Boot
Enable Apache to start on system boot:
```bash
systemctl enable httpd.service
```
### Restart Apache Service
Restart the Apache service:
```bash
systemctl restart httpd.service
```

## 6. Network and Port Verification
### Check Open Ports
Check open ports and services:
```bash
netstat -nltup
```
Check if port 80 is open:
```bash
netstat -nltup | grep 80
```
Check if Apache is listening:
```bash
netstat -nttup | grep httpd
```

## 7. Firewall Configuration
### Firewalld Configuration for Apache Web Server
#### 1. Check Firewalld Status
Check if `firewalld` is running:
```bash
systemctl status firewalld
```
#### 2. Start and Enable Firewalld
Start `firewalld`:
```bash
systemctl start firewalld
```
Enable `firewalld` at boot:
```bash
systemctl enable firewalld
```
#### 3. List Current Firewalld Rules
List all active zones and rules:
```bash
firewall-cmd --list-all
```
List active services and ports:
```bash
firewall-cmd --list-services
```
```bash
firewall-cmd --list-ports
```
#### 4. Configure Firewalld for Apache
Allow HTTP (port 80) and HTTPS (port 443) traffic:
```bash
firewall-cmd --permanent --add-service=http
```
```bash
firewall-cmd --permanent --add-service=https
```
Reload `firewalld`:
```bash
firewall-cmd --reload
```
#### 5. Open Specific Ports for Apache
Open port 80 manually:
```bash
firewall-cmd --permanent --add-port=80/tcp
```
Open port 443 for HTTPS:
```bash
firewall-cmd --permanent --add-port=443/tcp
```
Open additional ports (optional):
```bash
firewall-cmd --permanent --add-port=8080/tcp
```
Reload `firewalld` after adding ports:
```bash
firewall-cmd --reload
```
Check open ports:
```bash
firewall-cmd --list-ports
```
#### 6. Remove Rules (Optional)
Remove HTTP and HTTPS services:
```bash
firewall-cmd --permanent --remove-service=http
```
```bash
firewall-cmd --permanent --remove-service=https
```
Remove specific open ports:
```bash
firewall-cmd --permanent --remove-port=80/tcp
```
```bash
firewall-cmd --permanent --remove-port=443/tcp
```
Reload `firewalld`:
```bash
firewall-cmd --reload
```

#### 7. Configure Firewalld Zones (Optional)
List available zones:
```bash
firewall-cmd --get-zones
```
Assign Apache traffic to the `public` zone:
```bash
firewall-cmd --zone=public --add-service=http --permanent
```
```bash
firewall-cmd --zone=public --add-service=https --permanent
```
Reload firewall settings:
```bash
firewall-cmd --reload
```
#### 8. Enable Logging (Optional)
Enable logging for dropped packets:
```bash
firewall-cmd --set-log-denied=all
```
View firewall logs:
```bash
journalctl -f -u firewalld
```

## 8. Apache Process and User Management
### Check Running Processes
Check Apache processes:
```bash
ps -aux | grep httpd
```
### Check Apache User and Group
Find the Apache user:
```bash
cat /etc/passwd | grep apache
```
Find the Apache group:
```bash
cat /etc/group | grep apache
```

## 9. Configure Website Files
### Create or Edit Web Files
Navigate to the document root:
```bash
cd /var/www/html
```
Create or edit an `index.html` file:
```bash
vim index.html
```
### Remove Existing File
Remove the existing file:
```bash
rm -f index.html
```
### Set Ownership of Web Files
Set ownership of the website files to Apache user:
```bash
chown -Rv apache:apache /var/www/html/*
```

## 10. Load CSS Files
### Download and Extract Template Files
Download template:
```bash
wget https://template.com/example.zip
```
Unzip the template:
```bash
unzip example.zip
```
Remove the ZIP file:
```bash
rm -rf example.zip
```
Move extracted folder:
```bash
mv -v site1 /var/www/html/
```
### Configure Apache to Load CSS Files
Add the following line to `httpd.conf`: line number 311
```bash
AddType text/html .shtml
AddType text/css .css
AddOutputFilter INCLUDES .shtml
```
Edit the configuration file:
```bash
vim /etc/httpd/conf/httpd.conf
```
Restart Apache:
```bash
systemctl restart httpd.service
```
Set ownership:
```bash
chown -Rv apache:apache /var/www/html/*
```

## 11. Test Apache Setup
### Test Configuration
Create a test file:
```bash
echo 'Test123' > /var/www/html/test.html
```
### Check if Apache is Running
Use `curl` to test if Apache is responding:
```bash
curl http://localhost
```

**Apache Web Server Installation and Configuration Completed**

