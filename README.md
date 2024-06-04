Welcome to our project!

This README file serves as documentation of our work towards the project and how we will use github to host our files.


-----=BRANCHES=-----

main - The main branch which we will use for pushing our individual work towards.

seb - Work made by Sebastian will be put in here before pushed to main.

brody - Work made by Brody will be put in here before pushed to main.

phoebe - Work made by Phoebe will be put in here before pushed to main.

ryan - Work made by Ryan will be put in here before pushed to main.

luke - Work made by Luke will be put in here before pushed to main.

-----=============================================================-----

-----=INSTALLATION=-----
To install and enroll your Windows 10 Client to the Kali Siem copy this code below into powershell as admin and execute it by following the steps:

$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.4-windows-x86_64.zip -OutFile elastic-agent-8.13.4-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.13.4-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.13.4-windows-x86_64
.\elastic-agent.exe install --url=https://192.168.100.200:8220 --enrollment-token=TWtxMXg0OEJLMDFVWTVTamhQaXU6MDhfVi0wWXRRYXU0azdVVVRBbjhNUQ== --insecure

-------------------------
