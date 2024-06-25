## My documentation
This is documentation about the VM's that I have installed for the production network. It contains when and what stage I am up to with each machine and the steps i have completed. It also includes the credidentials required to log in into each VM.

## My Production network virtual machines

**Windows 10 Client:**
  - Username: Seb
  - Password: Password1

**KaliSiem:**
  - Username: kalisiem
  - Password: Password1

### Credentials
  **ElasticSearch**
  - Login: elastic
  - Password: GVnRTWTIuc4E=G6Cy*mt

  **Management User**
  - Username: admin
  - Password: Password1


### My notes and progress
These are notes and my progress with timestamps of what i've done for the production network
  - 25/06/24 Took more screenshots and performed more Red & Blue testing
    - A TCP flood attack which was logged into the alerts
    - A Nmap scan that logged into the alerts
    - A attempted Dos attack which was logged into the alerts
  - 21/06/24 Took more screenshots and performed Red and Blue testing the network
  - 20/06/24 Prepared and took screenshosts from completed tasks needed for the presentation
    - Peformed a successful ssh brute force attack with logged the attack with a alert
  - 18/06/24 Completed tasks, 16.1, 16.2, 16.3, 16.4
  - 13/06/24 Added rules for alert to brute force ssh (Still doesn't work WIP) finished the KaliSiem at this point, need to update all intergrations and agents afterwards
  - 11/06/24 Installed CISO IOS intergration onto the elastic, and have configured the switch to send logs between the two
  - 04/06/24 Connected Windows10 Clients to Elastic and DMZ web server, up to step 8.5
  - 28/05/24 Installed ElasticSearch & Kibana onto the KaliSIEM
  - 23/05/24	KaliSIEM has been freshly installed and yet to be configured
  - 21/05/24	Windows 10 Client has been installed and yet to be configued


## Tasks that have been completed
#### 2. Set up configure and interconnect Virtual environment, firewall , servers and client
#### 2.1 Install VirtualBox
#### 2.2 Virtual box networking
#### 2.4 Install kali Purple
#### 2.4.2 PRODUCTION NETWORK - connect to Palo Alto Firewall
#### 2.5 Install windows 10/11 Clients
#### 3.0 Configure kali Purple - for remote management - ssh, rdp, console
#### Kali Purple: Enable serial console 
#### 4.0 Implement kali Purple SIEM - install elastic stack - elastic , kibana
#### 4.0 What is Elastic and Kibana (ELK), and how can it be used as a SIEM
#### 4.1 Install Elasticsearch:  
#### 4.2 Convert to single-node setup (or replace fqdn name in initial_master_nodes list with IP address):  
#### 4.3 Install kibana
#### 4.4 Enrol Kibana
#### 4.5 Enable HTTPS for Kibana: 
#### 5.0 Implement Fleet Server and end point protection agent - overview / installation
#### 5.1 Fleet Server and elastic agent overview 
#### 5.2 Fleet Server Installation 
#### 5.3 Elastic Defend integration and install windows 10 endpoint protection agent
#### 5.4 Configure an integration policy for Elastic Defend
#### 5.5 Elastic secuirty UI  
#### 5.6 Set up security rules and alerts
#### 7.0 Integrate Palo Alto Firewall into Kali Purple SIEM  
#### 7.2 Palo Alto Firewall Elastic Integration
#### 7.2.2 Confirm firewall is sending logs using wireshark
#### 7.3 Display firewall logs in Kibana 
#### 8.5 Install Elastic Defend agent in DMZ WEB Server  
#### Fleet Agent Policy - Linux server policy
#### Install the agent in the linux server via Linux Tar script 
#### confirm Linux DMZ server logs are sucessfully sent to kibana 
#### 9.3 Configure and test elastic user accounts and role(s)
#### 9.11 Cisco switch logging to Fleet
#### 9.11.2  Cisco IOS elastic integration
#### 11.1 Elastic security alerts and rules
#### 13.0 Kibana discover
#### 13.1 Kibana discover - Endpoints Network events  (sample filters)
#### 13.4 Kibana Alerts
#### 16.0 Managing SIEM Updates
#### 16.1 Update kaliPurple and kibana
#### 16.2 Update Elastic integrations
#### 16.3 Update fleet and agents
#### 16.4 checking client agents
## Tasks that yet to be completed
