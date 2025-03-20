# CCNA Enterprise Lab 

This lab simulates a network with redundant links, utilizing PVSTP (Per-VLAN Spanning Tree Protocol) to prevent loops.\
Router R1 acts as a jump host, providing SSH access to other devices.\
Router R2 delivers DHCP, DNS, and TFTP services.\ 
Switches CS1, AS1, and AS2 are configured with VLANs, OSPF, PortFast, and Port Security.\
A redundant link between AS1 and AS2 is added to demonstrate PVSTP functionality.

Lab Topology:

    R1 (Jump Host, ROAS)
    R2 (DHCP, DNS, TFTP Server)
    CS1 (Core Switch)
    AS1, AS2 (Access Switches)
    5x PCs (VLAN 10, 20, 30, 40)
    1x Printer (VLAN 60)

Addressing:

    VLAN 10 (IT): 192.168.10.0/24
    VLAN 20 (Sales): 192.168.20.0/24
    VLAN 30 (Marketing): 192.168.30.0/24
    VLAN 40 (HR): 192.168.40.0/24
    VLAN 60 (Printer): 192.168.60.0/24
    R1 (LAN): 192.168.10.1, 20.1, 30.1, 40.1, 60.1 /24
    R1 (WAN): 203.0.113.1/30
    R2 (WAN): 203.0.113.2/30
    PCs: 192.168.10.10, 20.10, 30.10, 40.10 /24
    Printer: 192.168.60.10/24

Configuration:
```
R1 (Jump Host):
enable
configure terminal
hostname R1
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
service password-encryption
ip domain-name example.com
crypto key generate rsa general-keys modulus 2048
username admin privilege 15 secret cisco
line vty 0 15
transport input ssh
login local
ip ssh version 2

interface GigabitEthernet0/0
no shutdown
interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
interface GigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
interface GigabitEthernet0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
interface GigabitEthernet0/0.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
interface GigabitEthernet0/0.60
encapsulation dot1Q 60
ip address 192.168.60.1 255.255.255.0
interface GigabitEthernet0/1
ip address 203.0.113.1 255.255.255.252
no shutdown

router ospf 1
network 192.168.10.0 0.0.0.255 area 10
network 192.168.20.0 0.0.0.255 area 20
network 192.168.30.0 0.0.0.255 area 30
network 192.168.40.0 0.0.0.255 area 40
network 192.168.60.0 0.0.0.255 area 60
network 203.0.113.0 0.0.0.3 area 0

ip access-list standard SSH_ACCESS
permit 192.168.0.0 0.0.255.255
line vty 0 15
access-class SSH_ACCESS in
```

R2 (DHCP, DNS, TFTP Server):
```
enable
configure terminal
hostname R2
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
service password-encryption

interface GigabitEthernet0/0
ip address 203.0.113.2 255.255.255.252
no shutdown

ip dhcp pool IT
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 203.0.113.2
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp pool Sales
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 203.0.113.2
ip dhcp excluded-address 192.168.20.1 192.168.20.9
ip dhcp pool Marketing
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 203.0.113.2
ip dhcp excluded-address 192.168.30.1 192.168.30.9
ip dhcp pool HR
network 192.168.40.0 255.255.255.0
default-router 192.168.40.1
dns-server 203.0.113.2
ip dhcp excluded-address 192.168.40.1 192.168.40.9
ip dhcp pool Printer
network 192.168.60.0 255.255.255.0
default-router 192.168.60.1
dns-server 203.0.113.2
ip dhcp excluded-address 192.168.60.1 192.168.60.9

ip dns server
zone example.com
name R1.example.com address 192.168.10.1
name CS1.example.com address 192.168.10.2
name AS1.example.com address 192.168.10.3
name AS2.example.com address 192.168.10.4
exit

tftp-server flash:R1-config.txt
tftp-server flash:CS1-config.txt
tftp-server flash:AS1-config.txt
tftp-server flash:AS2-config.txt

line console 0
exec-timeout 0 0
password konzola
login local
line vty 0 15
exec-timeout 0 0
password telnet
login local
```
CS1 (Core Switch):
```
enable
configure terminal
hostname CS1
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
service password-encryption
ip domain-name example.com
crypto key generate rsa general-keys modulus 2048
username admin privilege 15 secret cisco
line vty 0 15
transport input ssh
login local
ip ssh version 2
spanning-tree mode pvst

vlan 10
name IT
vlan 20
name Sales
vlan 30
name Marketing
vlan 40
name HR
vlan 60
name Printer
exit

interface GigabitEthernet0/1
switchport mode trunk
interface GigabitEthernet0/2
switchport mode trunk
interface GigabitEthernet0/3
switchport mode trunk

ip access-list standard SSH_ACCESS
permit 192.168.0.0 0.0.255.255
line vty 0 15
access-class SSH_ACCESS in

line console 0
exec-timeout 0 0
password konzola
login local
line vty 0 15
exec-timeout 0 0
password telnet
login local
```
AS1 (Access Switch 1):
```
enable
configure terminal
hostname AS1
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
service password-encryption
ip domain-name example.com
crypto key generate rsa general-keys modulus 2048
username admin privilege 15 secret cisco
line vty 0 15
transport input ssh
login local
ip ssh version 2
spanning-tree mode pvst

vlan 10
name IT
vlan 20
name Sales
vlan 30
name Marketing
vlan 40
name HR
vlan 60
name Printer
exit

interface GigabitEthernet0/1
switchport mode trunk

interface GigabitEthernet0/2 (Spojenie s AS2)
switchport mode trunk

interface FastEthernet0/1
switchport mode access
switchport access vlan 10
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/2
switchport mode access
switchport access vlan 20
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/3
switchport mode access
switchport access vlan 30
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/4
switchport mode access
switchport access vlan 40
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/5
switchport mode access
switchport access vlan 60
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

ip access-list standard SSH_ACCESS
permit 192.168.0.0 0.0.255.255
line vty 0 15
access-class SSH_ACCESS in
```

