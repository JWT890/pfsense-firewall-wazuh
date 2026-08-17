# pfsense-firewall-wazuh

# VirtualBox Network Setup
In VirtualBox, go to file and select on Host Network Manager and click on create which should create vboxnet0 with a IP of 192.168.56.1 and turn off DHCP. Then click on preferences and select network -> NAT Network and click on create. 
Name the NAT Network to lab-wan and assign it the ip of 10.10.0.0/24 and click ok.  

# pfsense
To download pfsense go to pfsense.org/download or click this link:  
https://pfsense.org/download/   
After downloading, right click on the file and extract it which will have the iso needed to create the pfSense VM:  
![Pfsense](./images/pfsense.png)    
Then to go VirtualBox and click on create a new VM with the type set to BSD and FreeBSD 64 bit.  
RAM set to 1024 MB with 8 gigs of disk space.   
Then for the adapters, set adapter 1 to lab-wan and 2 to vboxnet0.  
Then boot up the VM and see the terms screen:   
![Terms](./images/terms.png)    
Accept it and continue and see the Welcome to pfSense screen and hit continue to see WAN interface set up:  
![WAN](./images/wan.png)    
The first option will be the NAT that was set up lan-wan and hit ok and continue/proceed with the installation and circle back to this: 
![LAN](./images/lan.png)    
The LAN will be the host only chosen adapter, hit ok and continue/proceed with the installation and confirm the settings and reach this screen: 
![Sub](./images/sub.png)    
Choose the Install CE option and wait a few seconds before getting here:    
![File](./images/file.png)  
Keep the default and hit next and get to this:  
![Disk](./images/disk.png)  
Then hit OK and it will ask if you want to destory the disk, hit yes and wait a few seconds and for the software version, choose the stable version:    
![Stable](./images/stable.png)  
Then hit ok and the install screen will pop up for downloading and updating so wait a few minutes then this screen will appear: 
![Screen](./images/screen.png)  
Then hit OK and get to the do you want to reboot screen, then go to devices -> optical drives and click on the option to remove the iso from virtual drive and click the option of force umount. Then shut down the VM and turn it back on again to get here:
![Replace](./images/replace.png)    
This shows that it installed successfully.  
Then to change the IP of the LAN since it needs to end in a 2. Start by typing 2 and for the next option press 2:   
![Image1](./images/image-1.png)    
Then do n, ignore screenshot, for IPv4 DHCP and no to IPv6 and enter the new LAN IP as 192.168.56.2. After which this screen will pop up:   
![Image2](./images/image-2.png)     
Then for the LAN Subnet type 24 and for IPv6 on LAN put n or just press Enter. Then for IPv6 address LAN put n and for LAN IPv6 address press enter. Then type y for enabling DCHP on LAN and set the range like so:    
![Image3](./images/image-3.png)     
Pressing enter you will see this:   
![Image4](./images/image-4.png)     
Then go type the new address of 192.168.56.2 in a browser:  
![Image5](./images/image-5.png)     
Click advanced and proceed. 
Then you will see the login screen and enter in the admin creds and get into pfsense:   
![Image6](./images/image-6.png)     
Then click next and set up the general information for pfsense, for hostname put something like lab-firewall, domain to lab.local, primary DNS to 1.1.1.1 and secondary to 8.8.8.8. Then hit next.  
For time server set to Central Time or Chicago and hit next.    
Then for WAN Interface section leave it as DHCP and scroll down to hit next.    
Then for LAN Interface section leave it as it and hit next. 
Then set the password and hit next. and for step 7 reload it. After a few seconds:  
![Image7](./images/image-7.png)     
It is finished and hit finish and be taken to the main page:    
![Image8](./images/image-8.png)     
Then to test connectivity go to Diagnostics -> Ping and for hostname type 1.1.1.1 and hit ping: 
![Image9](./images/image-9.png)     
![Image10](./images/image-10.png)   

Then its time to set up a few VLANs, which can be done in either VM or website: 
Website by going to Interfaces -> VLANs:    
![Image11](./images/image-11.png)
Or VM by pressing option 1:  
![Image12](./images/image-12.png)   
In this instance, lets use the Webiste. Click on add and see this screen:   
![Image13](./images/image-13.png)   
Change the parent interface to LAN, change VLAN tag to 10, keep priority at 0 and set Description to MGMT for for the first VLAN and hit save.  
Then set up the other VMs like below:    
![LAN1](./images/lan1.png)  

