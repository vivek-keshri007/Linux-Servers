## What is DNS ( Domain Name Server ) ?

A DNS server is a specialized server that performs domain name resolution, translating human-readable domain names into IP addresses that computers and devices use to communicate with each other. Without DNS, users would need to remember and input IP addresses to access websites, which would be much difficult to do.

DNS servers can be configured in different roles:

- **Cache Only DNS Servers**: They store previously resolved domain names for faster lookups.
- **Authoritative DNS Servers**: These provide definitive answers for domains they manage.
- **Recursive DNS Servers**: These servers resolve domain names by querying other DNS servers.

DNS also helps route emails, ensure efficient load balancing across servers, and improve the overall security of your network through mechanisms like **DNSSEC** (DNS Security Extensions).

## Step 1: Install BIND and Related Packages

```bash
yum install bind bind-utils -y
```

or 

```bash
yum install bind*
```

Check the files and information of installed Package

```bash
rpm -qa bind
rpm -qc bind   #check configuration files
```

Check the details of all root in servers

```bash
vim /var/named/named.ca
```

```
ANSWER SECTION:

518400    IN    NS    a.root-servers.net.
518400    IN    NS    b.root-servers.net.
518400    IN    NS    c.root-servers.net.
518400    IN    NS    d.root-servers.net.
518400    IN    NS    e.root-servers.net.
518400    IN    NS    f.root-servers.net.
518400    IN    NS    g.root-servers.net.
518400    IN    NS    h.root-servers.net.
518400    IN    NS    i.root-servers.net.
518400    IN    NS    j.root-servers.net.
518400    IN    NS    k.root-servers.net.
518400    IN    NS    l.root-servers.net.
518400    IN    NS    m.root-servers.net.
```

Start and enable the DNS service

```bash
systemctl status named.service

systemctl restart named.service

systemctl enable named.service
```

Change the dns setting for the network to localhost

```bash
vim /etc/NetworkManager/system-connections/enp0s3.nmconnection

dns=127.0.0.1
```

Try resolving the names through configured server

```bash
dig google.com 127.0.0.1
```


## Step 2: Configure BIND

Now we are creating our Cache Only DNS Server to Master DNS Server.

Create the Backup of your main configuration file

```bash
cp -v /etc/named.conf /etc/named.conf.bak

vim /etc/named.conf
```

Update the named.conf file with the following configuration:

