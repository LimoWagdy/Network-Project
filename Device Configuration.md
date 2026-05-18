# 1. Network Device Configurations (CLI)

This section contains all configuration commands used for routers, switches, and firewall devices in the network.

---

## 1. Router Configurations
### 1.1 R1 Internal
#### Basic Config
```bash
clock set 22:56:00 12 May 2026

hostname R1
ip domain-name limo.local

global password !l1m3o9273@

username limo password @limOnada!$4385
```

#### Interface Setup
```bash
interface gig0/0
ip address 10.0.0.1 255.255.255.0
no shutdown
 
interface gig0/1.5
encapsulation dot1Q 5
ip address 192.168.2.1 255.255.255.0
no shutdown

interface gig0/1.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
no shutdown

int gig0/2.5
encapsulation dot1Q 5
ip address 192.168.1.1 255.255.255.0
no shut

int gig0/2.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
ip helper-address 192.168.10.11
no shut

int gig0/2.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
ip helper-address 192.168.10.11
no shut

int gig0/2.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
ip helper-address 192.168.10.11
no shut
```


#### SSH from admin
```
ip ssh version 2

crypto key generate rsa
2048

access-list 10 permit host 192.168.20.11

line vty 0
transport input ssh
login local
access-class 10 in
```

#### VLAN ACLs
Deny rules for the IT department were placed on the firewall instead of the router, since router ACLs don’t inspect traffic or keep track of connection state. The `SERV-IN` ACL also includes explicit permits for DHCP and return IT traffic because router ACLs are stateless and don’t automatically allow reply traffic.

HTTP was also used instead of HTTPS because Packet Tracer doesn’t always handle HTTPS reliably. In a real setup, HTTPS (TCP port 443) should be used instead.

```
ip access-list extended IT-IN
	permit ip 192.168.20.0 0.0.0.255 any
	permit udp any any eq 67

int gig0/2.20
ip access-group IT-IN in

ip access-list extended FIN-IN
	permit ip 192.168.30.0 0.0.0.255 192.168.30.0 0.0.0.255
	permit tcp 192.168.30.0 0.0.0.255 any eq 80
	permit udp 192.168.30.0 0.0.0.255 any eq 53
	permit udp any any eq 67  
	deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255

int gig0/2.30
ip access-group FIN-IN in

ip access-list extended HR-IN
	permit ip 192.168.40.0 0.0.0.255 192.168.40.0 0.0.0.255
	permit tcp 192.168.40.0 0.0.0.255 any eq 80
	permit udp 192.168.40.0 0.0.0.255 any eq 53
	permit udp any any eq 67
	deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255

int gig0/2.40
ip access-group HR-IN in

ip access-list extended SERV-IN
	permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
	permit udp any any eq 67
	permit udp any any eq 68
	deny ip any any

int gig0/1.10
ip access-group SERV-IN in
```

### 1.2 R2 External
```
int gig0/0
ip address 209.0.113.1
no shutdown

int gig0/1
ip address 8.8.8.1
no shutdown
```

## 2.  Switch Configurations
This section contains the configuration for all switches in the network. Due to limitations with the firewall not supporting sub-interfaces in this setup, SSH and logging were not configured on Switch 3 and Switch 4 (DMZ and Guest). These devices are not part of the management network.

### 2.1 Global Switch Configuration
All switches were configured with the following commands:
```
clock set 22:56:00 12 May 2026 (Relevant time)
hostname SW1 (SW2, SW3, SW4)

global password !l1m3o9273@
ip domain-name limo.local
username limo password @limOnada!$4385
```

### 2.2 SW1 Internal Users
#### VLAN Creation
```
vlan 5
name Management
ip address 192.168.1.10 255.255.255.0

vlan 20 
name IT

vlan 30 
name FIN

vlan 40 
name HR
```

#### Access Port Assignment
The following mapping was used to assign switch ports to different departments
- ports 1-8: IT
- ports 9-16: FIN
- ports 17-24: HR

```
int fa0/1-3
switchport mode access
switchport access vlan 20
no shut

int fa0/9-10
switchport mode access
switchport access vlan 30
no shut

int fa0/17-18
switchport mode access
switchport access vlan 40
no shut

```

#### Port Security
```
int range fa0/24
switch port-security maximum 1
switchport port-security mac-address sticky
switch port-security violation shutdown
spanning-tree portfast
spanning-tree bpduguard enable

int range fa0/4-8
shutdown

int range fa0/11-16
shutdown

int range fa0/19-24
shutdown
```

### Trunk Configuration to R1

```
interface gig0/1
switchport trunk allowed vlan 5,20,30,40
switchport mode trunk
```

#### SSH from admin
```
ip ssh version 2

crypto key generate rsa
2048

access-list 10 permit host 192.168.20.11

line vty 0 (one session for admin)
transport input ssh
login local
access-class 10 in

ip default-gateway 192.168.1.1

```

