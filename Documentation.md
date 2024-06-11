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
   - https://192.168.100.1

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

After Windows Server DHCP has been set up, IP configuration will changed from static to DHCP  

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

Switch has been configured for secure remote access using local database:

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

Unused switch ports have been shutdown:

```
interface range G1/0/7 - 52
shutdown
```

Note: port-security

*Next Session: finish 9.4 Configure and test management VLAN / host isolation*, *9.11 Cisco switch logging to Fleet*  

---

*6/06/2024*

- Issues: No connection with other internal devices after assigning G1/0/4 to Management VLAN 100, either the devices will all need to be on the same VLAN or inter-VLAN routing needs to be configured on the Layer 3 Switch
- Also, cannot SSH to switch now that my device is no longer on the same VLAN (must use console connection once again)  

VLAN 100 creation, port assignment, and port-security

```
vlan 100
name Management
exit

interface range G1/0/1 - 6
switchport mode access
switchport access vlan 100

switchport port-security
switchport port-security maximum 4
switchport port-security mac-address sticky

show port security
show port security G1/0/4
show interface G1/0/4

show run | begin interface G1/0/1
show port-security address
```

- Issues: after using the switchport port-security command, all interfaces went down in error-disabled state as the default max number of MAC addresses (1) had been exceeded due to bridged networking (some ports had multiple VMs running all with separate MAC addresses that triggered the shutdown violation)
- To counter this, next session a list of trusted MAC addresses will be added to the switch config instead of setting a max. number of addresses per port  

To bring the interfaces back up after violation shutdown:

```
interface range G1/0/1 - 6
shutdown
no shutdown
```

- Port-security will cause a port to transition to a shutdown state if the limit of 4 MAC addresses is exceeded
- Use the show running-config to check that MAC addresses are sticking
- Check with Steve about vty line numbers (0-4 vs 5-15)  

ACL to restrict VTY lines to management IP addresses on internal network:

```
ip access-list standard ADMIN-MGMT
permit 192.168.100.0 0.0.0.255

line vty 0 4
access-class ADMIN-MGMT in

show accesss-lists
```

Cisco switch logging, port-security logging (auto if violation mode is shutdown by default):

```
configure terminal
service timestamps log datetime
service sequence-numbers
logging 192.168.100.200
logging trap 4
logging facility auth
```

- Need seb to install Cisco IOS elastic integration on SIEM
- NTP if time permits  

Elastic agent has been installed on Windows 10 Client and successfully integrated with the Fleet server on SIEM

---

*11/06/2024*

Changed switch port-security method from using dynamic sticky MAC addresses to static MAC entries (as per Steve's recommendations)  

Had issue "Found duplicate mac-address" when trying to manually configure MAC address for a device already connected to the port. This required that the affected device be disconnected from the network, their interface settings restored to default, and static configuration applied before they reconnected to the network.

```
default interface g1/0/2
default interface g1/0/3
default interface g1/0/4
default interface g1/0/5
default interface g1/0/6
```

Each port must be configured manually with all MAC addresses (including the host PC, and all VMs running on that PC)

Final Port-security configuration commands (for Brody - G1/0/4): 

```
switchport mode access
switchport access vlan 100
switchport port-security
switchport port-security maximum 5
switchport port-security mac-address 0800.2750.bef5 vlan access
switchport port-security mac-address 74d4.355f.70c7 vlan access
```
