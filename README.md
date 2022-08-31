# GNS3 Applicance Upgrade
upgraded an ESXi VM appliance from Ubuntu 14.x/gns3 2.2.7 to the newer Ubuntu 18.04/gns3 2.2.7 VM and found this procedure worked for migrating all appliances and projects for the old vm to the new ESXi VM:  I do not have a specific question other than if this is not recomened. 

*This procedure may not work for VM for Workstation/Fusion since project configuration files are stored on the client storage and not the server.

# Backup
1) cd/opt - provides a common storage volume with adequate storage space

#Backup vm server projects and images
2) sudo tar -zcvf opt-gns3-2_7.tar /opt/gns3

#Backup symbols and appliances in gns3 server folders
3) sudo tar -zcvf gns3-home2_7.tar /home/gns3
*use ftp client on vm to (binary) put/upload .tar files to shared storage

# Restore:
4) restore the two .tar files to original folders on new ESXi vm server after installing an ftp client on this 2.2.7 vm
5) Use ftp client to get .tar files from storage and (binary) put into /opt on new gns3 VM 
6) cd/ establishes the root folder for restore overlay

#restore vm server projects and images
7) sudo tar -zxvf /opt/opt-gns3-2_7.tar

#restore symbols and appliances in gns3 server folders
8) sudo tar -zxvf /opt/gns3-home2_7.tar
9) reboot vm and connect the GNS3 2.7 client to new ESXi vm
