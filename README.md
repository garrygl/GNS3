# GNS3
I recently upgraded an ESXi VM appliance from Ubuntu 14.x/gns3 2.2.7 to the newer Ubuntu 18.04/gns3 2.2.7 VM and found this procedure worked for migrating all appliances and projects for the old vm to the new ESXi VM:  I do not have a specific question other than if this is not recomened. 

*This procedure may not work for VM for Workstation/Fusion since project configuration files are stored on the client storage and not the server.

I used this on two instances with success.

# Backup
cd/opt - provides a common storage volume with adequate storage space

#Backup vm server projects and images
sudo tar -zcvf opt-gns3-2_7.tar /opt/gns3

#Backup symbols and appliances in gns3 server folders
sudo tar -zcvf gns3-home2_7.tar /home/gns3

#use ftp client on vm to (binary) put/upload .tar files to shared storage

# Restore:
restore the two .tar files to original folders on new ESXi vm server after installing an ftp client on this 2.2.7 vm
use ftp client to get .tar files from storage and (binary) put into /opt on new gns3 VM 
cd/ establishes the root folder for restore overlay
cd/
#restore vm server projects and images
sudo tar -zxvf /opt/opt-gns3-2_7.tar

#restore symbols and appliances in gns3 server folders
sudo tar -zxvf /opt/gns3-home2_7.tar
#reboot vm and connect the GNS3 2.7 client to new ESXi vm
