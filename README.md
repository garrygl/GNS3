# The following Procedure is a GNS3 Appliance Upgrade from Ubuntu 14.x to 18.x for Migrating Appliances and Projects 
- Upgraded an ESXi VM appliance from Ubuntu 14.x/gns3 2.2.7 to the newer Ubuntu 18.04/gns3 2.2.7 and newer VM. I verfied this procedure worked for migrating all appliances and projects for the old vm to the new ESXi VM. 
- This procedure may not work for VM for Workstation/Fusion since project configuration files are stored on the client storage and not the server.
- Reference - **[Link to GNS3 Post (HTML format)](https://gns3.com/upgrade-ubuntu-14-x-gns3-2-2-7-t)

## Backup GNS3 Projects and Appliances and Other Content
1. cd/opt - provides a common storage volume with adequate storage space
2. Backup vm server projects and images
    sudo tar -zcvf opt-gns3-2_7.tar /opt/gns3
### Backup symbols and appliances in gns3 server folders
1. sudo tar -zcvf gns3-home2_7.tar /home/gns3
2. use ftp client on vm to (binary) put/upload .tar files to shared storage

## Restore GSN3 Content
### Restore the two .tar files to original folders on new ESXi vm server after installing an ftp client on this 2.2.7 vm
1. Use ftp client to get .tar files from storage and (binary) put into /opt on new gns3 VM 
2. cd/ establishes the root folder for restore overlay

## Restore vm server projects and images
1. sudo tar -zxvf /opt/opt-gns3-2_7.tar
2. restore symbols and appliances in gns3 server folders
  sudo tar -zxvf /opt/gns3-home2_7.tar
  reboot vm and connect the GNS3 2.7 client to new ESXi vm
# End-of-Procedure

# GSN3 LAB NOTES
## Operational Notes and Information for GSN3 Simulation Lab 1 
- Reference - **[Link to Network-Lab.pdf (HTML format)](https://github.com/garrygl/GNS3/blob/20b25805cb25cc5977216e78a9a4a0c04696f3e3/Network-Lab.pdf)

- Webterm is a docker container and sometimse will not copy
change the ip address to 192.168.1.10 - .2 is OOB PAN

- Use bgp local pref to prefer CENIC over Concast for Azure
if CENIC is not availble, prefer Comcast

- OSPF Type 2 route redist into OSPF on SPWAN-1 to avoid having 
the metric change from link-to-link. EpressRoute is prefered
The static 10.30.0.0/19 on the PAN is inserted into the routing
table if the vWAN peer is reachable. The more specific
 /24 routes from the equinix peer are inserted into ospf type2 
from bgp to ospf redistribution from SPWAN-1

- address-family ipv4
  bgp redistribute-internal
## PAN Policy
- PAN Poilcy Based forwarding rule to Azure Cohesity
  PAN 10.30.0.0/19 is static and only inserted if tunnel peer is operational. ExpressRoute is prefered over vWAN VPN

* Show cry isa sa
* show cry ips as
* debug cry ipsec 255
* look for MM_Active 
* Must start NX9K firest, wait for boot followed by the CSR1K's
* Start PAN after 1K's wait 15mi.
* Start others one at a time
* My require 30m to boot lab

## PAN 
- Wait for boot at least 5 minutes
for new cofigs on pan run "delete deviceconfig system type dhcp-client"
set deviceconfig system ip-address 192.168.1.2 netmask 255.255.255.0
Webconsole-persistent select live-usb at boot screen

# End of Lab 1 Notes

# Operational Notes and Information for GSN3 Simulation Lab 2 
- Reference - **[Link to Cisco_VXLAN_APIC-EM_Stealthwatch_Prime-Infrastructure_Networking-Lab.pdf (HTML format)](https://github.com/garrygl/GNS3/blob/20b25805cb25cc5977216e78a9a4a0c04696f3e3/Cisco_VXLAN_APIC-EM_Stealthwatch_Prime-Infrastructure_Networking-Lab.pdf)


- VXLAN Tunnel Endpoint (VTEP): Map tenants’ end devices to VXLAN 
  segments. Used to perform VXLAN encapsulation/de-encapsulation.
- Virtual Network Identifier (VNI): identify a VXLAN segment. It hast up
to 224 IDs theoretically giving us 16,777,216 segments. (Valid VNI values are from 4096 to 16777215). Each segment can transport 
802.1q-encapsulated packets, theoretically giving us 212 or 4096 VLANs over a single VNI.

- Network Virtualization Endpoint or Network Virtualization Edge (NVE): overlay interface configured in Cisco devices to define a VTEP

# Usefull Show Commands
- List 
  show run | i ip pim|interface
  show ip igmp interface
  show ip multicast
  show ip multicast interface
*look for joined mcast group per int.
  show bridge-domain 1
  show ip access-list GROUP1-MCAST
  show nve interface nve 1 detail
  show ip mroute

## notes 
'''
power on/off vteps manually
may need to power cycle dr leaf switches to get mcast working.
## mcast testing
'''
#DR-vtep1
int lo0
 ip igmp join-group 239.0.0.4
'''
## Run From vtep1###
'''
ping 239.0.0.4 
remove "ip igmp join-group 239.0.0.4"
'''

## External Switch Information
'''
* dont use VMNet1 it has high cpu. It may help to change the ubridege.exe priority to low in task manager detailed settings.
'''
## Add these vlans after a switch reboot
'''
add vlans 192,193 to VXL-GW1 & 2 after reboot.
conf t
vlan 168
vlan 192
vlan 193
end
wr
'''
## Run the following appliance is powered on
'''
!IOU1
conf t
no spanning-tree vlan 168
int e0/0
 duplex full
int e0/1
 duplex full
int e0/2
 duplex full
int e0/3
 duplex full
int e1/0
 duplex full
 switchport host
end
wr
'''
# End of Lab 2 Notes
