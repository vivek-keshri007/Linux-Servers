# **Caching Nameserver Configuration**

## **Introduction**
A **caching nameserver** temporarily stores DNS query results to improve performance and reduce the load on authoritative nameservers. This guide covers the setup and configuration of a caching nameserver using **BIND (Berkeley Internet Name Domain)**.

---

## **1. Check and Update Hostname**
### **Display Current Hostname**
```bash
hostname
```

### **Update Hostname**
Open the hostname configuration file:
```bash
vim /etc/hostname
```
Modify the hostname value as needed, then save and exit.

### **Reboot the System**
```bash
reboot
```

### **Confirm the Hostname Change**
```bash
hostname
```

---

## **2. Check DNS Resolver Configuration**
### **View `/etc/resolv.conf` for DNS Settings**
```bash
cat /etc/resolv.conf
```
Example:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

---

## **3. Install and Configure BIND**
### **Check if BIND is Installed**
```bash
rpm -qa | grep bind
```

### **Install BIND and Utilities**
```bash
yum install bind bind-utils -y
```

### **Verify Installation**
```bash
rpm -qa | grep bind
```

### **Get Information About the Installed Package**
```bash
rpm -qi bind
```

### **List Installed Files**
```bash
rpm -ql bind
```

### **List Configuration Files**
```bash
rpm -qc bind
```

---

## **4. Key Configuration Files**
### **Main Configuration Files:**
- `/etc/named.conf` – Main configuration file for BIND.
- `/etc/named.iscdlv.key` – Contains TSIG (Transaction Signature) keys.
- `/etc/named.rfc1912.zones` – Defines default zones (e.g., localhost, reverse lookups).
- `/etc/named.root.key` – Root trust anchor key for DNSSEC.

### **Working Directories:**
- `/var/named/` – Stores zone files and cache data.
- `/var/named/data/` – Stores BIND-generated data files.
- `/var/named/dynamic/` – Stores dynamically generated zone files.
- `/var/named/named.ca` – Contains the list of root DNS servers.
- `/var/named/named.empty` – Defines empty zones.
- `/var/named/named.localhost` – Defines the zone for localhost.
- `/var/named/named.loopback` – Reverse lookup for 127.0.0.1.
- `/var/named/slaves/` – Stores slave zone files.

### **View Cache File**
```bash
cat /var/named/named.ca
```

## **5. Manage BIND Service**
### **Check Service Status**
```bash
systemctl status named.service
```

### **Start the Service**
```bash
systemctl start named.service
```

## **6. Configure Firewall Rules (Firewalld)**
### **Allow DNS Traffic (TCP and UDP Port 53)**
```bash
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --permanent --add-port=53/udp
firewall-cmd --reload
```

### **Check Open Ports for Named**
```bash
netstat -nltup | grep named
netstat -nltup | grep 53
```

### **Test Using `dig`**
```bash
dig google.com @127.0.0.1
```

---

## **7. Configure Network Settings**
### **(CentOS 7) Modify Network Interface Configuration**
Edit the network interface file:
```bash
vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
#### **Example:**
```
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
IPADDR=192.168.1.37
PREFIX=24
GATEWAY=192.168.1.1
DNS1=127.0.0.1
ONBOOT=yes
```

### **Check CentOS Version**
```bash
cat /etc/redhat-release
```

### **(CentOS 9) Modify NetworkManager Connection Configuration**
```bash
vim /etc/NetworkManager/system-connections/enp0s3.nmconnection
```
#### **Example:**
```bash
[connection]
id=enp0s3
uuid=5dabe1f1-79aa-3751-8ba8-b0f01d817434
type=ethernet
timestamp=1742804496

[ethernet]

[ipv4]
address1=192.168.112.145/24
dns=127.0.0.1;
gateway=192.168.112.98
method=manual

[ipv6]
addr-gen-mode=default
method=disabled

