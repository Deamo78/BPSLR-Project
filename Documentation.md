*23/05/2024*

**Tasks completed:**

**1.1 Palo Alto Firewall - basic setup**

- Connected directly to management interface on firewall with ethernet cable
- Accessed firewall management through web browser
  https://192.168.1.1
  Username: admin
  Password: Password1

**1.2 Palo Alto Firewall - basic feature configuration**

- Network interfaces
- Network zones
- Virtual Router
- Device Timezone
- Service Route
- DNS
- Source NAT
- Security Policies
- Network Management Profile (allows firewall management from internal network)

**1.3 Switch SW1 - basic setup**

PuTTY was used on host computer for switch configuration

Initial configuration using console (rollover?) cable

- Connection type: Serial

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

Subsequent configuration using telnet

- Connection type: Telnet
- IP Address: 192.168.100.254
- Password: cisco

**1.4 Cable up Physical network according to network Diagram**

- Firewall E1/1 connected to 108 network via Switch-A at top of network rack
- Firewall E1/2 connected to Switch SW1 (internal network)

Note: DMZ will be connected to firewall E1/3 after Luke has configured DMZ VM

**2.5 Install windows 10/11 Clients**

Device Name: DESKTOP-BRODY
OS: Windows 10 Enterprise Evaluation
Memory: 4.00 GB
Network Adapter 1: Bridged Adapter
VM Host Static IP Address: 192.168.100.4

*Next Session: 7.0 PaloAlto Firewall Settings*

---

*28/05/2024*

**Tasks completed:**

**7.0 PaloAlto Firewall Settings**

- Dynamic Updates
  
  - Antivirus (27/5/2024)
  - Applications and Threats (22/5/2024)

- Removed old AV and Applications and Threats images

**7.1 Palo Alto Firewall Syslog configuration**

- Palo Alto Firewall - Service Route
- Palo Alto Firewall - Syslog server profile
- Palo Alto Firewall - Log Forwarding profile
- Palo Alto Firewall - Security policy configuration to forward logs to kali Purple SIEM (Inside_To_Outside)

Palo Alto Firewall is now ready to be integrated with Kali Purple SIEM after Seb has configured SIEM

---

*30/05/2024*

**Tasks completed:**

**9.10.5 PA Firewall - Src nat and sec pol for DMZ server to outside**

- Configured Source NAT for DMZ with luke (troubleshooting no internet access from DMZ)
- After adding NAT policy, the DMZ web server can now be accessed from an outside 108 network PC with the following URL: <http://192.168.108.132>

**9.4 Configure and test management VLAN / host isolation**

- Switch has been configured for secure remote access using local database

```
service password-encryption  
no ip domain-lookup
ip domain-name LP.local 
username brody secret Password1   
crypto key generate rsa general-keys modulus 1024
login block-for 100 attempts 3 within 100

line vty 0 4
transport input ssh  
login local
exec-timeout 10

show ip ssh
```

- Management VLAN has been changed from default VLAN1 to VLAN100 (this required console connection again after removing the IP address from VLAN1)
- Unused switch ports have been shutdown

Note: port-security

*Next Session:  9.4 Configure and test management VLAN / host isolation* - finish this
                        *9.11 Cisco switch logging to Fleet*

---
