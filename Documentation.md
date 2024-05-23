DOCUMENTATION

#####################################################################################################

                                                Day One
                                            
Name: Ryan Saunders
Date: 23/05/2024


VM's

Windows 10 Client 
    -PC-Name: DESKTOP-RYAN
    -Username: Ryan
    -Password: Password1
-OS: Windows 10 (64-Bit)
-iso: en_windows_10_eval_22h2_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso
-Specs: 8GB RAM, 2 CPU Threads, 50GB Storage
-Network Adapter: Bridged Adapter
    -Name: Realtek PCIe GBE Family Controller

Kali Internal
    -Username: kaliinternal
    -Password: 799EQmwF
-OS: Linux, Ubuntu (64-bit)
-iso: kali-linux-2023.4-installer-purple-amd64.iso
-Specs: 4GB RAM, 1 CPU Threads, 50GB Storage
-Network Adapter: Bridged Adapter
    -Name: Realtek PCIe GBE Family Controller


Tasks Assigned:

- Install Windows 10 Client
Make sure its connected to the Windows22 Server

- Install Kali Purple (Internal)
Make sure its connected Internally

- Connect Host PC to PaloAlto
Make sure its connected to the internet


Tasks Completed:
- Installed Windows 10 Client 
![day1win10c]
(D:/Ryan CSOC/Screenshots/Day1/day1win10c.png)  
It isnt connected to the internet or to the server yet. The server isnt configured completely 

- Installed Kali Linux Purple
![day1kaliInt]
(https://github.com/Deamo78/BPSLR-Project/blob/ryan/day1kaliInt.png)
Used Step 2.4 as referenced from Steves GitHub.
It has internet connection and is able to ping the server, as well as the switch (PaloAlto Firewall).


Commands Used:

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


#####################################################################################################
                                                















