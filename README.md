# pfsense-firewall-wazuh

# VirtualBox Network Setup
In VirtualBox, go to file and select on Host Network Manager and click on create which should create vboxnet0 with a IP of 192.168.56.1 and turn off DHCP. Then click on preferences and select network -> NAT Network and click on create. 
Name the NAT Network to lab-wan and assign it the ip of 10.10.0.0/24 and click ok.  

# pfsense
To download pfsense go to pfsense.org/download or click this link:  
https://pfsense.org/download/   
Then click on create a new VM with the type set to BSD and FreeBSD 64 bit.  
RAM set to 1024 MB with 8 gigs of disk space.   
Then for the adapters, set adapter 1 to lab-wan and 2 to vboxnet0.  
Then go the downloads folder and find the netgate download and type gunzip and the file to get it unzipped. After that boot into the VM as a iso and configure the settings like wlan, lan and more to get pfsense installed, then click on ok for it to reboot. After it gets done rebooting, go the storage side on VirtualBox and remove the pfsense installer and restart the VM. After a few minutes in the VM, this screen will appear:   
![alt text](image.png)  
This shows that it installed successfully.  
Then to change the IP of the LAN since it needs to end in a 2. Start by typing 2 and for the next option press 2:   
![alt text](image-1.png)    
Then do n, ignore screenshot, for IPv4 DHCP and no to IPv6 and enter the new LAN IP as 192.168.56.2. After which this screen will pop up:   
![alt text](image-2.png)    
Then for the LAN Subnet type 24 and for IPv6 on LAN put n or just press Enter. Then for IPv6 address LAN put n and for LAN IPv6 address press enter. Then type y for enabling DCHP on LAN and set the range like so:    
![alt text](image-3.png)    
Pressing enter you will see this:   
![alt text](image-4.png)    
Then go type the new address of 192.168.56.2 in a browser:  
![alt text](image-5.png)    
Click advanced and proceed. 
Then you will see the login screen and enter in the admin creds and get into pfsense:   
![alt text](image-6.png)    