#### Syslog Config
```
service timestamps log datetime msec
logging buffered
logging host 192.168.10.14
```

### 2.3 SW2 Internal Servers
#### VLAN Creation
```
vlan 5 
name Management
ip address 192.168.2.10 255.255.255.0

vlan 10 
name Servers
```

#### Access Port Assignment
```
interface fa0/1  
switchport mode access  
switchport access vlan 5

interface fa0/1  
switchport mode access  
switchport access vlan 60
```

#### Port Security
```
int range fa0/24
switch port-security maximum 1
switchport port-security mac-address sticky
switch port-security violation shutdown
spanning-tree portfast
spanning-tree bpduguard enable

int range fa0/6-24
shutdown
```

### Trunk Configuration to R1

```
interface gig0/1
switchport trunk allowed vlan 5,10
switchport mode trunk
```

#### SSH from admin
```
ip ssh version 2

crypto key generate rsa
2048

access-list 10 permit host 192.168.20.11

line vty 0 (one session for admin)
transport input ssh
login local
access-class 10 in

ip default-gateway 192.168.2.1
```

#### Syslog Config
```
service timestamps log datetime msec
logging buffered
logging host 192.168.10.14
```

### 2.4 SW3 DMZ
#### VLAN Creation
```
vlan 60 
name DMZ
```

#### Access Port Assignment
```
interface fa0/1  
switchport mode access  
switchport access vlan 60

interface gig0/1  
switchport mode access  
switchport access vlan 60
```

#### Port Security
```
int range fa0/24
switch port-security maximum 1
switchport port-security mac-address sticky
switch port-security violation shutdown
spanning-tree portfast
spanning-tree bpduguard enable

int range fa0/2-24
shutdown
```

### 2.5 SW4 Guest
#### VLAN Creation
```
vlan 50 
name Guest
```

#### Access Port Assignment
```
int range fa0/1-2
switchport mode access
switchport access vlan 50
no shut

int gig0/1
switchport mode access
switchport access vlan 50
no shut
```

#### Port Security
```
int range fa0/1-24
switch port-security maximum 1
switchport port-security mac-address sticky
switch port-security violation shutdown
spanning-tree portfast
spanning-tree bpduguard enable

int range fa0/3-24
shutdown
```

## 3.  Firewall Configuration
Logging was not configured on the firewall as Packet Tracer does not fully support firewall logging features.
#### Basic Config
```bash
clock set 22:58:00 12 May 2026

hostname FW1
domain-name limo.local

global password !l1m0n$
username limo password @limOnada!$4385

```

#### SSH from admin
```
ssh 192.168.20.11 255.255.255.255 inside

aaa authentication ssh console LOCAL

no telnet 0.0.0.0 0.0.0.0 inside
no telnet 0.0.0.0 0.0.0.0 outside
```

#### Interface Setup
```bash
int gig1/1
ip address 192.168.60.1 255.255.255.0
security level 50
no shutdown

int gig1/2
ip address 10.0.0.2 255.255.255.252
security level 100
no shutdown

int gig1/3
ip address 192.168.50.1 255.255.255.0
security level 25
no shutdown

interface g0/0
nameif outside
ip address 203.0.113.2 255.255.255.0 (public ip)
```

#### Routing and NAT
```
route outside 0.0.0.0 0.0.0.0 203.0.113.1
route inside 192.168.0.0 255.255.0.0 10.0.0.1

object network inside
subnet 192.168.0.0 255.255.0.0
nat (inside,outside) dynamic interface
```

#### MPF
```
class-map inspection_default
match default-inspection-traffic

policy-map global_policy
class inspection_default
inspect icmp
inspect dns
inspect http

service-policy global_policy global
```

#### Zone ACL
Due to limitations in Packet Tracer’s ASA firewall simulation, stateful inspection is not fully implemented. In some cases, traffic allowed in one direction also required an explicit return rule on another interface for communication to work correctly.

For example, allowing outbound HTTP from the inside network may also require an additional inbound rule to permit the return traffic:

`access-list INSIDE-IN extended permit tcp 192.168.0.0 255.255.0.0 any eq 80`  
`access-list OUT-IN extended permit tcp any 192.168.0.0 255.255.0.0 eq 80`

To avoid making the ACLs overly permissive or unrealistic, and to prevent exposing internal networks in the simulation, some allow rules that would normally exist in a real deployment were left out.

ICMP was also used in some rules for testing and basic connectivity checks. In a real environment, this would typically be replaced or restricted in favour of secure management access such as SSH (TCP port 22).

Similarly, HTTP was used instead of HTTPS because Packet Tracer does not always handle HTTPS services consistently. In a real deployment, HTTPS (TCP port 443) would be used instead.

In a real-world setup, a properly stateful firewall would handle return traffic automatically for established connections, so these extra return rules would not be needed.

