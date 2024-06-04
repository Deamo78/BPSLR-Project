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
Date: 04*/06/2024*  


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