```bash
options {
 listen-on port 53 { 127.0.0.1; 192.168.43.61; };
 listen-on-v6 port 53 { ::1; };
 directory  "/var/named";
 dump-file  "/var/named/data/cache_dump.db";
 statistics-file "/var/named/data/named_stats.txt";
 memstatistics-file "/var/named/data/named_mem_stats.txt";
 secroots-file "/var/named/data/named.secroots";
 recursing-file "/var/named/data/named.recursing";
 allow-query     { localhost; 192.168.43.0/24; };

 recursion yes;

 // if the forwarder are available then will be able to resolve external
 forwarders {8.8.8.8; 8.8.4.4; };

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

### Keys Points

- **listen-on**: Specifies the IP addresses on which the server will listen.
- **allow-query**: Determines which hosts can query the DNS server.
- **forwarders**: Defines DNS servers to which unresolved queries will be forwarded.
- **recursion**: Enables recursive DNS queries.
- **zone "."**: Defines the root zone, which contains the default root DNS server references.


## Step 3: Create the Forward Zone file

### Set the host-name of the server as per the domain (armour.local)

```bash
vim /etc/hostname
```

```
ns1.armour.local
```

### Make the entry of the IP address and domain name

```
127.0.0.1   localhost ns1 ns1.armour.local
::1         localhost localhost6 ns1 ns1.armour.local
192.168.43.51 ns1 ns1.armour.local
```

### Making an entry of the ( armour.local ) zone file in the /etc/named.rfc1912.zones

```
zone "armour.local" IN {
 type master;
 file "forward.armour.local";
 allow-update { none; };
};
```

### Create the zone file with the same name provided while making the entry of the zone

```bash
vim /var/named/forward.armour.local
```

### Put this into the file, here you can create entry of all the DNS records needed

```
$TTL 1D
@       IN SOA  ns1.armour.local. root.armour.local. (
                                        0001    ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@                       IN      NS      ns1.armour.local.
ns1                     IN      A       192.168.43.51
armour.local.          IN      A       192.168.43.51
www                     IN      CNAME   armour.local.
router                  IN      A       192.168.43.1
mainpc                  IN      A       192.168.43.87
debian                  IN      A       192.168.43.41
```

### Key points:

- The SOA (Start of Authority) record specifies the authoritative server and email address.
- The NS (Name Server) record defines the authoritative nameservers for the domain.
- The A record maps a domain or subdomain to an IP address.

### Change the ownership of the file to named:named

```bash
chown -Rv named:named /var/named/forward.armour.local
```

### Restart the named service

```bash
systemctl restart named.service
```

## Step 4: Create the Forward Zone file

### Make an entry of the ( armour.local ) zone file in the /etc/named.rfc1912.zones

```bash
zone "43.168.192.in-addr.arpa" IN {
 type master;
 file "reverse.armour.local";
 allow-update { none; };
};
```

### Create the zone file with the same name provided while making the entry of the zone

```bash
vim /var/named/reverse.armour.local
```

```bash
$TTL 1D
@ IN SOA ns1.armour.local. root.armour.local. (
     001 ; serial
     1D ; refresh
     1H ; retry
     1W ; expire
     3H ) ; minimum
@   IN  NS ns1.armour.local.
@   IN  PTR ns1.armour.local.
31   IN  PTR ns1.armour.local.
87   IN  PTR mainpc.armour.local.
41   IN   PTR debian.armour.local.
100.43.168.192.in-addr.arpa. IN  PTR mail.armour.local.
```

### Key Points:
- The PTR (Pointer Record) record that maps an IP Address to a domain name.
## Step 5: Configure the Firewall

The CentOS system uses firewalld, you will need to allow DNS traffic ( port 53 ).

### Allow DNS through the firewall

```sh
firewall-cmd --zone=public --add-service=dns --permanent

firewall-cmd --reaload
```

### Check the firewall rules

```sh
firewall-cmd --list-all
```

## Step 6: Test the DNS Server

After Configuration, test whether the DNS Server is resolving the names correctly.

```sh
dig ns1.armour.local
```

It should be resolved through your DNS Server ( 127.0.0.1 or IP Address )

# Step 7: Configure DNS Clients

To make use of your DNS server, configure client machines to point to your server.

## Configure Client Machines
On each client machine, set the DNS server IP to the CentOS DNS server (e.g., 192.168.1.10).

```bash
vim /etc/resolv.conf
```

### Add the following line to the file:
```bash
nameserver 192.168.1.10
```

# Master and Slave DNS Server

Here we already have configured the Primary DNS (Master DNS Server). Now, we just have to configure another DNS Server, which will work as a Slave DNS.

## Install Bind on the Slave Server
```bash
yum install bind*
```

## Configure the Slave Server
- Configure the Slave server the same as the Master Server.
- Allow and add the zone transfer configuration in the zone entry in the zone file of the Master Server.

### Master Server Zone Configuration
```bash
zone "armour.local" IN {
    type master;
    file "forward.armour.local";
    allow-update { none; };
    allow-transfer { 192.168.43.61; };  // for the slave DNS server
};
```

## Create NS and A Records in the Master's Zone File
```bash
$TTL 1D
@       IN SOA  ns1.armour.local. root.armour.local. (
                            0001    ; serial
                            1D      ; refresh
                            1H      ; retry
                            1W      ; expire
                            3H )    ; minimum
@       IN      NS      ns1.armour.local.
@       IN      NS      ns2.armour.local.
ns1     IN      A       192.168.43.51
ns2     IN      A       192.168.43.61
armour.local.  IN      A       192.168.43.51
www     IN      CNAME   armour.local.
router  IN      A       192.168.43.1
mainpc  IN      A       192.168.43.87
debian  IN      A       192.168.43.41
```

## Check the Configuration File and Restart the Server
```bash
named-checkzone /var/named/filename

systemctl restart named.service
systemctl enable named.service
```

## Set the Hostname of the Slave Server
```bash
127.0.0.1   localhost ns2 ns2.armour.local
::1         localhost localhost6 ns2 ns2.armour.local
192.168.43.61 ns2 ns2.armour.local
```

## Configure the Firewall
```bash
firewall-cmd --zone=public --add-service=dns --permanent
```

## Create the Zone Entries in `/etc/named.rfc1912.zones`
```bash
zone "armour.local" IN {
    type slave;
    masters { 192.168.43.51; };
    file "slaves/forward.armour.local";
};
```

Restart the DNS service:
```bash
systemctl restart named.service
```

After restarting the server, the files will be automatically created in `/var/named/`.

## Test the DNS Server
```bash
dig ns1.armour.local
```

**Note:** Whenever a change happens in DNS records in the Master DNS server, the serial number needs to be updated to reflect the changes in the Slave Server.