#### ACL Used
```
access-list INSIDE-IN extended permit icmp host 192.168.20.11 192.168.50.0 255.255.255.0
access-list INSIDE-IN extended permit icmp host 192.168.20.11 192.168.60.0 255.255.255.0

access-list OUT-IN extended permit tcp any host 192.168.60.11 eq 80
access-list OUT-IN deny ip any any

access-list DMZ-IN extended permit icmp 192.168.60.0 255.255.255.0 host 192.168.20.11
access-list DMZ-IN extended permit tcp host 192.168.60.11 any eq www

access-list GUEST-IN extended permit icmp any host 192.168.20.11
access-list GUEST-IN extended permit tcp any host 192.168.60.11 eq www

access-group INSIDE-IN in interface inside
access-group OUT-IN in interface outside
access-group DMZ-IN in interface dmz
access-group GUEST-IN in interface guest
```

#### 'Realistic' ACL
In a real-world environment where the firewall fully supports stateful packet inspection, and where HTTPS and SSH are used for secure communication, the ACL configuration would typically be writtin as follows:

```
access-list INSIDE-IN extended
	permit tcp host 192.168.20.11 192.168.50.0 255.255.255.0 eq 22
	permit tcp host 192.168.20.11 192.168.60.0 255.255.255.0 eq 22

	permit tcp 192.168.0.0 255.255.0.0 host 192.168.60.11 eq 80

	deny ip 192.168.0.0 255.255.0.0 192.168.50.0 255.255.255.0
	deny ip 192.168.0.0 255.255.0.0 192.168.60.0 255.255.255.0

	extended permit tcp 192.168.0.0 255.255.0.0 any eq 443
	extended permit udp 192.168.0.0 255.255.0.0 any eq domain
	
access-list GUEST-IN extended
	permit tcp 192.168.50.0 255.255.255.0 host 192.168.60.11 eq 443
	deny ip 192.168.50.0 255.255.255.0 192.168.0.0 255.255.0.0
	permit tcp 192.168.50.0 255.255.255.0 any eq 443
	permit udp 192.168.50.0 255.255.255.0 any eq 53

access-list DMZ-IN extended
	deny ip any any
```

## 4.  Server Configuration
Packet Tracer does not fully support SSH server functionality. As a result, SSH was only configured on supported network infrastructure devices (routers and switches).

### 4.1 DHCP Server

#### IP Config
IPv4 Address: 192.168.10.11
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
#### DHCP Config

##### IT
Default Gateway: 192.168.20.1
DNS: 192.168.10.12
192.168.20.11
255.255.255.0
Max users 8 (10 for guest)

##### FIN
Default Gateway: 192.168.30.1
DNS: 192.168.10.12
192.168.30.11
255.255.255.0
Max users 8 (10 for guest)
##### HR
Default Gateway: 192.168.40.1
DNS: 192.168.10.12
192.168.40.11
255.255.255.0
Max users 8

#### Admin DHCP Request
<img src="/images/adminDHCPrequest.png" width="500" height="250">

### 4.2 DNS Server
#### IP Config
IPv4 Address: 192.168.10.12
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1

#### DNS Config
Name: limo.com
Address: 192.168.60.11

#### DNS Lookup
<img src="/images/DNSlookup.png" width="450" height="300">

### 4.3 Email Server
#### IP Config
IPv4 Address: 192.168.10.13
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1

#### Email Config
- Domain name: gmail.com
- 2 users for simplicity:
	- user limo password @limOnada!$4385
	- user sara password saraleboss!

#### Limo Send (192.168.20.11)
<img src="/images/EMAILSend.png" width="500" height="175">

#### Sara Receive (192.168.40.12) 
<img src="/images/EMAILreceive.png" width="500" height="175">

### 4.4 Syslog server
Due to Packet Tracer limitations, hostnames could not be resolved correctly in the syslog server. As a result, logs display device IP addresses instead of hostnames.

#### IP Config
IPv4 Address: 192.168.10.14
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1

The rest is configured on the switches and router.

#### Shutting Down Ports on R1, SW1, SW2
<img src="/images/SYSLOG.png" width="250" height="350">

### 4.5 FTP Server
#### IP Config
IPv4 Address: 192.168.10.15
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1

#### FTP Config
- 2 users for simplicity:
	admin limo password @limOnada!$4385 | RWDNL
	user essam password essamleboss!2004 | RWL

#### Limo Put (192.168.20.11)
<img src="/images/FTPput.png" width="400" height="300">

#### Essam Get (192.168.20.12)
<img src="/images/FTPget.png" width="400" height="300">

## 5. Other Devices

#### Outside PC (Simulated Public DNS)
IPv4 Address: 8.8.8.8
Subnet Mask: 8.8.8.1
Default Gateway: 8.8.8.1

#### AP1
- WPA2-PSK Pass Phrase: liMoU$inE4178
