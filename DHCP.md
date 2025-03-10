

## **DHCP Server Configuration Guide**

---

### **1. Install DHCP Server**
1. **Remove existing DHCP server (if any):**
   ```bash
   yum remove dhcp-server
   ```
   - Backup files are saved as:
     - `/var/lib/dhcpd/dhcpd.leases` → `/var/lib/dhcpd/dhcpd.leases.rpmsave`
     - `/etc/dhcp/dhcpd.conf` → `/etc/dhcp/dhcpd.conf.rpmsave`

2. **Clear YUM cache and install DHCP server:**
   ```bash
   yum remove makecache
   yum makecache
   yum install dhcp-server
   ```

---

### **2. Configure DHCP Server**
1. **Edit the DHCP configuration file:**
   ```bash
   nano /etc/dhcp/dhcpd.conf
   ```

2. **Basic DHCP configuration example:**
   ```bash
   authoritative;  # This server is the official DHCP server for the network

   # Define a subnet
   subnet 192.168.1.0 netmask 255.255.255.0 {
       range 192.168.1.50 192.168.1.250;  # Range of IPs to assign
       option routers 192.168.1.1;       # Default gateway
       option domain-name-servers 8.8.8.8, 8.8.4.4;  # DNS servers
       option broadcast-address 192.168.1.255;  # Broadcast address
       default-lease-time 600;           # Default lease time in seconds
       max-lease-time 7200;              # Maximum lease time in seconds
   }
   ```

3. **Save and exit:**
   - Press `Ctrl + O` to save.
   - Press `Enter` to confirm.
   - Press `Ctrl + X` to exit.

---

### **3. Start and Enable DHCP Service**
1. **Start the DHCP service:**
   ```bash
   systemctl start dhcpd
   ```

2. **Enable DHCP to start on boot:**
   ```bash
   systemctl enable dhcpd
   ```

3. **Check the status of the DHCP service:**
   ```bash
   systemctl status dhcpd
   ```
   - If the service fails, check the configuration file for errors.

---

### **4. Allow DHCP Port in Firewall**
1. **Check the DHCP port (UDP 67):**
   ```bash
   netstat -nltup | grep dhcpd
   ```

2. **Allow the DHCP port in iptables:**
   ```bash
   iptables -I INPUT 5 -p UDP --dport 67 -j ACCEPT
   ```

3. **Save and restart iptables:**
   ```bash
   service iptables save
   systemctl restart iptables
   ```

4. **Verify the rule is added:**
   ```bash
   iptables -n -L INPUT
   ```

---

### **5. Assign Static IP to a Client**
1. **Find the client's MAC address:**
   - Check the leases file:
     ```bash
     cat /var/lib/dhcpd/dhcpd.leases
     ```
   - Look for the `hardware ethernet` field.

2. **Assign a static IP in the DHCP configuration:**
   ```bash
   host win7 {
       hardware ethernet 08:00:27:d2:ae:8c;  # Client's MAC address
       fixed-address 192.168.1.51;           # Static IP to assign
   }
   ```

3. **Restart the DHCP service:**
   ```bash
   systemctl restart dhcpd
   ```

---

### **6. Block Clients from Receiving IPs**
1. **Deny IP assignment to specific clients:**
   ```bash
   host win7 {
       hardware ethernet 08:00:27:d2:ae:8c;  # Client's MAC address
       deny booting;                         # Block this client
   }
   ```

2. **Restart the DHCP service:**
   ```bash
   systemctl restart dhcpd
   ```

---

### **7. Allow or Deny Unknown Clients**
1. **Deny IP assignment to unknown clients:**
   ```bash
   deny unknown-clients;
   ```

2. **Allow unknown clients in a specific range:**
   ```bash
   pool {
       range 192.168.1.201 192.168.1.250;
       allow unknown-clients;
   }
   ```

3. **Allow known clients in a specific range:**
   ```bash
   pool {
       range 192.168.1.155 192.168.1.200;
       allow known-clients;
   }
   ```

4. **Restart the DHCP service:**
   ```bash
   systemctl restart dhcpd
   ```

---

### **8. Verify DHCP Leases**
1. **Check active leases:**
   ```bash
   cat /var/lib/dhcpd/dhcpd.leases
   ```
   - This file contains details of all leased IPs, including MAC addresses and lease times.

---

### **9. Troubleshooting**
1. **Check DHCP service logs:**
   ```bash
   journalctl -u dhcpd
   ```

2. **Verify configuration file syntax:**
   ```bash
   dhcpd -t
   ```
   - This checks for errors in the configuration file.

3. **Restart DHCP service after changes:**
   ```bash
   systemctl restart dhcpd
   ```

---

### **10. Example Configuration File**
Here’s a complete example of a DHCP configuration file:
```bash
authoritative;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.50 192.168.1.250;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    option broadcast-address 192.168.1.255;
    default-lease-time 600;
    max-lease-time 7200;

    # Static IP for a specific client
    host win7 {
        hardware ethernet 08:00:27:d2:ae:8c;
        fixed-address 192.168.1.51;
    }

    # Block a specific client
    host win7-2 {
        hardware ethernet 08:00:27:d2:ae:9c;
        deny booting;
    }

    # Allow unknown clients in a specific range
    pool {
        range 192.168.1.201 192.168.1.250;
        allow unknown-clients;
    }

    # Allow known clients in a specific range
    pool {
        range 192.168.1.155 192.168.1.200;
        allow known-clients;
    }
}
```

---

### **Summary**
- **Install DHCP:** `yum install dhcp-server`
- **Configure DHCP:** Edit `/etc/dhcp/dhcpd.conf`
- **Start DHCP:** `systemctl start dhcpd`
- **Enable DHCP:** `systemctl enable dhcpd`
- **Allow DHCP port in firewall:** `iptables -I INPUT 5 -p UDP --dport 67 -j ACCEPT`
- **Assign static IP:** Use `host` block in `dhcpd.conf`
- **Block clients:** Use `deny booting` in `dhcpd.conf`
- **Allow/Deny unknown clients:** Use `allow unknown-clients` or `deny unknown-clients`
