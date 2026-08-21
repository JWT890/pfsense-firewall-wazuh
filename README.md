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
Check the Install ETOpen Emerging Threats rules, Install Snort GPLv2 Community Rules, Install Feodo Tracker Botnet C2 IP rules and Install ABUSE.ch SSL Blacklist rules and set the updates interval to 12 hours. Then hit save and go to the updates tab and hit update rule set and wait a few seconds.   
Thn go to Interfaces tab and add the interfaces by clicking add for the first one see this screen:  
![Interface](./images/interface.png)    
The first one will be the WAN so enable it and check send alerts to system log with the log facility being SYSLOG and Log Priority to ALERT and hit save.   
After saving go back to Suricata and click on add once again, enable it and set the interface to SERVERS with a description of SERVERS IPS with log facility and log priority same as above, have block offenders checked and kill states enabled and hit save. 
Then add the third one for DMZ with a description of DMZ Kali Monitor, leave block offenders unchecked and send alerts to system log checked with it being the same as SYSLOG and Log Priority set to ALERT, then hit save. Then start all three and check for the green mark:  
![Green](./images/green.png)    
For ntopng, click on Diagnostics and click on ntopng Settings and this screen:  
![NTO1](./images/nto1.png)  
Check enable ntopng, set a strong admin password, for monitoring options Control + C for all except WAN and hit save. Then click the access ntopng option:  
![Option1](./images/option1.png)    
After clicking on it, the login will pop up:    
![Login](./images/login.png)    
After logging in:
![Map](./images/map.webp)   
![Screen1](./images/screen1.png)  
From watching the after login screen, you can watch the data move in real time and see what is going on in the network  
Before configuring pfBlockerNG go to Firewall -> Virtual IPs and click on add:  
![IP](./images/ip.png)  
Check IP Alias, change the interface to Localhost, set the address to 127.0.0.1 /32 with a description of pfBlockerNG VIP, save it and apply changes, then go to Firewall -> pfBlockerNG and see the setup screen:  
![Setup](./images/setup.png)    
On Step 2 have both Inbound and Outbound Firewall Interfaces be WAN and hit next and get to DNSBL config:   
![Config](./images/config.png)  
Select the dropdown for the first option and have it the same as the alias and hit next and hit finish and wait for the install:    
![Work](./images/work.png)  
From looking over this part alone:  
![Look](./images/look.png)  
Shows that over 17,000 IPs already have been blocked with the firewall enabled

# Wazuh
For the Wazuh SIEM setup:   
Linux, Debian   
RAM: 8192   
80 GB of Space and several processors   
Boot Order keep all and have adapter 1 set to host only like the switch and pfsense and adapter 2 set to NAT. Have an Ubuntu iso in the storage, then go about the installation process. When it gets to profile configuration, have the server's name as wazuh-siem and set a password for it and wait for it to get done installing. Make sure check SSH part as well.    
Then sign in, then through WSL ssh in and install Wazuh:    
First run sudo -i to get into root  
Then run apt update && apt install -y curl nano and wait for it to finish, then create the folder for it by typing mkdir /root/wazuh-install then cd into it, then run the two curl commands
curl -s0 https://packages.wazuh.com/4.7/wazuh-install.sh 
curl -s0 https://packages.wazuh.com/4.7/config.yml  
Then to verify run ls:  
![LS](./images/ls.png)  
After verifying open config.yml by running nano config.yml: 
![Node](./images/node.png)  
Change the ip for each to 10.0.10.10 and save the config and run bash wazuh-install.sh -a and wait for it to finish or 5-20 minutes.    
After installation, get the user and password and type the address of the VM into a browser and click on the advanced option on the warning page and get to Wazuh login:    
![Wazuh](./images/wazuh.png)    
The wazuh-dashboard-recovery.md file above is a good source for getting Wazuh up with errors. After logging in: 
![Wazuh1](./images/wazuh1.png)  
But whenever moving to different places, it seems that the same timeout error occurs, the wazuh-dashboard-backend-timeout-resolution.md file above is a good source.    
Now check Server Management -> Status to see if its working:    
![Working](./images/working.png)    
Then in command line type sudo nano /var/ossec/etc/ossec.conf:  
![Config1](./images/config1.png) 
And in in the ossec_config section: 
![Remote](./images/remote.png)  
Then delete the second ossec.conf section below it and to verify run sudo nano xmllint --noout /var/ossec/etc/ossec.conf to verify the xml. 
Then restart by running sudo systemctl restart wazuh-manager, wait a few seconds and run sudo systemctl status wazuh-manager --no-pager to verify all status are running.   
Then run sudo ss -lunp | grep ':514' to verify its listening:   
![Listen](./images/listen.png)  
Then go to the pfsense site and go to Status -> System Logs and go over to settings:    
![Settings](./images/settings.png)  
Scroll down and click on enable remote logging and set the source address to LAN, remote log server to 192.168.56.119:514 and others blank with all logs to everything: 
![Setup1](./images/setup1.png)  
Then hit save and it will save the setup.   
Then to test if the logs are flowing in the Wazuh side, type sudo tail -f /var/ossec/logs/archives/archives.log and ping in pfsense to generate some traffic but in the Wazuh side shows not much.  
Then type sudo ss -lunp | grep ':514' and sudo ufw status to check the status of the ufw firewall and it shows listening status and ufw is disabled but the listening status is sending from the LAN. Then type sudo tcpdump -i any udp port 514 -n for udp traffic and view the udp log for more.  
Then run sudo tail -f /var/ossec/logs/archives/archives.log and notice no output, so run sudo nano /var/ossec/etc/ossec.conf and look for logall and change both logall to yes to enable the arhive logging. Then restart by typing sudo xmllint --noout /var/ossec/etc/ossec.conf and sudo systemctl restart wazuh-manager. Then sudo ls -lh /var/ossec/logs/archives/ and see that the directory exists with data in it. Run sudo tail -f /var/ossec/logs/archives/archive.log to see this:   
![Log](./images/log.webp)   
Wazuh is logging data and looking at a couple lines like starting rootcheck scan and df -P filesystem output is active means its fully operational. Go to the Wazuh web site and go to Threat Hunting:  
![Result](./images/result.webp) 
And see the rootcheck alerts meaning its fully operational. 
Then go to Agent management and click on Summary and see this screen:   
![Agent](./images/agent.png)    
And click on Deploy new agent:  
![Agent1](./images/agent1.png)  
Select the DEB amd64 option with the server address set to 192.168.56.119 and name it endpoint-server. 
Before continuing create a endpoint VM and name it servers-endpoint, 2 GB of RAM, 20 GB of Space, network adapters like the others and an Ubuntu OS and create the VM with OpenSSH and just standard utilties.  
If running from the Wazuh site: wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.7-1_amd64.deb, pops up with a 403 forbidden error run: 
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg. Which will download the GPG key and create the folder and permissions for it: 
![Import](./images/import.png)  
Then run echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list. Which will create the signed by distinction for the key and append it to the wazuh.list. Then run sudo apt update to update and then run:  
sudo WAZUH_MANAGER='192.168.56.119' WAZUH_AGENT_NAME='endpoint-server' apt install wazuh-agent -y which says unable to locate wazuh-agent so run:   
![Commands](./images/commands.png)  
With the last command from the photo helps with installing the IP manager set and the wazuh agent.  
Then start the agent by running:    
sudo systemctl daemon-reload    
sudo systemctl enable wazuh-agent   
sudo systemctl start wazuh-agent and    
sudo systemctl status wazuh-agent --no-pager:   
![Status](./images/status.png)  

