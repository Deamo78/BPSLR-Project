DOCUMENTATION


                                                Day One
                                            
Name: *Ryan Saunders*  
Date: *23/05/2024*  


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: 799EQmwF
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller


**Tasks Assigned:**

- Install Windows 10 Client
Make sure its connected to the Windows22 Server

- Install Kali Purple (Internal)
Make sure its connected Internally

- Connect Host PC to PaloAlto
Make sure its connected to the internet


**Tasks Completed:**
- Installed Windows 10 Client 
![day1win10c]
(D:/Ryan CSOC/Screenshots/Day1/day1win10c.png)  
It isnt connected to the internet or to the server yet. The server isnt configured completely 

- Installed Kali Linux Purple
![day1kaliInt]
(https://github.com/Deamo78/BPSLR-Project/blob/ryan/day1kaliInt.png)
Used Step 2.4 as referenced from Steves GitHub.
It has internet connection and is able to ping the server, as well as the switch (PaloAlto Firewall).


**Commands Used:**
```
$sudo apt update  
$sudo apt upgrade -y

$sudo nano /etc/network/interfaces
    
    auto eth0
    iface eth0 inet static
        address 192.168.100.250/24
        gateway 192.168.100.1
        dns-nameservers 8.8.8.8

$sudo nano /etc/resolv.conf
nameserver 8.8.8.8

$ping 192.168.100.254
$ping 8.8.8.8
```


                                                Day Two
                                            
Name: *Ryan Saunders*  
Date: *28/05/2024*  


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    -Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: Password2
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Tasks Assigned:**
- Windows 10 Client
Make sure its connected to the Windows22 Server

- Kali Purple (Internal)
Make sure its **still** connected Internally

**Tasks Completed:**
- Connected the Windows 10 Client PC to the Windows Server 22
- I can now ping everyone else within the server
- Changed the password to Windows 10 Client because it was inconvenient.
- Moved the VMs to the SSD on the host. It was just too damn slow on the HDD
  
**Commands Used:**
No commands were used this time. except for:
```
ping 192.168.100.10
#Default Gateway
ping 192.168.100.1
#Server
ping 192.168.100.100
```
IP address 192.168.100.10 was Brodys Win 10 client PC. We had to make sure Network sharing was turned on because its usually turned off by default.


                                                Day Three

Name: *Ryan Saunders*  
Date: *04/06/2024*


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: password1
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Tasks Assigned:**
- Windows 10 Client
Make sure its connected to the Windows22 Server
Enroll to elastic search
Install Guest additions.

- Kali Purple (Internal)
Make sure its **still** connected Internally
Start Blue Team testing activities

**Tasks Completed:**
- Installed Guest Additions for convenience sake
- Also used a powershell scripted provided by Seb to enrol myself to the elastic user. Script shown below.
- Accessed Elastic Search through Kali Internal and setup:    
Security > Alerts - Manage Rules  
Add Elastic rules - Install All , Go back to Installed Elastic rules  
Rules must be enabled , view Potential Successful SSH Brute Force Attack - enable  
hydra attack against linux target via ssh brute force password guessing  

**Commands/Scripts used:**  
PowerShell Script used
```
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.4-windows-x86_64.zip -OutFile elastic-agent-8.13.4-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.13.4-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.13.4-windows-x86_64
.\elastic-agent.exe install --url=https://192.168.100.200:8220 --enrollment-token=TWtxMXg0OEJLMDFVWTVTamhQaXU6MDhfVi0wWXRRYXU0azdVVVRBbjhNUQ== --insecure
```

                                                Day Four

Name: *Ryan Saunders*  
Date: *06/06/2024*


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: password1
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Tasks Assigned:**
- Kali Purple (Internal)
Installing Elastic and kibana  
Testing RBT activities  
Configuring Elastic security alerts and rules  

**Tasks Completed:**
- Installed Elastic and Kibana  
Encyption keys:  
xpack.encryptedSavedObjects.encryptionKey: 71d561c9882a7914d322cf9056fe7c1e  
xpack.reporting.encryptionKey: 6ce999468e75b93d22b7cfd5610e8b77  
xpack.security.encryptionKey: c9bec62bb0bf108c2d1d872f6930ee91  

xpack.encryptedSavedObjects.encryptionKey: e74d34e08e291d1757c226eba35c645b
xpack.reporting.encryptionKey: 416abe1c08383fbdd90c38dfcdbccc48
xpack.security.encryptionKey: 47f7b66a21ff9248d073bdb995ed4b1d

**Commands/Scripts used:**  

```
sudo apt update && sudo apt upgrade 
sudo bash -c "export HOSTNAME=kaliinternal.lp.local; apt-get install elasticsearch -y"

sudo sed -e '/cluster.initial_master_nodes/ s/^#*/#/' -i /etc/elasticsearch/elasticsearch.yml 
echo "discovery.type: single-node" | sudo tee -a /etc/elasticsearch/elasticsearch.yml  

sudo cat /etc/hosts  
sudo nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       kaliinternal.lp.local   kaliinternal
192.168.100.200 kaliinternal.lp.local   kaliinternal

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters  

dnsdomainname  
ping kaliinternal.lp.local  


sudo apt install kibana  

sudo /usr/share/kibana/bin/kibana-encryption-keys generate –q  

echo "server.host: \"kaliinternal.lp.local\"" | sudo tee -a /etc/kibana/kibana.yml 
sudo systemctl enable elasticsearch kibana --now  

systemctl status kibana 
systemctl status elasticsearch

sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

```
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTkyLjE2OC4xMDAuMjUwOjkyMDAiXSwiZmdyIjoiNDk5YTcyMGYzMDViMTdjZWJlYzhiN2JjM2I3OTU2NTE1MmEzYzBjMTg4YTkwOWQ5YjI4YmJkNjBlMGM1MTkxZiIsImtleSI6IkdJX3ZCWkFCWnltMEdRbV9ZM3ZJOmppRzZ2X0pDU2lpbDZ1eUhmekhiQWcifQ==  
**Issues**
- Cant connect to Elatic or Kibana even though the theyre running perfectly fine
  Possible work around - just use another PC thats internal (Sebs PC) or find a way to fix the issue for next time

                                                Day Five

Name: *Ryan Saunders*  
Date: *11/06/2024*


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: password1
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Tasks Assigned:**
- Kali Purple (Internal)   
Testing RBT activities    
Configuring Elastic security alerts and rules   
- Elastic Login  
  https://192.168.100.250:9200  
  Username: elastic  
  Password: FKEn7VZpCDgA=BT5T36X  
**Tasks Completed:**
- Enable HTTPS for Kibana  

**Commands/Scripts used:**  

```
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca 
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --dns siem.lp.local,elastic.lp.local,siem --out kibana-server.p12 
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
echo "server.publicBaseUrl: \"https://kaliinternal.lp.local:5601\"" | sudo tee -a /etc/kibana/kibana.yml 
 
sudo /usr/share/kibana/bin/kibana-encryption-keys generate  
## Copy the generated keys into /etc/kibana/kibana.yml  

```

**Issues**
- Previously had an issue accessing Elastic. Now its fixed.

                                                Day Six
                                            
Name: *Ryan Saunders*  
Date: *13/06/2024*  


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    -Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: Password1
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Tasks Assigned:**
- Kali Purple (Internal)
Elastic security alerts and rules setup

**Tasks Completed:**
- Elastic security alerts and rules
  
**Commands Used:**

```

Security > Alerts - Manage Rules
Add Elastic rules - Install All , Go back to Installed Elastic rules

Rules must be enabled , view Potential Successful SSH Brute Force Attack - enable

hydra attack against linux target via ssh brute force password guessing

```

**Issues**
None at this time.

                                               Day Seven
                                            
Name: *Ryan Saunders*  
Date: *18/06/2024*  


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: password1
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller


**Tasks Assigned:**
- Perform a pentest and monitor as Blue team

**Tasks Completed:**
- Nothing because the alerts are still not working. 

**Issues**
- Kali external wasnt actually on the outside network, causing the issue of true pentesting as a red team member

                                               Day Eight
                                            
Name: *Ryan Saunders*  
Date: *20/06/2024*  


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: password1
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 8GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller


**Tasks Assigned:**
- Perform a pentest and monitor as Blue team with Elastic Alerts
- Take a bunch of screenshots in preparation for the presentation

**Tasks Completed:**

**Commands Used**
```
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.14.0-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.14.0-linux-x86_64.tar.gz
cd elastic-agent-8.14.0-linux-x86_64
sudo ./elastic-agent install \
  --fleet-server-es=https://192.168.100.250:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3MTg4NTU4MTgxNjA6Smo2cUpFbUdSOG0xTGlPODVpSGF3Zw \
  --fleet-server-es-ca-trusted-fingerprint=499a720f305b17cebec8b7bc3b79565152a3c0c188a909d9b28bbd60e0c5191f \
  --fleet-server-port=8220
```
-  Screenshots taken
-  Made a branch called "Presentation"

**Issues**
- Still no alerts
- When I installed the fleet server, my VM was running at 100% CPU usage. I even upgraded the RAM and CPU to 8GB RAM and 2 Cores for the CPU

##MAJOR SCREWUP##
-Misunderstood instructions. Was meant to be red Teaming with Phoebe but performing as an internal attack vector.

use OWAP ZAP
ref(Cisco EH - 6.12.13)
kali menu -> zap

Automated scan
URL to Attack: 192.168.50.100
Attack

Alerts tab

**Commands Used**
```
nmap -sn
nmap -sT
nmap -O 192.158.50.100
nmap -p80 -sV -A 192.158.50.100

nmap -A -p139,445 192.168.100.100    # SMB ports
nmap --script smb-enum-users.nse -p139,445 192.168.100.100
nmap --script smb-enum-shares.nse -p445 192.168.100.100

ping -c5 192.168.50.100
nmap -sV 192.168.50.100
sudo nmap -O 192.168.50.100
nmap -sV --script vulners --script-args mincvss=4 192.168.50.100

nikto --help
nikto -h 192.168.50.100
nikto -h 192.168.50.100 -ssl
nikto -h 192.168.50.100 -o pentest.htm
nikto -h 192.168.50.100 -o scan_results.txt -Format csv

```
                                              Day Nine
                                            
Name: *Ryan Saunders*  
Date: *21/06/2024*  


**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal**
    - Username: kaliinternal
    - Password: password1
- OS: Linux, Ubuntu (64-bit)
- iso: kali-linux-2023.4-installer-purple-amd64.iso
- Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller


**Tasks Assigned:**
- Red Teaming using Venom

**Tasks Completed:**
- installed Venom and ran the following
    - Brute Force
    - 
**Commands Used**
```
sudo apt update && sudo apt upgrade -y
1.) attack 1 Venom
Install Venom from https://github.com/hackingmastert56/Venom-Tools-Installer/
cd Venom-Tools-Installer 
chmod +x setup.sh
chmod +x venom.sh
./venom.sh
```

**Issues**
- couldnt install Hydra. Broken packages
- something to do with libfreerdp2-2t64 : Depends: libavcodec60 (>= 7:6.0)

                                          Day Ten
                                            
Name: *Ryan Saunders*  
Date: *25/06/2024*  

To Start, I installed a whole new VM from home, inbetween class time to save time.  
The VM was just a spare Kali Linux, not using purple version.  
I did this for the sake of time. Installed a bunch of pentest tools, and exported the VM as an OVA.  

**VM's:**

**Windows 10 Client**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Windows 10 (64-Bit)
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Kali Internal (Kali Linux)**
    - PC-Name: DESKTOP-RYAN
    - Username: Ryan
    - Password: Password1
- OS: Kali Linux
- iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
- Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
- Network Adapter: Bridged Adapter
    - Name: Realtek PCIe GBE Family Controller

**Tasks Assigned:**
- Red Teaming 
    - Nmap
    - GVM
    - Nikto
    - OWAP Zap
    - Greenbone
**Tasks Completed:**
- 
**Commands Used**
  
```
Enumeration with NMAP Scans
nmap -sn 192.158.50.100
nmap -sT 192.158.50.100
nmap -O 192.158.50.100
nmap -p80 -sV -A 192.158.50.100

nmap -A -p139,445 192.168.100.100    # SMB ports
nmap --script smb-enum-users.nse -p139,445 192.168.100.100
nmap --script smb-enum-shares.nse -p445 192.168.100.100

#Vulnerability scans
ping -c5 192.168.50.100
nmap -sV 192.168.50.100
sudo nmap -O 192.168.50.100
nmap -sV --script vulners --script-args mincvss=4 192.168.50.100

#GVM
sudo gvm-check-setup
sudo gvm-stop

#using GVM vulnerability scanner
sudo gvm-start
#https://127.0.0.1:9392
#Username: admin
#Password: kali
#Scans -> Tasks
#IP address or hostname: 192.168.50.100
#Download the report by clicking the Download Filtered Report button
sudo gvm-stop

#Nikto WEB vulnerability scan
nikto --help
nikto -h 192.168.50.100
nikto -h 192.168.50.100 -ssl
nikto -h 192.168.50.100 -o pentest.htm
nikto -h 192.168.50.100 -o scan_results.txt -Format csv

#use OWAP ZAP
#kali menu -> zap
#Automated scan
#URL to Attack: 192.168.50.100
#Attack
#Alerts tab

```

**Issues**
- 