### AS2 (Access Switch 2):

```enable
configure terminal
hostname AS2
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
service password-encryption
ip domain-name example.com
crypto key generate rsa general-keys modulus 2048
username admin privilege 15 secret cisco
line vty 0 15
transport input ssh
login local
ip ssh version 2
spanning-tree mode pvst

vlan 10
name IT
vlan 20
name Sales
vlan 30
name Marketing
vlan 40
name HR
vlan 60
name Printer
exit

interface GigabitEthernet0/1
switchport mode trunk

interface GigabitEthernet0/2 (Spojenie s AS1)
switchport mode trunk

interface FastEthernet0/1
switchport mode access
switchport access vlan 10
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/2
switchport mode access
switchport access vlan 20
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/3
switchport mode access
switchport access vlan 30
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/4
switchport mode access
switchport access vlan 40
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

interface FastEthernet0/5
switchport mode access
switchport access vlan 60
switchport port-security
switchport port-security maximum 1
switchport port-security violation restrict
switchport port-security mac-address sticky
spanning-tree portfast

ip access-list standard SSH_ACCESS
permit 192.168.0.0 0.0.255.255
line vty 0 15
access-class SSH_ACCESS in
```

**Verification Steps:**

+ **OSPF Verification:**
        On R1, use show ip ospf neighbor and verify that +how ip route ospf and verify that OSPF routes are in the routing table.
+ **VLAN and Trunk Verification:**
        On CS1, AS1, and AS2, use show vlan brief and verify that VLANs are created and assigned to the correct ports.
        On CS1, AS1, and AS2, use show interfaces trunk and verify that trunk ports are correctly configured.
+ **DHCP:**
        On PCs in VLAN 10, 20, 30, 40, and the printer, verify that they receive IP addresses from the DHCP server (R2).
        On R2, use show ip dhcp binding and verify that IP addresses are assigned.
+ **DNS:**
        On a PC, use nslookup R1.example.com and verify that it resolves to the correct IP address.
+ **SSH Access:**
        From R1, connect via SSH to CS1, AS1, and AS2 and verify that access is successful.
        Verify that SSH access is denied from any other device.
+ **Port Security:**
        On AS1 and AS2, use show port-security interface <interface> and verify that MAC addresses are learned and that Port Security is active.
        Attempt to connect another device to a port with Port Security and verify that access is blocked.
+ **Printer Access:**
        From PCs in VLAN 10, 20, 30, 40, ping the printer (192.168.60.10) and verify that access is possible.
        From the printer, attempt to ping PCs in VLAN 10, 20, 30, 40 and verify that access is blocked.
+ **HTTP Access (HR VLAN):**
        From a PC in VLAN 40, attempt to open a web page (e.g., 8.8.8.8) and verify that access is blocked.
        From PCs in other VLANs, verify that HTTP access is allowed.
+ **TFTP:**
        From R1, copy the CS1 configuration using copy running-config tftp: and enter the IP address of R2.
        On R2, verify that the configuration is saved to flash memory.
+ **Console and Telnet Access:**
        Verify that direct console and Telnet access to R2, CS1, AS1, and AS2 is blocked.
        Samozrejme, tu sú kroky overenia pre PVSTP v angličtine:

+ **PVSTP Verification:**

+ **Check Spanning-Tree Status:**
        On CS1, AS1, and AS2, use the command show spanning-tree vlan <VLAN-ID> (replace <VLAN-ID> with the actual VLAN number) to verify the PVSTP status for each VLAN.
        This command will display information about the root bridge, port roles (root, designated, alternate, blocked), and port states (forwarding, blocking).