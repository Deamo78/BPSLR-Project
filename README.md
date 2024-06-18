# 2023CyberProject - CSOC Cybersecurity Operations Centre for SME's


## Notes - restart elastic agent on powerup
```
$sudo elastic-agent restart
```

### Project overview
The cybersecuirty project implements the development execution, evaluation and documentation of a prototype cybersecurity model (CSOC) infrastructure for a small to medium size organisation. It includes both physical and virtualised firewalls, servers, clients and SIEM. Project systems testing, Red and Blue (Purple) teaming activities are undertaken and documented. 

### Project Requirements

The project incorportes the development, execution, evaluation and documentation of kaliPurple SIEM in a simulated virtual environment.
It includes creating, configuring and interconnecting the following physical and virtualised systems:  
- PaloAlto firewall,  
- Cisco Switch,  
- KaliPurple SIEM,   
- Windows server 22 Domain Controller  
- endpoint protected windows 10/11 clients using Elastc defend,  
- Ubuntu LAMP DMZ WEB server,    
- suricata IDS/IPS in OPNsense firewall.  

The Project also includes red and blue (purple) teaming exercises :  
- Red team testing and evaluation using Kali, for testing and evaluating the firewall rules, windows clients and DMZ Web Server 
- Blue team responding using Kali Purple SIEM for alert detection and response.  

## Initial VS Prototype VS Production system configuration 
The initial lab environment uses virtualised systems and a VirtualBox NAT Network firewall with VMs connected using VirtualBox iNAT networking.  

The prototype lab environment uses virtualised systems and a Virtual OPNsense firewall with VMs connected using VirtualBox internal networking.  

The production lab environment uses virtualised systems that use bridged networking to connect via a physical Cisco Switch to a physical Palo Alto Firewall.  
Each particpant will implement the prototype system and each team will implement a production system.  

### Project technical detail - Milestones: 