[proxy]
```

### **Check UUID**
```bash
nmcli connection show
nmcli connection show enp0s3 | grep uuid
```

### **Reboot OS**
```bash
reboot
```

---

## **8. Configure BIND (named.conf)**
Edit the BIND configuration file:
```bash
vim /etc/named.conf
```
#### **Configuration Data:**
```bash
options {
    listen-on port 53 { 127.0.0.1; 192.168.112.145; };
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    recursion yes;
    allow-query { 127.0.0.1; 192.168.1.0/24; };
    dnssec-validation yes;
};
```

#### **Main File:**
```bash
// named.conf
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching-only nameserver (as a localhost DNS resolver only).

options {
    listen-on port 53 { 127.0.0.1; 192.168.112.145; };
    listen-on-v6 port 53 { ::1; };
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    secroots-file "/var/named/data/named.secroots";
    recursing-file "/var/named/data/named.recursing";
    allow-query { localhost; 192.168.112.0/24; };

    recursion yes;
    dnssec-validation yes;
    managed-keys-directory "/var/named/dynamic";
    geoip-directory "/usr/share/GeoIP";
    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";

    /* Security policies */
    include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
    channel default_debug {
        file "data/named.run";
        severity dynamic;
    };
};

zone "." IN {
    type hint;
    file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

---

### **Restart BIND Service**
```bash
systemctl restart named.service
```

### **Check Port**
```bash
netstat -nltup | grep named
```

Here is the formatted version of your data:  

---

## **8. Define ACL in named.conf**
### **Edit Configuration File**
```bash
vim /etc/named.conf
```

### **Configuration Data**
```bash
acl ns_ip_add {
    127.0.0.1;
    192.168.112.145;
    192.168.112.146;
    192.168.112.147;
};

acl mynetwork {
    127.0.0.1;
    192.168.112.0/24;
};

options {
    listen-on port 53 { ns_ip_add; };
    listen-on-v6 port 53 { ::1; };
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    secroots-file "/var/named/data/named.secroots";
    recursing-file "/var/named/data/named.recursing";
    allow-query { mynetwork; };
};
```

---

### **Main Configuration File**
```bash
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching-only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

acl ns_ip_add {
    127.0.0.1;
    192.168.112.145;
    192.168.112.146;
    192.168.112.147;
};

acl mynetwork {
    127.0.0.1;
    192.168.112.0/24;
};

options {
    listen-on port 53 { ns_ip_add; };
    listen-on-v6 port 53 { ::1; };
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    secroots-file "/var/named/data/named.secroots";
    recursing-file "/var/named/data/named.recursing";
    allow-query { mynetwork; };

    /*
     - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
     - If you are building a RECURSIVE (caching) DNS server, you need to enable recursion.
     - If your recursive DNS server has a public IP address, you MUST enable access control
       to limit queries to your legitimate users. Failing to do so will cause your server 
       to become part of large-scale DNS amplification attacks. Implementing BCP38 within 
       your network would greatly reduce such attack surfaces.
    */
    recursion yes;

    dnssec-validation yes;

    managed-keys-directory "/var/named/dynamic";
    geoip-directory "/usr/share/GeoIP";

    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";

    /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
    include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
    channel default_debug {
        file "data/named.run";
        severity dynamic;
    };
};

zone "." IN {
    type hint;
    file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

---

## **9. Restart Services**
### **Restart BIND Service**
```bash
systemctl restart named.service
```

### **Restart Firewall**
```bash
systemctl restart firewalld.service
```

---

## **10. Test Setup**
### **Use `dig` to Test DNS Resolution**
```bash
dig google.com @192.168.112.145
```
```bash
dig youtube.com @192.168.112.145
```

---

## **11. Capture DNS Traffic Using tcpdump**
### **Basic UDP Traffic Capture**
```bash
tcpdump -t udp
```

### **Capture DNS Traffic on a Specific Interface**
```bash
tcpdump -i enp0s3 port 53 
```

### **Capture DNS Queries and Responses**
```bash
tcpdump -n udp port 53
```

### **Capture with Interface and Port Filtering**
```bash
tcpdump -t udp -i enp0s3 port 53 
```

```bash
tcpdump -n -t udp -i enp0s3 port 53
```

---