Then move on to Interfaces -> Assignments to set the assignments:   
![Assign](./images/assign.png)  
Click on add for the first one and add the rest like so:    
![Rest](./images/rest.png)  
And hit save click the dropdown for Interfaces: 
![Option](./images/option.png)  
Click on OPT1 and see this menu:    
![OPT1](./images/opt1.png)  
Enable the interface, change the description to MGMT, change the dropdown for IPv4 to Statis IPv4 and set the first IPv4 address to 10.0.10.1/24 and then hit save and repeat for the other ones.   
SERVERS should be 10.0.20.1/24  
GUEST should be 10.0.30.1/24
IOT should be 10.0.40.1/24  
DMZ should be 10.0.50.1/24
Then head towards Services -> DHCP Server and see this screen:  
![Server](./images/server.png)  
Click on the MGMT tab and enable the interface and for address pool range for MGMT do 10.0.20.100 - 10.0.0.20.199 and hit save, then repeat for the other respective ones and then hit apply changes.   
1-99 range will be reserved for static assignments such as IP for the servers, infrastructure with pfsense sitting at 1 while 100-199 serve as the DHCP pool, while 200-254 serve as future use such as a secondary DHCP pool. This helps since we will be able to tell what IP and DHCP is assigned and help speed up analysis during an incident and reporting. So for example 10.0.20.5 might be a static server while 10.0.20.147 will serve as a endpoint that is dynamic. 
Then go to Firewall dropdown and click on Aliases for this screen:  
![Firewall](./images/firewall.png)  
Before adding each respective one. First add the Private Address space, called RFC1918 first, with the description of Private address space, have it as network type with networks of 10.0.0.0 /8, 172.16.0.0 /12, 192.168.0.0 /16. Then hit save and apply changes. Then go to the Rules section and start adding rules for each VLAN like so: 
![Rules](./images/rules.png)    
THen click on add for MGMT and have the following set up:   
![MGMT](./images/mgmt.png)  
Then save and move on to SERVERS and set up 2 rules, for the first rule:    
![Rule1](./images/rule1.png)    
Then for the second rule:   
![Rule2](./images/rule2.png)    
Then move on to the Guest rule, with the Block above the Allow rule, any for Protocol, Source of Guest Subnets, for Destination of Address or Alias with the address being RFC1918, checked log and a description of Block Guest to internal networks.  
For the second guest role, have it set to Pass for Action, protocol set to Any, Source of Guest subnets, destination of any and a description of Guest Internet only    
![2-Rule](./images/2-rule.png)  
Next its time to create the IOT rule, with the Action set to block, protocol of any, source of OPT4/IOT subnets, destination of any, log checked, description of IOT fully isolated 
Next its time to create the DMZ rules.  
First rule will be a pass, with TCP protocol, source of DMZ subnets with a destination set to SERVERS subnet, dest ports set to custom with a setting that has 22 80 and 443, log checked and a description of DMZ to SERVERS attack simulation, then save. 
Second rule will be a allow, protocol set to ICMP, source of DMZ subnets with a desitination set to SERVERs subnet, log checked and a description of DMZ ICMP to SERVERS.   
Third Rule will be a block, with any protocol, source of DMZ subnets with a destination set to any, log checked, and a description of DMZ default deny. 
Then hit apply changes after saving each

# Linux Switch
Debian Linux download: https://cdimage.debian.org/cdimage/archive/12.0.0/amd64/iso-cd/. 
Linux Switch set up:    
2 GB of RAM
20 GB of Space 
Network Adapter 1 set to VirtualBox Host-Only Ethernet Adapter so it matches. Name the VM Linux-LabSwitch and go through the installation process but when it asks for a mirror say no and continue and uncheck any options except SSH server and system utilties:  
![Process](./images/process.png)    
Make sure to have Network Adapter 2 on during installation for internet access and then login as root after logging by typing su and running ping -c 3 8.8.8.8 like so: 
![Ping](./images/ping.png)  
After confirming run apt update && apt install bridge-utils -y and wait for the installation.   
Then run ip link add br0 type bridge,   
ip link set br0 type bridge vlan_filtering 1,   
ip link set enp0s3 master br0,  
bridge vlan add dev enp0s3 vid 10-50 trunk and if this command on this line doesn't work, rerun apt update && apt install bridge-utils -y and which bridge to see which one and if that returns nothing run:    
/sbin/bridge vlan add dev enp0s3 vid 10-50 trunk and then /sbin/bridge vlan show:   
![Show](./images/show.webp) 
Which shows the bridge is working on the trunked port and to make it permanent run echo 'export PATH=$PATH:/sbin:/usr/sbin' >> /root/.bashrc and then to save for after reboots type nano /etc/network/interfaces and add this at the end of it:    
![Show1](./images/show1.png)    
Save it and then type systemctl restart networking and then /sbin/bridge vlan show to verify:   
![Show2](./images/show2.png)    

# Monitoring
For the monitoring phase it will be a combination of Suricata, ntopng, pfBlockerNG-devel, and Sysmon    
Go to pfsense -> System -> Package Manager -> Available Packages and get to here:   
![Package](./images/package.png)    
First install ntopng through the search bar and see this:   
![NTO](./images/nto.png)    
Click install and wait for it to install, then Suricata next, wait for it, then install pfBlockerNG-devel and when done check installed packages:   
![Install](./images/install.png)    
Then click on Services and notice that Suricata appears in it:  
![Suricata](./images/suricata.png)  
Click on it and go to Global Settings page: 
![Global](./images/global.png)  