[1. Set up and interconnect Networking infrastructure](#1-set-up-and-interconnect-networking-infrastructure)

[2. Set up configure and interconnect Virtual environment, firewall , servers and client](#2-set-up-configure-and-interconnect-virtual-environment-firewall--servers-and-client)

[3. Configure kali Purple - for remote management - ssh, rdp, console](#3-configure-kali-purple---for-remote-management---ssh-rdp-console)  

[4. Implement kali Purple SIEM - install elastic stack - elastic , kibana](#4-implement-kali-purple-siem---install-elastic-stack---elastic--kibana)  

[5. Implement Fleet Server and end point protection agent - overview / installation](#5-implement-fleet-server-and-end-point-protection-agent---overview--installation)  

[6.Integrate OPNsense firewall logs with KaliPurple SIEM](#6-integrate-opnsense-firewall-logs-with-kalipurple-siem)  

[7.Integrate Palo Alto Firewall into Kali Purple SIEM](#7-integrate-palo-alto-firewall-into-kali-purple-siem)  

[8. DMZ Web server](#8-dmz-web-server) 

[9. Configure and Test Network](#90-configure-and-test-network)  

[10.0 Establish a Network Security Baseline](#100-establish-a-network-security-baseline)  

[11. Testing RBT activities](#11-testing-rbt-activities)  

## 1. Set up and interconnect Networking infrastructure

### 1.0 IP adresses
- KaliPurple        192.168.100.200
- Windows 22 DC     192.168.100.100
- Windows Clients   192.168.100.10-20
- Kali Inside       192.168.100.250
- Switch VLAN1      192.168.100.254
- Palo Fw E1/2      192.168.100.1
- Palo Fw E1/1      192.168.108.2XX
- Palo Fw E1/3      192.168.50.1
- LAMP DMZ Web      192.168.50.100 
- Kali Outside      192.168.108.X  

### 1.1 Palo Alto Firewall - basic setup
Palo alto firewall has the following interfaces and Zones:  
E1/1 - WAN zone - 192.168.108.2XX/24 where XX is unique    
E1/2 - LAN zone - 192.168.100.1/24 - running a DHCP server  
E1/3 - DMZ Zone - 1092.168.50.1/24  
Management interface - 192.168.1.1/24  

### 1.2 Palo Alto Firewall - basic feature configuration
#### Static IP on Ethernet Interface E1/1
Network > Interfaces - Interafce ethernet1/1  
- Interface Type: Layer 3  
- Config - Assign interface to: 
    - Virtual Router:  V-Router 
    - Security Zone: outside  
IPv4  
- Type: Static , IP : 192.168.108.2XX/32  
#### Zones
Network > Zones   
- Inside - layer 3 , ethernet1/2   
- Outside - layer 3 , ethernet1/1   
- DMZ - layer 3 , ethernet1/3   
#### Default static route in Virtual Router
network > Virtual Routers 
- V-Router 
    - Router Settings > General > Interfaces: ethernet1/1, ethernet1/2, ethernet1/3  
    - Static Routes > IPv4 > (add)  
        - Name: Default-route  
        - Destination: 0.0.0.0/0
        - Interface: ethernet1/1
        - Next Hop: IP Address  
        - 192.168.108.1  


#### Device timezone
Device > Setup > Management - General Settings
- Timezone : Australia / Melbourne
#### Service Routes
Device > Setup > Services > Service Features - service Route Configuration - customise  IPv4  
- DNS - ethernet1/1 , 192.168.108.2XX  
- Palo Alto Networks Services - ethernet1/1 , 192.168.108.2XX  

Device > Setup > Services > Services - DNS Settings  
- Primary DNS Server: 8.8.8.8  
- Secondary DNS Server: 192.168.210.2  

#### Configure Source NAT from LAN to WAN (using PAT) 
refer labs:
- PAN10_EDU210_Lab_05 Configuring Security Policy Rules and NAT Policy Rules  
- PAN10_EDU210_Lab_17 Capstone  
#### Configure Base security policy allow any from LAN to WAN  
refer labs:
- PAN10_EDU210_Lab_05 Configuring Security Policy Rules and NAT Policy Rules  
- PAN10_EDU210_Lab_15 Implementing Day-One Best Practice Configuration    
- PAN10_EDU210_Lab_17 Capstone  
#### Optionally set up manangemnt profile to allow firewall management from host in internal network  
Network > Network profiles > Inetrface Management - Add  
- name: inside-management-profile  
- Administrative management services: HTTPS, SSH
- network services: Ping, Response pages
- Permitted IP Addresses: 192.168.100.0/24  (or a single IP - refer security policy)

Network > Interfaces > Ethernet - ethernet1/2 - Advanced  
- Management Profile: inside-management-profile   

### 1.3 Switch SW1 - basic setup

VLAN1 - 192.168.100.254/24 for remote management via SSH  
No port security at this stage  
```
!
hostname S1
!
!
interface Vlan1
 ip address 192.168.100.254 255.255.255.0
!
logging trap debugging
logging 192.168.100.200
!
!
!
line con 0
 password cisco
 login
!
line vty 0 4
 password cisco
 login
!
!
!
ntp server 192.168.100.100
!
end
```
### 1.4 Cable up Physical network according to network Diagram

![Network diagram image](images/NetworkDiagram.png)  

### 1.5 Zone Location of components
- Palo Alto Firewall - Inside, Outside, DMZ
- Network Switch - Inside
- 1 x Wireless AP - Inside
- 1 x Virtual Windows server DC , AD DNS - Inside 
- 3-4 x Virtual Windows clients with agents - Inside
- 1 x Virtual Kali Purple SIEM - Inside 
- 1 x Virtual Ubuntu Linux Web server LAMP - DMZ
- 1 x Virtual Kali Pentest VM - Outside 

## 2. Set up configure and interconnect Virtual environment, firewall , servers and client

### 2.1 Install VirtualBox

### 2.2 Virtual box networking

Your choice of Virtual box networking to use for your LAN zone VMs will depend on whether you are implementing the prototype (with or without OPNsense firewall) or implementing the production network as follows:  
- Initial LAN VMs without OPNsense firewall - use NAT Network     
- Prototype LAN VMs behind OPNsense firewall - use internal Network  
- Production LAN VMs behind Palo Alto Firewall - used Bridged Network  
- Production DMZ VM behind Palo Alto Firewall - used Bridged Network
- Prototype DMZ VM behind OPNsense firewall - use internal Network 



#### 2.2.0 Setting up a NAT Network in VirtualBox for prototype VM installation - Initial configuration
File > Tools > Network manager - NAT Networks
Create  
General Options  
Name: CyberProjectNAT  
IPv4 Prefix: 192.168.100.0/24  
tick - Enable DHCP  (optional if using DHCP on Windows Domain Controller) 
[APPLY]   

#### Simulated Firewall: Initial configuration VirtualBox using NAT Network
You can prototype your SOC/SIEM and Domain in a virtual network by creating a VirtualBox NAT network to simulate a firewall:  
NAT Network address 192.168.100.0/24 and optionally enabling DHCP  

When installing Windows DC or Client, install from iso into an internal network initially.  
After the install of Windows DC and Clients you can move them to NatNetwork.  

When installing KaliPurple you can use the NatNetwork 192.168.100.0/24. 

VirtualBox NAT Network  will provide DHCP.  

Ultimately in the production network all VMs will be bridged to the physical host adapter and connected to the Physical switch and Palo Alto Firewall.  

#### OPNsense firewall (Prototype): using Internal Network
IF using OPNsense firewall your KaliPurle, Windows Server and Windows Client VMs can use Internal network. OPNSense firewall will provide DHCP.  

#### Palo Alto Firewall (Production):  using bridged Network
Ultimately all VMs will be bridged to the physical host adapter and connected to the Physical switch and Palo Alto Firewall. 
This allows multiple Virtual Windows Clients for all SOC Team members to access the one SIEM/SOC from their Windows Clients.  PaloAlto firewall will provide DHCP. 

### 2.3 Install OPNsense Firewall into Virtual Box 

Download OPNsense (dvd:iso) and unzip .iso from here:  https://opnsense.org/download/  
Verify download checksum: https://docs.opnsense.org/manual/install.html#download-and-verification  
```
C:\Users\xxx\Downloads>certutil -hashfile OPNsense-24.1-dvd-amd64.iso.bz2 sha256
SHA256 hash of OPNsense-24.1-dvd-amd64.iso.bz2:
6d1e22713bf031d0a36a73b3820cd1564f426cae9c67a6ade4b7fa6518afa2d5
CertUtil: -hashfile command completed successfully.
```
use 7zip to unzip .iso  
New virtual machine in Virtual Box  
- attach iso  
- install directory to portable SSD  
- General settings > Free BSB 64-bit, 4 Gig RAM, 20 Gig storage   

Virtualbox > new VM  

VM Name and OS  

Name: OPNsenseXX  
Folder: (on your USB3SSD)  
ISO Image: OPNsense-24.1-dvd-amd64.iso  
Type: BSD  
Version: FreeBSD (64-bit)  
NEXT  

Hardware

Base Memory: 4 Gig RAM
Processors: 1

NEXT

Virtual Hard disk  

Create a virtual hard disk now  , 20 Gig storage   

Settings > Network  
#### Add 2 network adapters cards   
Network > Adapter 1 - Attached to Internal network  = LAN  
Network > Adapter 2 - Enable , Attached to Bridged Adapter - Ethernet card on PC (or WIFI)  = WAN  

OK


Note: Adapter 1 will be allocated to em0 , Adapter 2 will be allocated to em1 - in OPNsense  

#### Start OPNsense VM and copy files from .iso to virtual disk
Start OPNsense VM  

login as: installer  
Passwd: opnsense  

accept defaults 

copy files to disk partition 
select VBOX HARDDISK 10 (20 GB)  with down arrow  , enter for OK  
Yes to destroy disk contents  , left arrow and enter  
files will be copied from the iso to the VDI file

Enter root password: opnsense 
confirm password
down arrow to 
Complete install Exit and reboot  , enter to OK


ref: https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/201_10:-Byzantium-installation

#### shutdown OPNsense and remove .iso from CD drive and reboot
Start OPNsense VM  

login as: root  
Passwd: opnsense  

Enter an option: 5  
Do you want to proceed: y  

Remove iso from CD drive   
Settings > Storage 
Optical drive - Remove disk from Virtual Drive  


#### Start OPNsense VM and configure LAN
Start OPNsense VM  

login as: root  
Passwd: opnsense  

#### 2.3.1 Configure OPNsense firewall 
WAN interface should get a DHCP allocated IP address  

#### OPNsense Networks
OPNsense firewall has the following interfaces and Zones:  
em1 - WAN - 192.168.108.XX/24 where XX is unique (DHCP Allocated initially)   
em0 - LAN - 192.168.100.1/24 - running a DHCP server  
em2 - DMZ - 192.168.50.1/24   


#### Set up DHCP on LAN interface em0  

Enter an option: 2 (set interface IP address)  
Enter the number of interface to configure: 1 - LAN (em0)  
Configure IPv4 address LAN interface via DHCP: N  
Enter the New LAN IPv4 Address: 192.168.100.1  
Enter the New LAN IPv4 subnet bit count (1 to 32): 24  
Configure IPv6...: y  
Do you want to enable the DHCP server on LAN: y  
Enter the start address: 192.168.100.40  
Enter the end address: 192.168.100.49  
Do you want to change the web GUI protocol from HTTPS to HTTP? : y  
Restore web GUI access defaults: y  



#### shutdown OPNsense and remove .iso from CD drive and reboot

#### 2.3.2 Integrating OPNsense firewall with internal hosts 
When using OPNsense firewall , internal hosts will be attached to the virtualBox internal network
Ie - KaliPurple, Windows Server 22, Windows client

####

### 2.4 Install kali Purple


#### Installation Video

Video YouTube - https://youtu.be/kGXabs7Sq6c

This video demonstrates Install of Kali Purple behind OPNsense firewall

Download kaliPurple .iso from here:  

Virtualbox > new VM  
attach iso  
install directory to portable SSD  
General settings > Debian 11 bullseye 64-bit, 8192MB RAM, 50 Gig storage  

#### Virtual Box Networking : change to NAT Networking (for basic setup) or Bridged (for Palo Alto Firewall) OR Internal (for OPNsense Firewall)
Settings > Network > Adapter 1  attached to Internal/NAT/Bridged network  

VirtualBox > Start VM 

Graphical install  
English  
Australia  
American English  

set hostname , domain, username and password  - subsitute sg for your initials  
hostname: siem  
domain: sg.local  

Guided - use entire disk  
all files in one partition  
finish partitioning and write changes to disk  
Partition disks - yes  

Software selection - accept defaults  

install grub - y  
on /dev/sda  

reboot  
login 

open terminal  

#### Kali Purple check IP and connectivity

```
$ip a
$ ping 8.8.8.8
```
#### Kali Purple update OS, services and apps from apt repository
```
$sudo apt update  
$sudo apt upgrade -y  
```
#### Kali Purple Set static IP

ref - https://wiki.debian.org/NetworkConfiguration


##### Configuring the interface manually

Add the below yaml to the bottom of the /etc/network/interfaces file using nano  
```
$sudo nano /etc/network/interfaces
```
```
    auto eth0
    iface eth0 inet static
        address 192.168.100.200/24
        gateway 192.168.100.1
        dns-nameservers 8.8.8.8

```

the complete file will look like this:
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 192.168.100.200/24
    gateway 192.168.100.1
    dns-nameservers 8.8.8.8

```
Edit the resolv.conf file to set the dns server address (8.8.8.8) for a static IP configured network  
```
$sudo nano /etc/resolv.conf
nameserver 8.8.8.8
```
#### Note resolv.conf will be overwritten if using GUI based Network Manager

##### Starting and Stopping Interfaces to change the IP address on eth0  
```
$ sudo ifdown eth0
$ sudo ifup eth0

```
##### Reinitialize new network setup
```
# systemctl status networking
# systemctl restart networking
```

check connectivity  assumes DHCP is running on Palo Alto Firewall

```
$ip a  
$ping 192.168.100.1    
$ping 192.168.108.1  
$ping 8.8.8.8    
```
#### if you are having networking issues try this:  

```
$sudo nano /etc/NetworkManager/NetworkManager.conf
Change false to true

$service networking restart
```
#### do a reboot and check if nameserver setting have stuck  
```
$reboot
$sudo apt update
$cat /etc/resolv.conf
```
#### Run OPNsense firewall configuration wizard and install updates
Ref video:
Open firefox in kaliPurple, Navigate to http://192.168.100.1 login as root and run the wizard 
System > Wizard  
- Next  
System: Wizard: General Information  
- Next  
System: Wizard: Time Server Information  
- Timezone - Australia/Melbourne  
- Next  
System: Wizard: Configure WAN Interface  
- Next  
System: Wizard: Configure LAN Interface  
- Next  
System: Wizard: Set Root Password  
- Next  
System: Wizard: Reload Configuration  
- Reload  
System: Firmware - Status   
- Check for updates  
- Update  
- OK  

After a reboot you will have the latest OPNsense version.   


#### Virtual Box Networking : change to NAT Networking (for basic setup) or Bridged (for Palo Alto Firewall) OR Internal (for OPNsense Firewall)

### 2.4.2 PRODUCTION NETWORK - connect to Palo Alto Firewall
open browser and navigate to https://192.168.100.1  
login as admin on PAFW (Bridged only)  and requires a https management profile on E1/2   
passwd: Password1  

### 2.5 Install windows 10/11 Clients
Download win iso from here :  S Drive  
VirtualBox > new VM  
attach iso  
install directory to portable SSD  
Note: Skip unattended install  

General settings > Windows 10 64-bit, 4Gig RAM, 50 Gig storage  
Network > Adapter 1  attached to Internal network  



Notes: when it asks to connect to internet say - no internet continue install  

Login and set Initial IP to 192.168.100.20/24 , GW: 192.168.100.1 DNS: 192.168.100.100, 8.8.8.8   

network profile - select private network  

login and check connectivity 

```
C:>ipconfig /all   
C:>ping 192.168.100.1  
C:>ping 192.168.108.1  
C:>ping 8.8.8.8  
```

#### Virtual Box Networking : change to NAT Networking (for basic setup) or Bridged (for Palo Alto Firewall) OR Internal (for OPNsense Firewall)

open browser and navigate to https://192.168.100.1  
login as admin on PAFW (Bridged only)  and requires a https management profile on E1/2   
passwd: Password1  


### 2.6 Install and configure Server 22 Domain controller
#### 2.6.1 Install Server 22
Download win iso from here :  
VirtualBox > new VM  
attach iso  
install directory to portable SSD  
Note: Skip unattended Install  

General settings > Windows 22 64-bit, 4Gig RAM, 50 Gig storage  
Network > Adapter 1  attached to Internal network  ( or Not Attached if OPNsense is running)

Install desktop experience either standard or data centre edition  
Notes: when it asks to connect to internet say - no internet continue install  

Login and set Initial IP to 192.168.100.100/24 , GW: 192.168.100.1 , DNS: 127.0.0.1, 8.8.8.8    

network profile - select private network  

login and check connectivity 
#### Virtual Box Networking : change to NAT Networking (for basic setup) or Bridged (for Palo Alto Firewall) OR Internal (for OPNsense Firewall)


```
C:>ipconfig /all   
C:>ping 192.168.100.1  
C:>ping 192.168.108.1  
C:>ping 8.8.8.8  
```
open browser and navigate to https://192.168.100.1  
login as admin on PAFW (Bridged only)  and requires a https management profile on E1/2   
passwd: Password1  


#### 2.6.1 Promote server to Domain controller
Nmne the server using your initials - eg. sg    
hostname: sgserver  
domain: sg.local  

Note: change the hostname BEFORE promoting to a domain controller!!!   

#### 2.6.2 Create a security group and user accounts
#### 2.6.3 Join windows clients to domain

#### 2.6.4 Set up DNS on domain controller

##### Entry for Kali Purple  

Server manager > Tools > DNS  

DNS - "ServerName" - Forward lookup zones - "domain"   
Right Click Domain - New A record  :  
Name: siem.sg.local  
IP Address: 192.168.100.200   
ADD HOST  

##### Forwarder

DNS - "ServerName"  
Right Click - Properties - Forwarders  
EDIT  
Clich here to add  
IP address: 8.8.8.8  
OK  



## 3. Configure kali Purple - for remote management - ssh, rdp, console
Video YouTube - https://youtu.be/pCPVpgsbPOk

This video demonstrates configuration of Kali Purple for ssh and xrdp (unedited)  


refer - https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/301_10:-Kali-Purple-installation 

```
$sudo systemctl enable ssh --now  

$sudo apt update && sudo apt install xrdp 

$sudo systemctl enable xrdp --now 

$sudo wget -P /etc/polkit-1/localauthority/50-local.d https://gitlab.com/kalilinux/documentation/kali-purple/-/raw/main/301_kali-purple/overlays/etc/polkit-1/localauthority/50-local.d/45-allow-colord.pkla
```
## Kali Purple: Enable serial console 

```
$sudo nano /etc/default/grub  
```

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet console=ttyS0,115200n8 console=tty1" 
GRUB_TERMINAL="serial console"  
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1" 
 

$sudo update-grub 

$sudo apt update && sudo apt full-upgrade  
```

## 4. Implement kali Purple SIEM - install elastic stack - elastic , kibana
ref: https://gitlab.com/kalilinux/kali-purple/documentation/-/blob/main/301_kali-purple/installation.txt  

### 4.0 What is Elastic and Kibana (ELK), and how can it be used as a SIEM
Ref: Getting started with Elastic - https://www.elastic.co/webinars/getting-started-elasticsearch?baymax=default&elektra=docs&storm=top-video  
16:00 mins - Install  
17:30 mins - Demo of API Devtools  
27:00 mins - upload a file  
39:40 mins - SQL queries  
42:00 mins - sample data (sample web logs)  
44:20 mins - kibana views  
54:50 mins - time series  
58:30 mins - text analysers  


Ref: Getting started with Kibana - https://www.elastic.co/webinars/getting-started-kibana?baymax=default&elektra=docs&storm=top-video  
06:20 - getting started in cloud (create deployment)  
08:30 - kibana home  
09:00 - sample web logs  
10:00 - tour of kibaba  
10:30 - discover (search and filter data)  
15;00 - filter  
16:30 - field statistics  
17:30 - visualise in lens  
18:30 - dashboards and visualisations  
23:30 - edit visualisations  
25:50 - elastic maps  
30:30 - creating stories  
31:00 - add integrations (data sources)  
32:00 - data visualiser - upload a file .csv  
33:30 - import data .csv  
36:30 - create a dashboard from scratch  
40:30 - elastic maps  
51:30 - add saved search from discover to dashboard data    



Ref: Intro to logging with the ELK stack - https://www.elastic.co/webinars/introduction-elk-stack?baymax=default&elektra=docs&storm=top-video  
06:50 min send logs to elasticsearch  
08:30 min - filebeat syslog server  
XX:XX min - log analytics  
25:30 min - Observability / system metrics (Windows)  
29:50 min - APM agent (Flask - services , service mapping )    
34:45 min - Leveraging elastic security (lock down data , users , roles in elastic)  

Ref: SIEM migration simplified - https://www.elastic.co/virtual-events/siem-migration-simplified

09:00 min - Migration Phases  
10:50 min - Planning and design  
17:00 min - Agile vs Waterfall approach  
19:30 min - Execution and implementation  
28:30 min - Operationalize and refine - Workflows  

Ref: Introduction to Elastic Security: How to shrink MTTR - https://www.elastic.co/webinars/introduction-to-elastic-security-how-to-shrink-mttr  
05:00 min - sources  
08:00 min - Elastic agent with endpoint security  
10:15 - sysmon  
10:30 - endpoint security  
14:00 - Security detection rules  
14:50 - Mitre mapping of elastic agent  
17:30 - Demo _ windows client  
21:30 - Detection rules  
23:30 - Tactics and techniques  
24:40 - Detection rule - create custom query  
28:30 - Q&A  





### 4.1 Install Elasticsearch:  

### NOTE: Ensure you have a static IP configured to 192.168.100.200 on KaliPurple and have internet access before attempting to install ElasticStack

Video YouTube - https://youtu.be/HuHNKIwJGw4

This video demonstrates the installation of elastic stack - Elastic and Kibana into Kali Purple 


```
$sudo apt update && sudo apt upgrade 
sudo bash -c "export HOSTNAME=siem.sg.local; apt-get install elasticsearch -y" 
```
The generated password for the elastic built-in superuser is :  

-[] make a note of the password  

### 4.2 Convert to single-node setup (or replace fqdn name in initial_master_nodes list with IP address):  

``` 

$sudo sed -e '/cluster.initial_master_nodes/ s/^#*/#/' -i /etc/elasticsearch/elasticsearch.yml 
echo "discovery.type: single-node" | sudo tee -a /etc/elasticsearch/elasticsearch.yml 
```

### 4.3 Install kibana

add to /etc/hosts file  
192.168.100.200 siem.sg.local  
```
sudo cat /etc/hosts  
sudo nano /etc/hosts 
```
```
127.0.0.1       localhost
127.0.1.1       siem.sg.local   siem
192.168.100.200 siem.sg.local   siem

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
check using the following commands  
```
dnsdomainname
ping siem.sg.local
```


```

sudo apt install kibana  

sudo /usr/share/kibana/bin/kibana-encryption-keys generate –q 
```
Note the encryption keys generated   

```
echo "server.host: \"siem.sg.local\"" | sudo tee -a /etc/kibana/kibana.yml 

sudo systemctl enable elasticsearch kibana --now
```
### 4.4 Enrol Kibana

#### check the Elastic and Kibana services are running  
```
systemctl status kibana 
systemctl status elasticsearch 
```
#### generate an enrolment token  
```
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana 
```
note the enrolment token

open browser and navigate to :  http://192.168.100.200:5601  
paste the enrolment token
Press - Configure elastic  

#### generate a verification code
```
sudo /usr/share/kibana/bin/kibana-verification-code 
```
note the verification code and enter it into the browser  
press - verify  

login as:  
username: elastic  
Password: (noted password from 4.1) 

You are now logged in to Kibana  

### 4.5 Enable HTTPS for Kibana: 
#### configure kibana for https by generating the required certificates and keys  



ref - https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup.html#generate-certificates

Execute the below commands one at a time - when asked for a password use the enter key 
##### NOTE: subsitute your domain name for the one below ie sg.local to me.local    
```
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca 
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --dns siem.sg.local,elastic.sg.local,siem --out kibana-server.p12 
sudo openssl pkcs12 -in /usr/share/elasticsearch/elastic-stack-ca.p12 -clcerts -nokeys -out /etc/kibana/kibana-server_ca.crt 
sudo openssl pkcs12 -in /usr/share/elasticsearch/kibana-server.p12 -out /etc/kibana/kibana-server.crt -clcerts -nokeys 
sudo openssl pkcs12 -in /usr/share/elasticsearch/kibana-server.p12 -out /etc/kibana/kibana-server.key -nocerts -nodes 
sudo chown root:kibana /etc/kibana/kibana-server_ca.crt 
sudo chown root:kibana /etc/kibana/kibana-server.key 
sudo chown root:kibana /etc/kibana/kibana-server.crt 
sudo chmod 660 /etc/kibana/kibana-server_ca.crt 
sudo chmod 660 /etc/kibana/kibana-server.key 
sudo chmod 660 /etc/kibana/kibana-server.crt 
 
echo "server.ssl.enabled: true" | sudo tee -a /etc/kibana/kibana.yml 
echo "server.ssl.certificate: /etc/kibana/kibana-server.crt" | sudo tee -a /etc/kibana/kibana.yml 
echo "server.ssl.key: /etc/kibana/kibana-server.key" | sudo tee -a /etc/kibana/kibana.yml 
echo "server.publicBaseUrl: \"https://siem.sg.local:5601\"" | sudo tee -a /etc/kibana/kibana.yml 
 
sudo /usr/share/kibana/bin/kibana-encryption-keys generate  
## Copy the generated keys into /etc/kibana/kibana.yml  

```

Note the encryption keys generated  

### Copy the generated keys into /etc/kibana/kibana.yml 
```
$sudo nano /etc/kibana/kibana.yml

paste the generated encryption keys at the bottom of the file
```


```
$sudo systemctl restart kibana  

```
#### Note2: These keys may have been copied in an earler step  

use https to access Kibana - open browser and navigate to :  https://192.168.100.200:5601  
login as:  
username: elastic  
Password: (noted password from 4.1)  


NOTE: if you cant access Kibana try - open browser and navigate to :  https://siem.sg.local:5601  


### create a elastic user(s)

Management > Stack Management > Users - Create user  
Username: steve  
Password: Password1
Priveleges: superuser  
Create user

## 5. Implement Fleet Server and end point protection agent - overview / installation

### Elastic Security and Elastc Defend overview

Ref: Elastic security Overview (8.8) - https://www.elastic.co/guide/en/security/8.8/es-overview.html#es-overview  

Ref: Additional elastic defend information (8.8) - https://www.elastic.co/guide/en/security/8.8/es-overview.html#_additional_elastic_defend_information  
Ref: Ingest data to Elastic security (current) - https://www.elastic.co/guide/en/security/current/ingest-data.html#ingest-data  


### 5.1 Fleet Server and elastic agent overview 
Ref - https://www.elastic.co/guide/en/fleet/current/fleet-overview.html


#### Elastic Agent
Elastic agent adds log monitoring to a host, and can also protect a host from security threats. Operating system data, remote services and hardware can be queried via elastic agent.  
Elastic agents can be deployed to hosts across your infrastructure for host monitoring and security threat detection and prevention.  

The Agent policy can be updated for new data sources and security protections


![Agent diagram](https://www.elastic.co/guide/en/fleet/current/images/agent-architecture.png)

#### Integrations

Elastic integrations connect to external services and systems for example AWS logs or Palo Alto Firewall logs.
Kibana manages elastic integrations.

#### Elastic agent policies

Elastic agent policies determine which elastic agents  run on which hosts in the infrastructure.

The individual logs and metrics selected when adding an integration are saved to an elastic agent policy.   

When a host agent contacts fleet,  the policy is updated

#### Fleet

Fleet centrally manages elastic agents and their policies in Kibana and provides the communications channel to the Elastic agents.
An Agent policy can have multiple agents installed on hosts in the infrastructure. Agents regularly check for changes and updates to agent policy.

#### Fleet server 


Fleet server is a control plane that connects elastic agents to fleet and communicates with them .
Multiple elastic agents can have updated agent policies that are coordinated by fleet server.
[Ref Set up fleet server](https://www.elastic.co/guide/en/fleet/current/fleet-server.html)

Ref set up fleet server -  https://www.elastic.co/guide/en/fleet/current/fleet-server.html


#### Elastic defend

When Elastic defend integration is added to the agent policy it can prevent malware execution on the host

Ref: - https://www.elastic.co/guide/en/security/current/configure-endpoint-integration-policy.html  



### 5.2 Fleet Server Installation 
Note: - Need 8 Gig RAM in Kali Purple VM to Install fleet  

YouTube Video - https://youtu.be/Oq-jCW00s9E

This Video demonstrates installing Fleet server and Elastic agent on Kali Purple  

Ref: Kali Purple Installation - https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/301_34:-Fleet-Server-Installation

#### Install fleet server agent on kali-purple
open a terminal window and update:  
```
sudo apt update
sudo apt upgrade
```
Open web browser:   
From Kibana 
Browse to Management: Fleet -> Add a Fleet server  
Name: Fleet server policy  
URL: https://192.168.100.200:8220  
Press: - Generate Fleet server policy- BUTTON  

#### Install fleet server to a central host 

copy the commands into a terminal window - generated by fleet 

example below  
```
$curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.X.X-linux-x86_64.tar.gz tar xzvf elastic-agent-8.X.X-linux-x86_64.tar.gz cd elastic-agent-8.7.0-linux-x86_64 sudo ./elastic-agent install \ --fleet-server-es=https://192.168.100.200:9200 \ --fleet-server-service-token= ....
```
Wait till files are copied, unpacked and elastic agent is installed and connection is confirmed to fleet server (password is required) , y to run as a service  
it will display:  Elastic Agent has been successfully installed. 

Fleet will then display :Fleet Server connected

Press: -Continue enrolling Elastic Agent- BUTTON 
then close: X   


Browse to Elastic: Fleet -> Agents  
Confirm agent is healthy  


### 5.3 Elastic Defend integration and install windows 10 endpoint protection agent

YouTube video - https://youtu.be/3Oh0QQr-_tI  

This video demonstrates adding Elastic defend integration to Fleet running in Kali Purple elastic stack (Kibana). It also demonstrates installing the Elastic defend endpoint protection agent into a Win10 client and establishing connection between the client agent and Fleet server.


Ref: Elastic defend (manage endpoint protection) - https://www.elastic.co/guide/en/security/current/admin-page-ov.html

Ref: Install and configure the Elastic Defend integration (current) - https://www.elastic.co/guide/en/security/current/install-endpoint.html

YouTube video - Install in windows 11 client -  https://youtu.be/qgc3YPadAq8  
This video demonstrates installing Elastic Defend endpoint agent in Windows 11 client and verifying agent logging to Fleet running on KaliPurple Elastic SIEM  

YouTube video - Install in windows 22 server - https://youtu.be/5KnbByVQxcs  
This video demonstrates installing Elastic Defend endpoint agent in Windows Server 22 and verifying agent logging to Fleet running on KaliPurple Elastic SIEM  


#### Add integration (Elastic defend)
Browse to: Management > Integrations > add security integrations

select: - elastic defend  
Add elastic defend  
Integration name: Windows defend  
Description:   
select: - traditional endpoints
select: Next gen antivirus

##### Where to add this integration
New Hosts - create agent policy  
New agent policy name: Windows defend policy  
Tick: - Collect system logs and metrics  
Press: -Save and continue- BUTTON  

Press: -Add elastic Agent to your Host- BUTTON

#### Add Agent - Enrol in Fleet
Select: - enrolment token  

##### Install elastic agent on your host - Windows

Ensure windows client can connect to KaliPurple and internet  
```
C:>ping 192.168.100.200  
c:>ping 8.8.8.8  
```
Copy the windows PowerShell script to notepad on the windows client  
add --insecure to the end of the script  


Note: the --insecure nees to be added to the execution of the elastic agent install due to the client not trusting the self-signed certificate of fleet server. Alternatively self-signed cert could be copied to win client trusted root.  

Execute the PowerShell script in windows client, PowerShell running as administrator  

Example of PowerShell script generated   
```
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.7.0-windows-x86_64.zip -OutFile elastic-agent-8.7.0-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.7.0-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.7.0-windows-x86_64
.\elastic-agent.exe install --url=https://192.168.100.200:8220 --enrollment-token=UWdaN2VJY0JhdHJlUmMzaGJ0TU46WGlEampQa3BROGltQTc2R3FvWHJzdw== --insecure
```
agent install will ask to be run as a service - answer: y  

On Client Wait for:  
Successfully enrolled elastic agent  
Elastic agent has been successfully installed  

In Kibana wait for:  
Agent Enrolment confirmed  
Incoming data Confirmed 

Close:  and refresh browser

#### Confirm Endpoint protection is configured in Win client agent

The Elastic defend Agent installed in Win client is now communicating with Fleet server in Elastic SIEM.  

Browse to: Elastic > Integrations > Elastic Defend  
Click: - the 1 Agent that is running  

Browse to: Elastic > Fleet > Agents  
View the Windows client and Windows defend policy as healthy   
Click: - Windows Defend policy  
Click: - Windows defend  
Browse to: Elastic > Fleet > Agent policies > windows defend policy > edit integration  

Note: Protection level - prevent  , Windows events collected   

#### check that data is being collected from Endpoint and forwarded to elastic via Fleet
Browse to: Elastic > Discover  
Locate an event with the agent.name field set to your endpoint and expand to view details    

#### Unhealthy fleet server after reboot 

if fleet server is unhealthy after reboot  you can restart the fleet server (elastic agent ) in kali Purple  
ref Restart fleet server- https://www.elastic.co/guide/en/fleet/current/elastic-agent-cmd-options.html#elastic-agent-restart-command  
ref Windows client restart agent - https://www.elastic.co/guide/en/fleet/current/start-stop-elastic-agent.html#start-stop-elastic-agent


```
$sudo elastic-agent restart
# reboot client or
PS C:\Windows\system32>Stop-Service Elastic Agent
PS C:\Windows\system32>Start-Service Elastic Agent

```
#### Incompatable version error (between fleeet server and elastc defend agent)
The version of fleet server must be greater than or equal to the elastic defend integration trying to establish the connection  .
If you get the version error you can update fleet server policy to the latest version to fix.
Note: sudo apt upgrade kibana - does NOT upgrade the fleet agent but will upgrade the elastic defend agent thats deployed to the client.      

Fleet > Agents - Fleet server policy 
- Actions: Upgrade agent   


### 5.4 Configure an integration policy for Elastic Defend
Ref: https://www.elastic.co/guide/en/security/current/configure-endpoint-integration-policy.html#configure-endpoint-integration-policy  
Configure, enable and disable preventions against malware, ransomware, memory threats, and malicious behavior in Elastic defend  

#### Policy settings
Elastic Security > Manage > Policies  - Policy settings  

Note the difference between basic (free) tier and paid subscription:
- Malware protection, detect / prevent (free and paid)  
- Ransomware protection, detect / prevent (paid) 
- Memory threat protection, detect / prevent (paid)
- Malicious behavior protection, detect / prevent (paid)
- Attack surface reduction, credential hardening (free and paid)  
- Event collection (free and paid)  
- Register Elastic Security as antivirus (optional)  (free and paid)  
- Advanced policy settings (optional)

Policy settings - Save  

#### Register Elastic Security as antivirus  
Note: This can register Elastic as official AV for win OS, and will disable defender  
Note: Not supported in Windows Server  
Ref: https://www.elastic.co/guide/en/security/current/configure-endpoint-integration-policy.html#register-as-antivirus  

### 5.5 Elastic secuirty UI  

Ref: Elastic Security UI - https://www.elastic.co/guide/en/security/current/es-ui-overview.html  

### 5.6 Set up security rules and alerts
Elastic Security > explore > hosts - all hosts   
Note hosts displayed 

#### Rules  
ref: Rules - https://www.elastic.co/guide/en/security/current/es-ui-overview.html#_rules  
Elastic Security > Alerts - manage Rules 
- select all rules 
- Bulk actions - enable  

Note – not all rules will be enabled (basic license does not cover machine learning rules)  

#### Alerts  
ref: Alerts - https://www.elastic.co/guide/en/security/current/es-ui-overview.html#_alerts  


Elastic Security > alerts  

View any alerts , investigate high severity alerts 

Elastic Security > explore > hosts
Elastic Security > explore > network 

### 5.7 Cases 
Ref Cases - https://www.elastic.co/guide/en/security/current/es-ui-overview.html#_cases  

### 5.8 Timelines 
Ref Timelines - https://www.elastic.co/guide/en/security/current/es-ui-overview.html#_timelines  

### 5.9 Explore 
Ref: Explore hosts, network, Users - https://www.elastic.co/guide/en/security/current/es-ui-overview.html#_explore  



## 6. Integrate OPNsense firewall logs with KaliPurple SIEM  
### PROTOTYPE NETWORK
Note: KaliPurple VM Networking should be set to internal to communicate with OPNsense firewall prototype network.  

### 6.0 SET OPNsense WAN IP from Console
Note: 6.0 is only required for E208 network, if you use the E108 network you can leave the WAN interface IP to be configured from DHCP and bypass the gordon firewall and get kali updates. This is more convenient when using your VMs in multiple locations ie E208 and home.  
#### Only required for E208 network
Login: root  
Passwd: opnsense  

Enter an Option: 2  
Enter the number of the interface to configure: 3 - WAN (em1 - static)  
Config IPv4 address WAN Interface via DHCP? [y/n] : n  

Enter the new WAN IPv4 adderess .:  
>192.168.108.2XX  
Enter the new WAN IPv4 subnet bit count (1 to 32):
>24  
For a WAN , enter the new WAN IPv4 upstream gateway address:
>192.168.208.1  
Do you want to use it as the IPv4 Gateway [Y/n] y  
Do you want to uest the gateway as the IPv4 name server too? [Y/n] n  
Enter the IPv4 name server... :
>8.8.8.8  
Config IPv6 address for the WAN interface via DHCP6 [Y/n] n  
Enter the new WAN IPv6 address . Press <enter> for none : <ENTER>  

Restore web GUI access defaults [y/N] y  

The new WAN interface will now be displayed  



### 6.0.1 Update OPNsense
Browse to http://102.168.100.1  
System > Firmware > Status  
- Check for updates  

This will switch to:  
System > Firmware > Updates  

Install any updates so that you recieve the message :  
Your packages are up to date.  


### 6.1 Install pfsense integration on fleet server
Ref: - https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/201_30:-Elastic-Agent  

Ref: YouTube - https://youtu.be/LTdAy3RpspM  

Browse to Kibana: Fleet -> Agent policies -> Fleet Server Policy  
Click: - Add Integration - BUTTON  
Search for: Pfsense  
Select  
Click: - Add pfsense - BUTTON  

Set "Syslog host" to "0.0.0.0"  

This will listen to UDP syslog traffic on all interfaces  

Set Syslog port to "9001"  

This is the port the pfsense integration will listen on  

Set Internal networks to  "private"   

Click: -Save and Continue - BUTTON  

Click: - Save and deploy changes - BUTTON  

The integration will now listemn on port 9001   

### 6.2 Configure OPNSense to send syslogs to kali Purple SIEM on UDP port 9001:


Browse to 192.168.100.1: System -> Settings -> Logging - Remote  

[+] Add  
Enabled: tick    
Transport: UDP(4)  
Applications: select all applications  (Leave empty for all)  
Levels: nothing selected  
Facilities: nothing selected  
Hostname: 192.168.100.200  
Port: 9001  
rfc :  
Description: Syslog to kali Purple SIEM   

Click: - Save - BUTTON  
Click: - Apply - BUTTON  

#### 6.2.1 Set default firewall rule to log

Browse to 192.168.100.1: Firewall -> Rules -> LAN  
Edit the " Default allow LAN to any rule "  
Tick  	Log packets that are handled by this rule  
Click: - Save - BUTTON  
Click: - Apply Changes - BUTTON   

#### 6.2.2 Confirm firewall is sending logs using wireshark

Run Wireshark in KaliPurple OR on the Host OS for a bridged interface.

Capture packets on the adapter that connects to OPNsense and confirm logs are being sent to the elasticAgent in kali purple (192.168.100.200) UDP on port 9001.

### 6.3 Confirm Data is ingested to kali Purple SIEM

Browse to Kibana: Discover -> logs   
Search for: data_stream.dataset : "pfsense.log"  

browse to a speed test and generate some NAT traffic  

Browse to Kibana: Discover -> logs  (dropdown)  
Search for: data_stream.dataset : "pfsense.log"  
Click: - Refresh - BUTTON  

Dashboard > Firewall - Dashboard [pfSense]  

Ref: firewall setup - https://www.linkedin.com/learning/firewall-administration-essential-training-2/initial-setup


## 7. Integrate Palo Alto Firewall into Kali Purple SIEM  
### PRODUCTION NETWORK
Note: KaliPurple VM Networking should be set to bridged to communicate with PaloAlto firewall production network.  

Ref Video: https://youtu.be/-vi1mY3MuYc  
Ref Video (older): https://youtu.be/xG_vZrU3WVg

This video demonstrates configuration of the Palo Alto firewall to syslog traffic and threat logs to KaliPurple Elatstic SIEM. Using the Fleet elastic agent integration logs are viewed in Kibana and also in a customisable dashboard.  

### 7.0 PaloAlto Firewall Settings
Dont forget to commit changes  

##### NOTE: Palo Alto Firewall will send syslog logs to port 9001. If PFsense intergration is installed Ensure PFsense integration is switched off as it uses the same port 9001   

#### Timezone setting to Australia/Melbourne
Device > Setup > Management - General Settings  
- Timezone : Australia / Melbourne  

### Latest AV, applications and threats database installed
Device > Dynamic Updates
- Check Now 
- AntiVirus , download and install   
- Applications and threats , download and install     

### Create a Security policy - inside to outside allow
Policys > security - Add  
- General  
    - Name: Inside-to-Outside  
- Source  
    - Source Zone: Inside  
- Destination  
    - Destination Zone: Outside  
- Application
    - APPLICATIONS: Any  
- Service/URL Category  
    - SERVICE: application-default  
    - URL Category: Any  
- Actions  
    - Action setting: Allow  
    - Log Setting: Log at session end  
OK  
### Create a source NAT policy - inside to outside 
Policies > NAT - Add  
- General  
    - Name: NatIinsideToOutside  
    - Nat Type: IPv4  
- Original packet  
    - SOURCE ZONE: Inside  
    - Destination Zone: Outside
    - Destination interface: ethernet1/1  
    - Service : any  
    - SOURCE ADDRESS: any  
    - DESINATION ADDRESS: Any  
- Translated Packet  
    - Source Address translation  
        - Translation type: Dynamic IP and Port  
        - Address Type: Interface Address  
        - Interface: ethernet1/1  
        - IP Address: 192.168.108.2XX
    - Destination Address Translation  
        - Translation type: None
OK  

COMMIT  

test by pinging / browsing from Inside Zone, and check traffic logs (note wait for session end) 
Monitor > Logs > Traffic 



### 7.1 Palo Alto Firewall Syslog configuration 

Note: below needs to be tested  
#### Palo Alto Firewall - Service Route
Device > setup > Services - service route configuration   
IPv4 –   
Service = Syslog   
Source interface = E1/2  
Source address = 192.168.100.1/24 
OK  

#### Palo Alto Firewall - Syslog server profile
Device > Server Profiles > Syslog  
Add - Name : SyslogToKaliPurpleSIEM
Add   
- Name: KaliPurpleSIEM  
- Syslog Server: 192.168.100.200  
- Transport: TCP  
- Port: 9001  
- Format: - IETF  
- Facility: LOG_USER  
OK  

#### Palo Alto Firewall - Log Forwarding profile
For each log type and each severity level or WildFire verdict, select the Syslog server profile and click OK. 

Objects > Log Forwarding  
Add - Name : KaliPurpleSIEMProfile  
    Add  
        Name: LogTraffic  
        LogType: Traffic  
        Filter: All Logs  
        Syslog   
            Add: SyslogToKaliPurpleSIEM  
    OK  
    Add   
        Name: LogThreat  
        LogType: Threat  
        Filter: All Logs  
        Syslog   
            Add: SyslogToKaliPurpleSIEM  
    OK  
OK  


#### Palo Alto Firewall - Security policy configuration to forward logs to kali Purple SIEM 
Configure security policy rules to logg and forward to kaliPurple SIEM  

Policies > Security  
Select Policy rule  
Actions -   
    Log Setting - Tick Log at session End   
    Log forwarding  = SyslogToKaliPurpleSIEM   
OK

#### Palo Alto Firewall - Commit changes  

REF: https://docs.paloaltonetworks.com/pan-os/10-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring    

Video ....   

### 7.2 Palo Alto Firewall Elastic Integration

Browse to Kibana: Fleet -> Agent policies -> Fleet Server Policy  
Click: - Add Integration - BUTTON  
Search for: Palo Alto Next-Gen Firewall 
Select  
Click: - Add Palo Alto Next-Gen Firewall - BUTTON  

![PAN oS Agent](images/PAN-OSagent.png)  

#### Configure Integration 
Integration Name: panw-1

Collect Logs over TCP
Set "Syslog host" to "0.0.0.0"  

This will listen to TCP syslog traffic on all interfaces  

Set Syslog port to "9001"  

This is the port the panw integration will listen on  


#### Where to add this Integration 
Existing Hosts
Agent policy: Fleet server Policy

Click: - Save and Continue - BUTTON
Click: - Save and Deploy Changes - BUTTON

#### 7.2.2 Confirm firewall is sending logs using wireshark

Run Wireshark in KaliPurple OR on the HOS OS for a bridged interface.

Capture packets on the adapter that connects to Palo Alto Firewall and confirm logs are being sent to the elasticAgent in kali purple (192.168.100.200) TCP on port 9001.

### 7.3 Display firewall logs in Kibana 

Discover - logs-*  
Filter = data_stream.dataset : "panw.panos" 
Time = Last 7 days  


Look for data in the following fields:
- @timestamp  
- panw.panos.ruleset   
- panw.panos.action   
- observer.ingress.zone  
- observer.egress.zone  
- source.ip  
- destination.ip  
- destination.port  
- network.transport  
- network.application  
- event.action  
- panw.panos.url.category  
- panw.panos.threat_category  
- panw.panos.threat.name   
- log.level 

Other Useful fields:  

- data_stream.dataset  
- destination.nat.ip  
- destination.nat.port
- event.category  
- observer.egress.interface.name  
- panw.panos.endreason  
- rule.name  
- source.nat.ip  
- source.port
- tags  

Select from above list which fields you want to view  

## 8. DMZ Web server

### 8.1 Set up DMZ Web server in NAT Network
Obtain a copy of the .vmdk file and 

VirtualBox
Tools > Network Manager - Nat Networks  
Create NatNetwork1 - Right click - Properties   
General Options  
Name: DMZ Network  
IPv4Prefix: 192.168.50.0/24  
APPLY  

### 8.2 Set up DMZ Web server Virtual Machine 

VirtualBox
Machine > new  
Name: DMZ Web Server
Folder: (portable drive folder)    
ISO Image: Not selected  
Type: Linux  
Version: Ubuntu 22.04 LTS (64-bit)   

Memory: 4096 MB  

Use an existing Virtual hard disk : xx.vmdk  (copied to portable drive folder)  

Settings > network  
Adapter 1  
Attached to: NAT Network  
Name: DMZ Network   

### 8.3 Update Ubuntu DMZ WEB Server  
VirtualBox  
Start (VM)   

Login: Steve  
Passwd: Password1  

```
$ip a  
$ping 8.8.8.8  
$sudo apt update  
$sudo apt upgrade 
$shutdown now  
```


### 8.4 Configure Host only network to connect DMZ WEB Server to OPNsense firewall  
#### PROTOTYPE NETWORK
Ref: Host only networking - https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/network_hostonly.html  

Ref: Create DMZ in OPNsense - https://homenetworkguy.com/how-to/create-basic-dmz-network-opnsense/  
Ref : Configure OPNsense DMZ - https://getlabsdone.com/how-to-configure-opnsense-dmz-step-by-step/

We can use a host only network in VirtualBox for the third (Adapter 3) DMZ network for OPNsense firewall.
VirtalBox > Network  
- Adapter 1 
    - Attached to: Internal Network  
    - Name: intnet
- Adapter 2  
    - Attached to: Bridged Adapter 
    - Name: Ethernet OR Wifi
- Adapter 3  
    - Attached to: Host Only Adapter  
    - Name: VirtualBox Host-Only Ethernet Adapter #2 

VirtualBox  
File > Tools > network manager  
- Host-only Networks  
    - Create  - VirtualBox Host-Only Ethernet Adapter #2  
        - Adapter 
            - Configure adapter Manually 

            - IPv4 Address: 192.168.50.254  
            - Subnet mask: 255.255.255.0  
        - DHCP server (disable - dont enable)  

Apply  

VirtualBox - OPNsense VM
Machine > settings > Network
- Adapter 3  
    - Enable network Adapter 
    - Attached to: Host Only Adapter  
    - Name: VirtualBox Host-Only Ethernet Adapter #2 
 
OK  

Virtual Box
DMZ server VM - Settings  
Network  - Adapter 1
Enable  
Attached to: Host-only Adapter
Name: VirtualBox Host-Only Ethernet Adapter #2  

VirtualBox  
OPNsense VM  - Start  

VirtualBox  
kaliPurple VM - Start  

Firefox - http://192.168.100.1  
root  
opnsense  
Interfaces > Assignments  

New interface: em2  
Description: DMZ  
[+] Add  
OPT1 - select link  
Enable:  Tick Enable interface  
Description: DMZ  
IPv4 configuration type: Static IPv4  
Static IPv4 configuration  
IPv4 Address: 192.168.50.1 /24  

Save  
Apply Changes  

### 8.4.1 OPNsense DMZ firewall rules (incoming to DMZ)

Firewall > Rules > DMZ  
[+] Add  
Action: pass  
Interface: DMZ  
Direction: in  
TCP/IP Version: IPv4  
Protocol: any  
Source: any  
Destination: any  
Dest port range: any  
log: log packets  

Save  
Apply Changes  

VirtualBox  
KaliPurple VM   

Firefox - http://192.168.50.100  
```
$ping 192.168.50.100   
```

### 8.5 Install Elastic Defend agent in DMZ WEB Server  
Video: Youtube - https://youtu.be/fbHRrl_3Jiw
This video demonstrates Installing Elastic defend agent in Ubuntu Linux DMZ WEB server to send logs and metrics to Kibana running on KaliPurple  

REF: Apache agent - https://docs.elastic.co/integrations/apache  

Virtual Box
kaliPurple VM - kibana

#### Fleet Agent Policy - Linux server policy

Fleet > Agent Policies - Create Agent policy - Linux Server policy  
- Create Agent Policy   

Linux server policy - Add Agent  
Enroll in Fleet  
Copy linux Tar code to clipboard  

#### Install the agent in the linux server via Linux Tar script 
Virtual Box
DMZ server VM - terminal
```
$sudo apt install curl  
$nano elasticAgentScript.sh
```
add --insecure to the last line of the file  
Paste Linux Tar contents into this file and save the file  
Paste the code into a terminal prompt to execute the script with --insecure  

eg. (with your enrolment token)
```
$curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.8.2-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.8.2-linux-x86_64.tar.gz
cd elastic-agent-8.8.2-linux-x86_64
sudo ./elastic-agent install --url=https://192.168.100.200:8220 --enrollment-token=XXXXXXXXXXXXXXX --insecure  
```


Execute the script ./elasticAgentScript.sh  

```
chmod +x elasticAgentScript.sh
./elasticAgentScript.sh
```
the last line will prompt for a do you want to continue: y (yes) , When the agent is sucessfully installed you can go back to the kaliPurple VM  

#### confirm Linux DMZ server logs are sucessfully sent to kibana 

Kibana - Discover  
filter = agent.name : "Your server name here"

eg.
filter = agent.name : "I40-DMZ"

Confirm that logs are coming in from the DMZ server  

#### HOW TO Uninstall elastic agent ONLY if you have issues
Try a re-install after uninstall
```
$sudo /opt/Elastic/Agent/elastic-agent uninstall
```
ref: https://www.elastic.co/guide/en/fleet/current/uninstall-elastic-agent.html

#### Incompatable version error (between fleeet server and elastc defend agent)
The version of fleet server must be greater than or equal to the elastic defend integration trying to establish the connection  .
If you get the version error you can update fleet server policy to the latest version to fix.
Note: sudo apt upgrade kibana - does NOT upgrade the fleet agent but will upgrade the elastic defend agent thats deployed to the client.  

### 8.6 Configure DMZ WEB server for production Network (Palo Alto Firewall)
#### PRODUCTION NETWORK  

#### 8.6.0 Bridge DMZ VM to Ethernet adapter on host
For the production network DMZ web server will be Bridged to the Host Ethernet adapter and Patched to Ethernet1/3 interface on the Palo Alto Firewall.  

Settings > network  
Adapter 1  
Attached to: Bridged  
....
#### 8.6.1 PA FW - inside to DMZ security policy
A security policy will be needed allowing traffic from inside to DMZ.  
Policies > security - Add  
- General   
    - Name: inside-to-DMZ   
- Source  
    - SOURCE ZONE - add inside  
- Destination  
    - DESTINATION ZONE - add DMZ    
- Application - any  
- Service / URL Category - application-default  
- Actions  
    - Action Setting - Action : Allow  
    - Log setting - log at session end  
        - Log Forwarding: KaliPurpleSIEMProfile  

OK   

Note: Security profiles, Applications, source and destination IPs could be used to strengthen this policy.  

#### 8.6.2 PA FW - DMZ to outside security policy

For updates the DMZ WEB server needs to communicate with the Internet (outside)   
Policies > security - Add  
- General   
    - Name: DMZ-to-outside  
- Source  
    - SOURCE ZONE - add DMZ  
- Destination  
    - DESTINATION ZONE - add outside    
- Application - any  
- Service / URL Category - application-default  
- Actions  
    - Action Setting - Action : Allow   
    - Log setting - log at session end   
        - Log Forwarding: KaliPurpleSIEMProfile   

OK     
Note: Security profiles, Applications, source IP could be used to strengthen this policy.  
(eg. applications apt-get , dns)  

You should now be able to update the dmz server from apt repository (outside)  
```
ping 8.8.8.8
sudo apt update
```
#### 8.6.3 PAFW - DMZ WEB Server comnnectivity from Internet
#### PRODUCTION NETWORK  
Suitable security and Destination NAT policy in Palo Alto firewall to enable connectivity to from Internet (outside) to DMZ WEB server  
##### 8.6.3.1 PAFW Security policy - outside to DMZ WEB Server
This policy allows web browsing application traffic from outside to DMZ server with the outside address of 192.168.108.1XX  

Policies > security - Add  
- General   
    - Name: outside-to-DMZ   
- Source  
    - SOURCE ZONE - add outside
- Destination  
    - DESTINATION ZONE - add DMZ   
    - DESTINATION ADDRESS - 192.168.108.1XX
- Application - web-browsing 
- Service / URL Category - application-default  
- Actions  
    - Action Setting - Action : Allow  
    - Log setting - log at session end  
        - Log Forwarding: KaliPurpleSIEMProfile  
    - Profile Setting - Profile Type: None   

OK  
Yopu should now be able to http://192.168.108.1XX to the DMZ WEB server from outside (E108) network PC.   


##### 8.6.3.2 PAFW NAT policy - destination NAT from outside to DMZ WEB server
This destination NAT policy allows use of the 192.168.108.1XX outside address for the DMZ WEB server at 192.168.50.100 for http traffic  
Policies > NAT - Add  
- General
    - Name: dstNat-outside-dmz  
    - NAT Type: ipv4
- Original Packet  
    - SOURCE ZONE: outside
    - Destination zone: outside
    - Destination Interface: any
    - Service : Service-http
    - SOURCE ADDRESS : any
    - DESTINATION ADDRESS: 192.168.108.1XX  
- Translated packet
    - Source Address translation: None
    - Destination Address translation:
        - Translation Type: Static IP  
        - Translated Address: 192.168.50.100
        - Translated Port:

OK  

### 8.8 DMZ WEB Server remote management testing 

#### 8.8.1 XRDP Install
ref: https://www.digitalocean.com/community/tutorials/how-to-enable-remote-desktop-protocol-using-xrdp-on-ubuntu-22-04

```
$sudo apt install xrdp -y
$sudo systemctl status xrdp
$sudo systemctl start xrdp

```

#### 8.8.2 SSH install
```
$sudo apt-get install openssh-server
$sudo systemctl enable ssh  
```


#### 8.8.3 test remote management 
Install the following applications into windows client  
Putty 

Test remote desktop to DMZ WEB Server using rdp from windows client  
Test remote SSH to DMZ WEB Server using putty from windows client  

#### 8.8.4 OPTIONAL UFW security
Additional layers of security can be added by configuring Linux firewall (iptables) using ufw on the Linux DMZ WEB server endpoint. 

Sample security policys  
- http , https from LAN and WAN (tcp ports 80,443)  
- SSH remote mgt from LAN IP address(s)  (tcp port 22)  
- RDP remote desktop from LAN IP address(s) (tcp port 3389)  
- DNS from WAN ip 8.8.8.8 (UDP port 53)  


REF: UFW - Uncomplicated Firewall - https://wiki.ubuntu.com/UncomplicatedFirewall  
REF: UFW manual - https://manpages.ubuntu.com/manpages/jammy/en/man8/ufw.8.html  
REF: IPtables - http://www.computingstudies.net/tutorials/iptables/  

Configure and test UFW for the following ports  
80, 443, 22, 3389 , 53    
example 
sudo ufw allow from your_local_ip/32 to any port 3389   
```
sudo ufw allow from 192.168.100.200/32 to any port 3389

```

### 8.10 Bridge Ubuntu DMZ WEB Server to PA firewall
#### PRODUCTION NETWORK
Cable PC up to network switch using patch cable

VirtualBox 
Settings > network  
Adapter 1  
Attached to: Bridged adapter
Start (VM)  
### 8.10.1 DMZ WEB Server comnnectivity to KaliPurple SIEM

Place suitable security policies in PA firewall to enable connectivity to from DMZ WEB server to kali Purple SIEM  
from DMZ WEB check connectivity   
```
$ping 192.168.100.200
```
from KaliPurple check connectivity  
```
$ping 192.168.50.100
```

### 8.11 Change ubuntu DMZ WEB server static Ip address (should not be required)

```
sudo nano /etc/netplan/99_config.yaml
```
```
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      addresses:
        - 192.168.50.100/24
      gateway4: 192.168.50.1
      nameservers:
          search: [mydomain, otherdomain]
          addresses: [208.67.222.222, 8.8.8.8]

```
```
sudo netplan apply  
```

### 8.12 OPTIONAL - Configure OPNsense firewall for Production network
#### OPTIONAL PRODUCTION NETWORK Using OPNsense Firewall
Note: This is optional if you do not have access to a Palo Alto Firewall..  
For the production network DMZ web server VM and OPNsense firewall VM run of the same physical host. We user the hosts WIFI adapter for WAN (ISP) connection , and the hosts Ethernet adapter for the Production LAN and connect it to your (core) switch. The DMZ WEB server can connect to OPNsense firewall DMZ interface via either Virtual box internal network OR Host only network.  

summary:  
F/W - Zone - VM network  
em0 - LAN  - bridged to Ethernet adapter  
em1 - WAN  - bridgerd to WIFI adapter  
em2 - DMZ  - host only network # 2 OR internal network  

Shutdown OPNsense VM
login:root  
Passwd: opnsense  

Option 5  
y  



VirtualBox > OPNSense VM - Settings  
Network - Adapter 1 
Enable   
Attached to: Bridged Adapter  
Name: Realtek PCIe GbE Family Controller  (Ethernet adapter)

Network - Adapter 2  
Enable  
Attached to : Bridged Adapter  
Name: Wireless Adapter  

Network - Adapter 3  
Enable  
Attached to: Host-only Adapter
Name: VirtualBox Host-Only Ethernet Adapter #2 


Patch (Cable) Ethermet port on PC to Switch Port
Connect wifi to ISP ()

Start OPNsense VM
Shutdown OPNsense VM
login:root  
Passwd: opnsense  

Option:  


on Host pc browse to http://192.168.100.1  
login: root
Passwd: opnsense  

Bridge KaliPurple VM to Ethernet adapter and patch (cable) to switch port  

From KaliPurple VM open terminal and test networking connectivity:  

```
$ping 192.168.100.1
$ping 8.8.8.8
$ping 192.168.50.100
```

From DMZ WEB server VM open terminal and test networking connectivity:  
```
$ping 192.168.50.1
$ping 8.8.8.8
$ping 192.168.100.200
```

Production Firewall is now functional  
### 8.12.1 OPTIONAL - OPNsense NAT and Security policy for DMZ access from WAN 
#### OPTIONAL PRODUCTION NETWORK Using OPNsense Firewall
Ref : Configure OPNsense DMZ - https://getlabsdone.com/how-to-configure-opnsense-dmz-step-by-step/  


## 9.0 Configure and Test Network

### 9.1 Configure and test domain user and computer accounts

### 9.2 Configure and test domain group policies
Minimum Password Policy:    
- Enforce password history
- Maximum password age  
- Minimum password length  
- Password must meet complexity requirements  

Ref: Group policy Password policy - https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/password-policy 

Ref: Windows security baselines - https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines  

### 9.3 Configure and test elastic user accounts and role(s)
In the elastic web gui, search for "roles" in the search bar, this brings you to a page where you can make new roles.  
Additionally, searching for users lets you make new users.

### 9.4 Configure and test management VLAN / host isolation
Ref - Network security lab 4.4.8 Packet Tracer - Configure Secure Passwords and SSH  
Ref - Network security lab 14.3.1 Packet Tracer  Implement port security  
Ref - Network security lab 6.7.12 Packet Tracer - Configure Cisco Devices for Syslog, NTP, and SSH Operations  
Ref - Network security lab 14.9.11 Packet Tracer - Layer 2 VLAN Security  
Ref - Network Security lab 7.2.3.4 Lab - Configuring and Verifying VTY Restrictions (using ACL)   
Set up switch VLAN for remote management via SSH    
Set up switch port security  
- disable unused switch ports  
- enable switchport security  (for 3-4 sticky mac addresses )  

Set up ACL on VTY to restrict management IPs to 192.168.100.X  
Optionally Set up NTP  (from windows server time service or ntp.org)  




### 9.6 Configure and test domain DNS forwarding
Server manager: Tools - DNS Manager  
DNS - Server - properties  
Forwarders - Edit  
Ip Address: 192.168.100.1, 8.8.8.8  
OK  


### 9.7 Optional Install ufw on KaliPurple
```
$sudo apt -y install ufw
```
###

### 9.8 Install ssh / rdp on DMZ WEB server for remote management
```
sudo apt install ssh   
sudo systemctl enable ssh --now  
sudo apt install xrdp  
sudo systemctl enable xrdp --now  
```

### 9.10  PA Firewall - Remote Management of DMZ server 
ref: Video PA firewall NAT and security Policies - https://youtu.be/kE_BOtEe0MY  
Ref: PaloAlto Firewall - LAB: PAN10_EDU210_Lab_05 Configuring Security Policy Rules and NAT Policy Rules  

#### 9.10.1  PA Firewall - Enable remote Management  (ports 80,22,3389,3306) to DMZ server from the internet Zone (E208 PCs)  
Enable remote management access to DMZ server from the Internet Zone from only 1 specific IP address (E208 PC) – for SSH , Remote Desktop Protocol (RDP) , and MySQL remote administration (port 3306).  


Modify firewall security policy to:  
1.	allow all access to 192.168.108.1XX  on port 80 only  
2.	allow ssh , RDP and port 3306 to 192.168.108.1XX  from a single Internet PC IP address in E208 Network   

Modify NAT policy:  dstNAT-outside-dmz  

Source Zone: Outside  
Destination Zone: Outside 
Destination Interface: Any  
Source Address: Any  
Destination Address: 192.168.108.1XX  
Service: any   (change to any to allow SSH, RDP, 3306)  

Source Address translation: None  
Destination Address Translation:  
Translation Type: Static IP  
Translated address: 192.168.50.100  

OK  


Add a security policy:  outside-to-DMZ-REMOTE-MGT  
Source Zone: Outside
Source address: 192.168.108.YY  (this is the remote maganemnt IP address from a single IP) 
Destination Zone: DMZ  
Destination Address: 192.168.108.1XX  
Application: SSH, ms-rdp, 3306 
Service: Application-default  
Actions 
- Action Setting - Action : Allow
- Log setting - log at session end
- Log Forwarding: KaliPurpleSIEMProfile
- Profile Setting - Profile Type: None

#### 9.10.2  PA Firewall - test remote management of DMZ server from a single public IP address 

Test rdp from an E208 PC on E208 Network to 192.168.108.1XX  
Test ssh (putty) from an E208 PC on E208 Network to 192.168.108.1XX  
Run nmap from an E208 PC on E208 Network to 192.168.108.1XX  

#### 9.10.3 Remote manage MySQL server on DMZ WEB server

Create MySQL remote user account  
http://localhost/phpmyadmin  
New > User accounts  
User account 'I40Remote'@'%' (any Host) OR 'I40Remote'@'X.Y.Z.A'  
grant SELECT Data privileges only  
NOTE: I40Remote needs to use the Authentication plugin = auth_plugin='mysql_native_password'  
Config MySQL and firewall rules for remote connections on port 3306  
https://www.digitalocean.com/community/tutorials/how-to-allow-remote-access-to-mysql  
$sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf  
Change the bind address – note security issues – this should be a static IP in production  
eg.  
$ip a  
192.168.50.100 - server ip address  
change the bind address to server address ie 192.168.50.100  
This will allow connectivity from a remote windows host that will be analyzing data  

You will now be able to make remote connections to MySQL on port 3306 from your E208 PC using MySQL workbench  

#### 9.10.4 PA Firewall - Manage DMZ WEB server from LAN IP(s)
Add a security policy:  secPol-inside-to-DMZ  
Source Zone: LAN  
Source address: Any - (add specific management IPs)  
Destination Zone: Extranet  
Destination Address: 192.168.50.100  
Application: Web-Browsing (add for part 2 - to allow ping, SSH, ms-rdp, mysql, web-browsing [3306])  
Service: Application-default  
Action: Allow  

Test using putty , ms-rdp client, mysql_workbench, web browser   

#### 9.10.5  PA Firewall - Src nat and sec pol for DMZ server to outside

##### PA Firewall - NAT - DMZ to outside src nat rule  
Name: srcnat-DMZ-Outside  
Original packet
 - source zone: DMZ  
 - Destination zone: outside  
 - Dest interface: ethernet1/1
 - Service : any  
 - source address: any  
 - destination address: any  
 translated packet  - Source address translation:  
 - Translation type: Dynamic Ip and port  
 - Address type: interface address   
 - Interface: ethernet1/1
 - Ip Address: none



##### PA Firewall - Security policy - DMZ to outside Security policy 
Name: secpol-DMZ-to-outside  
Source Zone: DMZ  
Destination Zone: outside  
Applications: any  
Service : application-default  
Url category: any  
Action: allow  
Profile Setting: TBA  
Log setting: log at session end  

NOTE: Security policy should ideally specify what applications are allowed eg (apt-get, dns etc)  

#### 9.10.6 PA Firewall - DMZ WEB server to inside (WEB server agent connect to SIOEM)
Add a security policy:  secPol-DMZ-to-inside 
Source Zone: DMZ 
Source address: 192.168.50.100 
Destination Zone: inside 
Destination Address: 192.168.100.200  
Application: any  
Service: (new service: SIEM-agent:tcp port 9200)
Action: Allow  

Test that DMZ WEB server agent is sending logs to Elasic SIEM   

#### 9.10.7  PA Firewall IDS / IPS - Add security profiles to security policies to log and block threats
Ref: What is an IPS - https://www.paloaltonetworks.com/cyberpedia/what-is-an-intrusion-prevention-system-ips  

ref lab: PAN10_EDU210_Lab_15 Implementing Day-One Best Practice Configuration  


Adding security profiles to security policies will enable IDS / IPS in the palo Alto firewall and generate threat logs when the IDS / IPS discovers a threat.  
IDs / IPS security profiles include:  
- AntiVirus profiles  
- Anti-Spyware Profiles  
- Vulnerability Protection Profiles  
- URL Filtering Profiles  
- Data Filtering Profiles  
- File Blocking Profiles  
- WildFire Analysis Profiles  
- DoS Protection Profiles  
- Zone Protection Profiles  
- Security Profile Group  

Ref: Security Profiles - https://docs.paloaltonetworks.com/pan-os/10-2/pan-os-admin/policy/security-profiles  

### 9.11 Cisco switch logging to Fleet
ref: cisco logging - https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SysMsgLogging.html  
Ref: Cisco IOS elastic integration - https://docs.elastic.co/en/integrations/cisco_ios  

Ref: Youtube Configuring Cisco switch syslog logging to fleet integration  - https://youtu.be/H9iYJ4ln_Jo  
This video demonstartes configuring and testing a Cisco layer 3 switch to send logs to Elastic Security SIEM using syslog and Fleet cisco_ios integration  


#### 9.11.1 Enable switch logging

```
configure terminal
service timestamps log datetime
service sequence-numbers
logging 192.168.100.200
logging trap 4
logging facility auth
```

#### 9.11.2  Cisco IOS elastic integration
Add Integration  

set logging to 0.0.0.0 udp port 514  
disable tcp    
this is the default for cisco switch  
#### 9.11.3 Enable switch Port Security logging
```
configure terminal
interface G0/0/X
switchport mode access
switchport port-security violation shutdown
exit
show port-security interface G0/0/X
```

#### 9.11.4 Discover
ensure cisco switch has correct date time for logs from ntp server or set the clock date time to current date time   

a login / logout shoutd trigger the switch to log to syslog (SIEM)  
Search  (last 15 minutes)
data_stream.dataset : cisco_ios.log
## 9.12 OPTIONAL OPNsense - Configure and test WAN to DMZ security and NAT policy to access DMZ WEB server from Internet
Sample security policys  
- http , https from LAN and WAN to DMZ (tcp ports 80,443)  
- SSH remote mgt from LAN IP address(s) to DMZ (tcp port 22)    
- RDP remote desktop from LAN IP address(s) to DMZ (tcp port 3389)  
- DNS from WAN ip 8.8.8.8 to DMZ (UDP port 53)  

- http, https, dns from LAN to WAN  
#### PRODUCTION
NAT policies  
- Source NAT from LAN to WAN - dynamic Ip and port  
- Destination NAT from WAN to DMZ - static IP (use WAN IP)  


Ref: OPNsense Firewall - https://homenetworkguy.com/how-to/ways-to-secure-access-to-opnsense-and-your-home-network/  
Ref: OPNsense firewall - https://youtu.be/u7IXficvT1c  


### 9.13 OPTIONAL - Configure and test Suricata on OPNsense 
####  OPTIONAL PRODUCTION NETWORK Using OPNsense Firewall
Note: This is optional if you do not have access to a Palo Alto Firewall..   
ref: Sudicata IDS - https://www.linkedin.com/learning/ecih-cert-prep-certified-incident-handler-v2-212-89/using-suricata-ids  
ref: https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/201_40:-Beats  
ref: https://www.youtube.com/watch?v=QmFxpd39v6c&t=80s   

Note : Give OPNsense at least 8 Gig RAM and ensure you have latest version os and updates installed prior to commencing



#### Allow SSH / SCP on OPNsense  
KaliPurple VM  
http://192.168.100.1

System > Settings > Administration - Secure Shell  

Secure Shell  
Check - Enable secure shell  
Check - Permit Roort user login  
Check - permit password login  
Save  
Apply  

KaliPurple VM
```
$ssh root@192.168.100.1  
Choose option 8 - Shell
# mkdir /etc/pki
# mkdir /etc/pki/root  
# cd /etc/pki/root 
# ls

```

Open another terminal window in kaliPurple  
```
$sudo su  
$cd /etc/elasticsearch/cert                 (/certs )   
$scp http_ca.crt root$192.168.100.1:/etc/pki/root/  

```


#### Install filebeat on OPNsense firewall  
In other termial window  
```
#ls -l
#opnsense-code ports
#cd /usr/ports/sysutils/beats8
#make install

```
[X] FILEBEAT  
OK  

OK  


```
#pkg install nano  
#nano /usr/local/etc/beats/filebeat.yml  

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["192.168.100.200:9200"]

  # Protocol - either `http` (default) or `https`.
  protocol: "https"
  # List of root certificates for HTTPS server verifications
  ssl.certificate_authorities: ["/etc/pki/root/http_ca.crt"]

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  username: "elastic"
  password: "<secret>"
  
  
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "https://192.168.100.200:5601"
  ssl.verification_mode: none

```
Note: paste in your elastic password in place of "<secret>"  

```
#cp /usr/local/share/examples/beats/filebeat.modules.d/suricata.yml.disabled /usr/local/etc/beats/filebeat.modules.d/
#cp /usr/local/share/examples/beats/filebeat.modules.d/system.yml.disabled /usr/local/etc/beats/filebeat.modules.d/
#cp /usr/local/share/examples/beats/filebeat.modules.d/nginx.yml.disabled /usr/local/etc/beats/filebeat.modules.d/
#cd /usr/local/etc/beats/
#filebeat modules list
```
> Enabled:
> 
> Disabled:
> suricata
> system

```
#filebeat modules enable suricata nginx system
```
> Enabled suricata
> Enabled nginx
> Enabled system
```
#filebeat modules list

```

> Enabled:
> suricata
> nginx
> system

> Disabled:
> squid

```
#nano filebeat.modules.d/suricata.yml
# Module: suricata
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.17/filebeat-module-suricata.html

- module: suricata
  # All logs
  eve:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:
```
```
#filebeat test output -c /usr/local/etc/beats/filebeat.yml
#filebeat test config -c /usr/local/etc/beats/filebeat.yml

#filebeat setup -e
```


enable and start fileveat
```
#sysrc filebeat_enable="YES"
#service filebeat start
```


https://192.168.100.1
services > Intrusion detection > Administration  - 

Settings  
Check - enabled  
Check - promiscuous mode  
Check - enable syslog alerts  

Download  
check -  all  



#### test Suricata

from kaliPurple run an nmap scan  


elastic > diacover - filebeat-*   
filter -  event.dataset : suricata.eve  

look for event 


#### setup dashboards 19:00
Uncomment the dashboards line in filebeat.yml  
```
#nano /usr/local/etc/beats/filebeat.yml

setup.dashboards.enabled: true
```


Kibana Dashboards -  
[Filebeat Suricata] Events overview
[Filebeat Suricata] Alerts overview



## 10.0 Establish a Network Security Baseline
Refer to ASD essential 8  , select appropriate maturity level (1)  
Ref: https://www.cyber.gov.au/resources-business-and-government/essential-cyber-security/essential-eight/essential-eight-maturity-model  
Ref: Windows security baselines - https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines  
### 10.1 Windows client application control
Ref: WDAC and Smart Application control - https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/wdac  
Ref: WDAC Vs Applocker - https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/wdac-and-applocker-overview  


### 10.2 Application Patching  
DMZ server (exposed service patching - regularity)  
Vulnerability scanning  Regularity specified  
Firewall profile - Vulnerability threat detection  
Client applications  
SIEM  
### 10.3 Macro settings  
    

#### Firewall security profile to block office files
PA firewall security profile block office files downloaded from internet   

#### Block Office macros via group policy
Ref: Block Office macros via group policy - https://4sysops.com/archives/restricting-or-blocking-office-2016-2019-macros-with-group-policy/

Ref: Download Group policy administrative templates - https://www.microsoft.com/en-us/download/details.aspx?id=49030


Group policy macro settings  
Computer or User Configuration > Policies > Administrative Templates > Microsoft Office 2016 > Security Settings> "Disable VBA for Office applications"  

and  

User configuration > Policies > Administrative templates > Microsoft Word 2016 > Word options > Security > Trust Cente> "Disable all macros with notification"  

### 10.4 Application hardening   
DMZ server  
Group policy browser settings  
REF: Java group policy disable - https://www.thewindowsclub.com/disable-change-java-permissions-windows-group-policy-editor  


### 10.5 restrict admin privileges
Windows Server 22  - priveliged account policies  
DMZ server  
Palo ALto Firewall  
OPNsense Firewall   
Cisco Switch  
Windows Clients  

### 10.6 Patch operating systems
ALL OS - regulatity specified  
Vulnerability scanning  
Unpatched vulnerabilities identified  

### 10.7 MFA  
ref: Auth security maturity model - https://danielmiessler.com/p/casmm-consumer-authentication-security-maturity-model/  

### 10.8 regular backups
Windows Server 22  
DMZ server  / Database  
Regularity specified  



### 10.9 Network Segmentation and segregation 
Firewall zones  
Isolate VLAN for management traffic    
Ref: Network Segmentation and segregation - https://www.cyber.gov.au/resources-business-and-government/maintaining-devices-and-systems/system-hardening-and-administration/network-hardening/implementing-network-segmentation-and-segregation  


### 10.10 firewall security policies  / profiles  
list all policies and profiles implemented  
- PA FW  
- OPNsense FW  
- UFW - ubuntu DMZ Web server  
- Windows firewall clients  


## 11. Testing RBT activities

Ref: RBT best practice RBT Video recording offsec (computingstuies login) - https://drive.google.com/file/d/1hleneAd2eVWr3qzOOVi54KqsVazBtdaB/view?usp=drive_link 


Ref: Video - Isolating hosts - https://www.youtube.com/watch?v=6pX20491G8Q  
Ref Video: - Mac reverse shell payload test (Mac) - https://www.youtube.com/watch?v=IGfgOzeMmkY  
ref: Video - Detect bad word documents - https://www.youtube.com/watch?v=ApIzTSNYwXQ  


Split your team into Red and Blue team roles.  
Develop and implement suitable RBT penetration and vulnerability tests from both inside and outside the network using suitable Kali Linux tools. Monitor testing using: SIEM/SOC events and alerts; firewall logs and document the test procedures, test results and SIEM logs and alerts detected.  
Document and adjust any firewall rules , patching , OS and application hardening as appropriate.  
You may want to reduce the strength of some security policies to generate some alerts in the first instance and then tighten the security policies / profiles.  
You may consider adding new or customising SIEM rules to capture appropriate events OR reduce noise.  
Optionally you could use Kali Autopilot to automate pen testing.   

### 11.1 Elastic security alerts and rules
ref: https://www.linkedin.com/learning/a-complete-guide-to-kali-purple/monitoring-alerts-with-elkstack  
Ref: https://www.techtarget.com/searchsecurity/feature/Elastic-Stack-Security-tutorial-How-to-create-detection-rules 

Ref: Elastc security 101 - https://www.elastic.co/videos/training-how-to-series-security 

Ref: how to enable detection rules - https://www.elastic.co/videos/training-how-to-series-security 
Ref: End to end incident response with Elastic security - https://youtu.be/C5bYHCjF4qI   

Security > Alerts - Manage Rules  
Add Elastic rules  - Install All , Go back to Installed Elastic rules  

Rules must be enabled  , view Potential Successful SSH Brute Force Attack  - enable  

hydra attack against linux target via ssh brute force password guessing  



### 11.2 Enumeration with NMAP scans
Ref (Cisco EH - 3.2.6)  
DMZ server:  
```
nmap -sn
nmap -sT
nmap -O 192.158.50.100
nmap -p80 -sV -A 192.158.50.100
```

Ref (Cisco EH - 3.3.6) 
windows server SMB shares:  
```
nmap -A -p139,445 192.168.100.100    # SMB ports
nmap --script smb-enum-users.nse -p139,445 192.168.100.100
nmap --script smb-enum-shares.nse -p445 192.168.100.100
```
### 11.3 Vulnerability scans
ref: (Cisco EH - 6.1.8)

```
ping -c5 192.168.50.100
nmap -sV 192.168.50.100
sudo nmap -O 192.168.50.100
nmap -sV --script vulners --script-args mincvss=4 192.168.50.100
```
https://nvd.nist.gov/vuln/search  


GVM - 
```
sudo gvm-check-setup
sudo gvm-stop
```
#### using GVM vulnerability scanner  
ref: (Cisco EH - 6.1.8)
```
sudo gvm-start

```
https://127.0.0.1:9392  
Username: admin  
Password: kali  
 
Scans -> Tasks  
 IP address or hostname: 192.168.50.100  
Download the report by clicking the Download Filtered Report button  
```
sudo gvm-stop
```
https://www.cve.org  
Find: CVE-.........   

### 11.4 Nikto WEB vulnerability scan 
Ref: (Cisco EH - 6.1.7)

```
nikto --help
nikto -h 192.168.50.100
nikto -h 192.168.50.100 -ssl
nikto -h 192.168.50.100 -o pentest.htm
nikto -h 192.168.50.100 -o scan_results.txt -Format csv

```
https://cwe.mitre.org/about/
https://cve.mitre.org/


### 11.5 use OWAP ZAP
ref(Cisco EH - 6.12.13)  
kali menu -> zap  

Automated scan  
URL to Attack: 192.168.50.100  
Attack  

Alerts tab  
### 11.X Feel free to add any additional Penatration tests 
Feel free to add any additional Penetration tests that you believe could be useful to uncover potential vulnerabilities in any of the OS / application systems or to test any of the security policies and security profiles that you have implemented.   
### 11.6 Reporting
ref (Cisco EH - 9.1.9) (Cisco EH - 9.2.7)  

ref: sample public pen test reports - https://github.com/santosomar/public-pentesting-reports  
```
nikto -h 192.168.50.100 -o pentest.htm
sudo gvm-start
```
GVM -> scans -> reports - results  

look at priority of vulnerabilities found (if any)  

suggest reccomendations  

### 11.7 OPTIONAL - Kali AutoPilot
Ref: kali Autopilot User Guide - https://gitlab.com/re4son/kali-autopilot/-/wikis/User-Guide  
Ref: kail Autopilot Hub - https://gitlab.com/kalilinux/kali-purple/purple-hub  

Ref: KaliAutopilot Youtube Video - https://youtu.be/rV75CM9tjcw  
Ref: Kali Autopilot - https://gitlab.com/kalilinux/kali-purple/documentation/-/wikis/O_1001_20:-Kali-Autopilot  

#### 11.7.0 Installing Kali AutoPilot
Run kaliPurple / Kali VM shell  
```
sudo apt update  
sudo apt upgrade   

sudo apt install kali-autopilot  
mkdir autopilot  
cd autopilot  
git clone https://gitlab.com/kalilinux/kali-purple/purple-hub.git  
kali-autopilot  
```

#### 11.7.1 import dvwa script sample 

Attack Scripts > import - ./purple-hub/dvwa/dvwa_kali-autopilot.json    

#### 11.7.2 set up a new test script test01.py

ref: Youtube - https://youtu.be/rV75CM9tjcw  
This Video demonstrates using Kali Autopilot to generate a basic python scripted NMAP test by creating , modifying and generating the test script. The script is then run and sequence controlled via the mutex api.   

Script: test01  
Attack Scripts > add  


#### 11.7.3 code script test01.py
Variables for test01  
Name    Value  
subnet  192.168.100.0/24  

Scripted attack sequence  
STAGE   1   Scanning
OS          nmap -sn {subnet}

Attack scripts - SAVE  
Scripted attack sequence  - Generate  

script will be saved in ./test01/test01.py  
OK
#### 11.7.4 ensure pip and python packages are installed
```
python3 -m pip install --upgrade pip
python3 -m pip install paramiko 
python3 -m pip install pexpect 
```
#### 11.7.5 run script
script will be saved in ./test01/test01.py  
open a terminal window and run the script  
```
~/kali-autopilot/test01/test01.py
```
#### 11.7.6 Control script stages via Mutex interface  
ref: web api - https://gitlab.com/re4son/kali-autopilot/-/wikis/web-api

advance the Stage with the mutex interface  

http://192.168.100.200:443  
login:  offsec  
passwd: offsec  

http://192.168.100.200:443/set  

Start stage 1 of test  
http://192.168.100.200:443/set?mutex=1  


#### 11.7.7 Make a copy of test01 called test02 and edit

Attack Scripts:   
Highlight test01  
Script:test02  

Attack scripts - COPY

Add code to scripted attack sequence  

Variables:  
Name    Value  
subnet  192.168.100.0/24  
kp_ip   192.168.100.200  

Scripted attack sequence  
STAGE   1   Scanning
OS          nmap -sn {subnet}
OS      nmap -PS -sV {kp_ip}

Attack scripts - SAVE  
Scripted attack sequence  - Generate   

##### 11.7.8 run script test02.py
```
~/kali-autopilot/test02/test02.py
```

##### 11.7.9 advance the stage of test02.py
Start stage 1 of test02.py  
http://192.168.100.200:443/set?mutex=1  

#### 11.7.10 continue building your test scripts  

**END**
