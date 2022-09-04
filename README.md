# This Document Contains Information for Ugrading GNS3 Appliances and Two Projects hosted on the GNS3 Platform

## Table of contents
* [Upgrading GNS3 Appliances](#Upgrading-GNS3-Appliances)
* [GNS3 Simulation Labs](#GNS3-Simulation-Labs)
* [Lab 1](#Simulation-Lab1-Azure-ExpressRoute)
* [Lab 2](#Simulation-Lab2-VXLAN-&-DNA-C)

## Upgrading GNS3 Appliances 

Abstract: This following outlines a prodecure for upgrading an ESXi GNS3 VM appliance from Ubuntu 14.x/gns3 2.2.7 to the newer Ubuntu 18.04/gns3 2.2.x VM. I verfied this procedure worked for migrating all appliances and projects for the old vm to the vm appliance. 
This procedure may not work for a Workstation/Fusion VM since project configuration files are stored on client side storage and not on the server.
Reference - **[Link to GNS3 Post (HTML format)](https://gns3.com/upgrade-ubuntu-14-x-gns3-2-2-7-t)

## Backup GNS3 Projects and Appliances
```bash
cd/opt - provides a common storage volume with adequate storage space
# Backup vm server projects and images
sudo tar -zcvf opt-gns3-2_7.tar /opt/gns3
# Backup symbols and appliances in gns3 server folders
sudo tar -zcvf gns3-home2_7.tar /home/gns3
# Use ftp client on vm to (binary) put/upload .tar files to shared storage
```
## Restore GSN3 Projects and Appliances

Restore the two .tar files to original folders on new ESXi vm server after installing an ftp client on this 2.2.7 vm
1. Use ftp client to get .tar files from storage and (binary) put into /opt on new gns3 VM 
2. cd/ establishes the root folder for restore overlay

## Restore VM and Images
```bash
sudo tar -zxvf /opt/opt-gns3-2_7.tar
# Retore symbols and appliances in gns3 server folders
sudo tar -zxvf /opt/gns3-home2_7.tar
# reboot vm and connect the GNS3 2.7 client to new ESXi vm
```
## End-of-GNS3-Appliance-Upgrade-Procedure

## GNS3 Simulation Labs

## Lab 1

Abstract: This working lab simulates connecting on-premises networking to Azure ExpressRoute and an S2S VPN via a PAN firewall. It also connects AWS with a S2S VPN via the PAN firewall and a simulated Internet backbone. An external switch running ubridege.exe provides the bridged path allowing a connection to the following external lab systems: Cisco DNA-C appliance, Cisco APIC-EM, Stealwatach, Prime Infrastruture, and Cisco ISE. 
- Reference - **[Link to Network-Lab.pdf (HTML format)](https://github.com/garrygl/GNS3/blob/20b25805cb25cc5977216e78a9a4a0c04696f3e3/Network-Lab.pdf)
* Azure routes are redistributed from BGP to OSPF and from OSFP to BGP using Type-2 link-state advertisements
* Webterm is a docker container and sometimse will not copy change the ip address to 192.168.1.10 - .2 is OOB PAN
* Use bgp local pref to prefer CENIC over Comcast for Azure ExpressRoute if CENIC is not availble, prefer Comcast
* OSPF Type 2 route redist into OSPF on WAN-1 to avoid having the metric change from link-to-link. ExpressRoute is prefered
* The static 10.30.0.0/19 on the PAN is inserted into the routing table if the vWAN peer is reachable 
```bash
# The more specific /24 routes from the equinix peer are inserted into ospf type2 from bgp to ospf redistribution from WAN-1
# BGP Redistribute Command
address-family ipv4
  bgp redistribute-internal
```
## PAN Routing Policy
```bash
# Look for MM_Active 
# PAN Poilcy Based forwarding rule to Azure Cohesity
# PAN 10.30.0.0/19 is static and only inserted if tunnel peer is operational
# ExpressRoute is prefered over vWAN VPN
Show cry isa sa
show cry ips as
debug cry ipsec 255
```
### Appliance Notes
* Must start NX9K firest, wait for boot followed by the CSR1K's
* Start PAN after 1K's wait 15mi.
* Start the other applainces one at a time
* May require 30m to boot lab

## PAN Appliance Notes - Wait for booting (at least 5 minutes) 
### For new cofigs on pan run 
```
delete deviceconfig system type dhcp-client"
set deviceconfig system ip-address 192.168.1.2 netmask 255.255.255.0
Webconsole-persistent select live-usb at boot screen
```
# End-of-Lab 1

## Lab 2 

Abstract: This lab simulates a working CLOS leaf-and-Spine VXLAN MCAST fabric using Cisco CSR1000v appliances as the spines. An external switch simalates a bridge connecting 

- Reference - **[Link to Cisco_VXLAN_APIC-EM_Stealthwatch_Prime-Infrastructure_Networking-Lab.pdf (HTML format)](https://github.com/garrygl/GNS3/blob/20b25805cb25cc5977216e78a9a4a0c04696f3e3/Cisco_VXLAN_APIC-EM_Stealthwatch_Prime-Infrastructure_Networking-Lab.pdf)
- VXLAN Tunnel Endpoint (VTEP): Map tenants’ end devices to VXLAN segments. Used to perform VXLAN encapsulation/de-encapsulation.
- Virtual Network Identifier (VNI): identify a VXLAN segment. It can contain up to 224 IDs theoretically giving us 16,777,216 segments. (Valid VNI values are from 4096 to 16777215). 
- Each segment can transport 802.1q-encapsulated packets, theoretically giving us 212 or 4096 VLANs over a single VNI.
- Network Virtualization Endpoint or Network Virtualization Edge (NVE): overlay interface configured in Cisco devices to define a VTEP
# Useful Show Commands
```bash
# Look for joined mcast group per interface
show run | i ip pim|interface
show ip igmp interface
show ip multicast
show ip multicast interface
# Look for nve per interface
show bridge-domain 1
show ip access-list GROUP1-MCAST
show nve interface nve 1 detail
show ip mroute
```
## Operational Notes 
```
power on/off vteps manually
may need to power cycle dr leaf switches to restore mcast process.
```
## MCAST Testing 
```bash
# DR-vtep1
int lo0
 ip igmp join-group 239.0.0.4
# Run From vtep1
ping 239.0.0.4 
remove "ip igmp join-group 239.0.0.4"
```
## External Switch Information
* dont use VMNet1 it has high cpu. It may help to change the ubridege.exe priority to low in task manager detailed settings.
## Bug: Fixes a Missing Config For Vlans After Switch Power On
```bash
# add vlans 192,193 to VXL-GW1 & 2 after reboot. This is a bug.
conf t
vlan 168
vlan 192
vlan 193
end
wr
```
## Bug: Fixes a Missing Config on IOU1 After a Power On
```bash
# IOU1
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
```
## End-of-Lab 2
# End-of-File

