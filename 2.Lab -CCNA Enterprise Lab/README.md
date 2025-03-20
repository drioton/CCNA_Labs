# CCNA Enterprise Lab (Advanced) – Full Configuration
This lab is designed to simulate an enterprise network environment, implementing advanced network configurations and security best practices.\
It covers VLAN segmentation, OSPF multi-area routing, SSH security, ACLs, spanning tree, and more.

## Key Features & Implementations:
✅ 5 VLANs (Management, Sales, IT, HR, Printer)\
✅ SSH-enabled devices (Secure remote access on all switches and routers)\
✅ Router with DNS, DHCP, SSH, and excluded addresses\
✅ PVST (Per-VLAN Spanning Tree) for redundancy and loop prevention\
✅ 3 Switches (1 Core + 2 Access)\
✅ Password encryption for enhanced security\
✅ OSPF with 5 areas for efficient inter-VLAN routing\
✅ PortFast and Port Security on access ports to prevent loops & unauthorized access\
✅ Access Control Lists (ACLs)

VLAN 60 (Printers) isolated from all other VLANs\
VLAN 40 (HR) restricted from HTTP access\
✅ TFTP Server for automated configuration backups\

**Lab Setup:** \
Router (R1) – Manages VLANs, OSPF, DHCP, and ACLs\
Switches (Core + Access) – VLAN implementation, trunking, security policies\
End Devices – Workstations, Printers, Servers\

This configuration ensures security, scalability, and efficient traffic management in a simulated enterprise network.