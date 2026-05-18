
# 1. Security Testing and Attack Simulation

This section demonstrates how the firewall, ACL policies, and port security configurations respond to unauthorised access attempts.

---

## 2. Attack Scenario 1: Guest attempting access to Inside Network

### Description
A device in the Guest network (192.168.50.0/24) attempts to access internal systems in the Inside network.

### Expected Result
- Traffic should be blocked by firewall ACL rules
- No response should be received from internal devices

### Failed Telnet, SSH, and FTP to Inside
<img src="./images/ATTACKfailedTelnetSSHFTP.png" width="500" height="300">

### Failed HTTP to Inside
<img src="/images/ATTACKfailedHTTPinside.png" width="550" height="250">

### Failed Ping Inside
<img src="/images/ATTACKfailedPingInside.png" width="500" height="200">

---

## 2. Attack Scenario 2: Rogue laptop connected to SW1

### Description
An intruder disconnects a PC from interface FastEthernet0/18 (HR) and attempts to access SW1.

### Expected Result
- Failed ping to SW1

### Failed Ping 192.168.1.10
<img src="/images/ATTACKrogueLaptop.png" width="450" height="400">

---

## 3. Attack Scenario 3: Successful Web Access to DMZ

### Description
External or guest user accesses the public web server in the DMZ.

### Expected Result
- HTTP access allowed
- No access to internal systems

### Successful Outside HTTP
<img src="/images/ATTACKsuccessfullHTTP.png" width="450" height="300">

### Failed Ping
<img src="/images/ATTACKfailedPingDMZ.png" width="450" height="300">

---

## 5. Security Conclusion

The security configuration successfully enforces:
- Network segmentation
- Access control between zones
- Controlled exposure of DMZ services
- Isolation of guest network
- Port security