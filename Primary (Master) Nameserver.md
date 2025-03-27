# Primary DNS Server (Master) Configuration  

## Setting the Hostname for the DNS Server  
```bash
nano /etc/hostname
```
### Example:
```
ns1.nikhil.local
```

## Editing the Host File as per the Zone  
```bash
nano /etc/hosts
```
### Configuration Data (Optional):
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.112.145 ns1 ns1.nikhil.local
```

## Removing Recursion Forwarders from Configuration File  
```bash
nano /etc/named.conf
```
### Remove Forwarders:
```bash
         forwarders {
                8.8.8.8;
                8.8.4.4;
        };
```

## Editing the DNS Zone Configuration  
```bash
nano /etc/named.rfc1912.zones
```
### Configuration Data:
```
zone "nikhil.local" IN {
        type master;
        file "forward.nikhil.local";
        allow-update { none; };
};
```

## Main DNS Zone File Configuration  
```bash
// named.rfc1912.zones:
//
// Provided by Red Hat caching-nameserver package
//
// ISC BIND named zone configuration for zones recommended by
// RFC 1912 section 4.1 : localhost TLDs and address zones
// and https://tools.ietf.org/html/rfc6303
// (c)2007 R W Franks
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// Note: empty-zones-enable yes; option is default.
// If private ranges should be forwarded, add
// disable-empty-zone "."; into options
//

zone "localhost.localdomain" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};

zone "localhost" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
        type master;
        file "named.loopback";
        allow-update { none; };
};

zone "1.0.0.127.in-addr.arpa" IN {
        type master;
        file "named.loopback";
        allow-update { none; };
};

zone "0.in-addr.arpa" IN {
        type master;
        file "named.empty";
        allow-update { none; };
};

zone "nikhil.local" IN {
        type master;
        file "forward.nikhil.local";
        allow-update { none; };
};
```

## Creating the Forward Zone File  
### Copy Example Zone File and Modify:
```bash
cd /var/named/
cp -v named.localhost forward.nikhil.local
```

## Editing the Forward Zone File  
```bash
nano /var/named/forward.nikhil.local
```
### Blank File:
```
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
```

## Final Forward Zone Configuration  
```
$TTL 1D
@       IN SOA  ns1.nikhil.local. root.nikhil.local (
                                        20250324        ; serial
                                        3600            ; refresh
                                        1800            ; retry
                                        604800          ; expire
                                        86400 )         ; minimum
@               IN              NS      ns1.nikhil.local.
ns1             IN              A       192.168.112.145
nikhil.local.   IN              A       192.168.112.145
www             IN              CNAME   nikhil.local.
router          IN              A       192.168.112.98
emp1            IN              A       192.168.112.200
emp2            IN              A       192.168.112.201

; Mail exchange
@               IN              MX      10 mail.nikhil.local.
mail            IN              A       192.168.112.150

; Text record for SPE
@               IN              TXT     "v=spr1 mx a ~all"

; Additional Services
ftp             IN              CNAME   nikhil.local.
dev             IN              A       192.168.112.100
fileserver      IN              A       192.168.112.105
db              IN              A       192.168.112.110
test            IN              A       192.168.112.115
vpn             IN              A       192.168.112.120
git             IN              A       192.168.112.125
webapp          IN              A       192.168.112.130
logs            IN              A       192.168.112.135
```

## Setting Permissions  
```bash
chgrp named /var/named/forward.nikhil.local
chmod 640 /var/named/forward.nikhil.local
```

## Restarting BIND Service  
```bash
systemctl restart named
```

## Checking DNS Resolution  
```bash
dig nikhil.local
```

## Validating Configuration  
```bash
named-checkconf
named-checkconf /etc/named.conf
named-checkconf /etc/named.rfc1912.zones
named-checkconf /etc/named.root.key
```

## Verifying Forward Zone  
```bash
named-checkzone nikhil1.local /var/named/forward.nikhil.local
```
### Output:
```
zone nikhil1.local/IN: loaded serial 20250324
OK
```

## Testing Name Resolution  
```bash
dig @192.168.112.145 nikhil.local
dig @192.168.112.145 emp1.nikhil.local
dig @192.168.112.145 ftp.nikhil.local
```
