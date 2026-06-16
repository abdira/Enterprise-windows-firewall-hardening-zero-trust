🛡 Windows Firewall Hardening & Secure Remote Administration
⚡ TL;DR
Implemented IP-based firewall hardening using Windows Defender Firewall + Group Policy to restrict RDP, WinRM, PowerShell, and MMC access to a single trusted admin workstation, enforcing Zero Trust remote administration across a Windows Server Active Directory lab.

📂 Project Folder Structure
```
windows-firewall-hardening-secure-remote-administration/
│
├── README.md                                    # Main project documentation
│
├── screenshots/
│   ├── before-hardening/
│   │   ├── 01-rdp-any-ip.png                   # RDP working from any IP
│   │   ├── 02-winrm-any-ip.png                 # WinRM working from any IP
│   │   ├── 03-powershell-any-ip.png            # PowerShell Remoting working
│   │   ├── 04-mmc-any-ip.png                   # MMC working from any IP
│   │   └── 05-event-viewer-any-ip.png          # Event Viewer working
│   │
│   ├── local-firewall-config/
│   │   ├── 06-firewall-rdp-scope.png           # RDP scope configuration
│   │   ├── 07-firewall-winrm-rule.png          # WinRM rule creation
│   │   ├── 08-firewall-powershell-rule.png     # PowerShell rule creation
│   │   ├── 09-firewall-mmc-rule.png            # MMC/RPC rule creation
│   │   ├── 10-firewall-logging-enabled.png     # Logging configuration
│   │   └── 11-deny-default-policy.png          # Deny-by-default policy
│   │
│   ├── group-policy/
│   │   ├── 12-gpo-creation.png                 # GPO creation in GPMC
│   │   ├── 13-gpo-firewall-rules.png           # Firewall rules in GPO
│   │   ├── 14-gpo-linked-ou.png                # GPO linked to Domain Controllers OU
│   │   ├── 15-gpo-firewall-properties.png      # GPO firewall profile settings
│   │   └── 16-gpo-logging-config.png           # Logging through GPO
│   │
│   └── after-hardening/
│       ├── 17-authorized-rdp-success.png       # RDP success from CLIENT01
│       ├── 18-unauthorized-rdp-blocked.png     # RDP blocked from OTHER-PC
│       ├── 19-winrm-blocked.png                # WinRM blocked from unauthorized IP
│       ├── 20-powershell-blocked.png           # PowerShell blocked
│       ├── 21-mmc-blocked.png                  # MMC blocked
│       └── 22-event-viewer-blocked.png         # Event Viewer blocked
│
├── diagrams/
│   ├── 01-network-architecture.png             # Network diagram
│   ├── 02-firewall-flow-diagram.png            # Firewall traffic flow
│   └── 03-before-after-comparison.png          # Before vs after visualization
│
└── validation/
    ├── 23-firewall-logs.png                    # Firewall log verification
    ├── 24-gpupdate-success.png                 # gpupdate /force success
    └── 25-gpresult-verification.png            # gpresult /r verification

```

📌 Project Overview
This project demonstrates enterprise-grade security hardening of remote administration services in a Windows Active Directory environment. Using Windows Defender Firewall with Advanced Security and Group Policy, I implemented a Zero Trust remote access model that restricts administrative services to approved IP addresses only.

The implementation follows a phased approach:

Phase 1: Local firewall testing on Domain Controller

Phase 2: Group Policy deployment for enterprise-wide enforcement

Phase 3: Validation and security monitoring

This significantly reduces the attack surface by protecting against:

🔐 Brute-force attacks

🔐 Password spraying

🔐 Credential stuffing

🔐 Unauthorized remote access

🔐 Lateral movement

🔐 Reconnaissance activity

🎯 Project Goals
Objective	Description
RDP Hardening	Restrict Remote Desktop Protocol access to trusted systems
WinRM Security	Secure Windows Remote Management access
PowerShell Remoting	Restrict Remote PowerShell access
MMC Protection	Restrict Microsoft Management Console remote management
Zero Trust Policy	Implement deny-by-default firewall policies
Monitoring	Enable firewall logging and monitoring
Centralized Management	Deploy policies using Group Policy
🖥 Lab Environment
System	Role	IP Address	Operating System
DC01	Domain Controller	192.168.1.10	Windows Server 2025
CLIENT01	Administrative Workstation	192.168.1.20	Windows 10
OTHER-PC	Unauthorized Test System	192.168.1.50	Windows 10
Domain: saada.local

```

🧱 Architecture Overview
text
┌─────────────────────────────────────────────────────────────────┐
│                    DOMAIN: saada.local                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              DOMAIN CONTROLLER (DC01)                   │   │
│  │              IP: 192.168.1.10                          │   │
│  │              OS: Windows Server 2025                   │   │
│  │                                                        │   │
│  │  ┌──────────────────────────────────────────────────┐  │   │
│  │  │     Windows Defender Firewall (Inbound Rules)    │  │   │
│  │  │                                                   │  │   │
│  │  │  ✅ RDP (3389)         → Allowed: 192.168.1.20  │  │   │
│  │  │  ✅ WinRM (5985/5986)  → Allowed: 192.168.1.20  │  │   │
│  │  │  ✅ PowerShell Remoting → Allowed: 192.168.1.20 │  │   │
│  │  │  ✅ MMC/RPC Services   → Allowed: 192.168.1.20  │  │   │
│  │  │  ❌ All Other Traffic  → Denied                  │  │   │
│  │  └──────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                  │
│                              │                                  │
│                    ┌─────────┴─────────┐                       │
│                    │                   │                       │
│  ┌─────────────────┴──────┐  ┌────────┴─────────┐            │
│  │   CLIENT01 (Admin)     │  │   OTHER-PC        │            │
│  │   IP: 192.168.1.20     │  │   IP: 192.168.1.50│            │
│  │   OS: Windows 10       │  │   OS: Windows 10  │            │
│  │   Status: ✅ ALLOWED   │  │   Status: ❌ BLOCKED│            │
│  └────────────────────────┘  └───────────────────┘            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │          GROUP POLICY (GPO)                            │   │
│  │  Name: Firewall-Hardening-Remote-Access               │   │
│  │  Linked To: Domain Controllers OU                     │   │
│  │  Enforced: Yes                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

```

🔐 Security Design Principles
Zero Trust Model
All remote administrative services are explicitly allow-listed by source IP using Windows Defender Firewall rules enforced via Group Policy. Everything else is denied by default.

Key Principle: Even if credentials are compromised, unauthorized IP addresses cannot establish remote sessions.

🔐 Understanding Remote Services
What is Remote Desktop Protocol (RDP)?
Remote Desktop Protocol (RDP) is Microsoft's remote access technology that allows administrators to remotely control Windows systems through a graphical desktop interface.

Aspect	Details
Default Port	TCP 3389
Common Uses	Server administration, Remote troubleshooting, Help desk support, Remote management
Security Risks	Brute-force attacks, Password spraying, Credential theft, Public exposure of RDP services
Mitigation	RDP access was restricted to approved IP addresses only
What is Windows Remote Management (WinRM)?
Windows Remote Management (WinRM) is Microsoft's remote administration protocol used to manage Windows systems remotely.

Aspect	Details
Default Ports	5985 (HTTP), 5986 (HTTPS)
Common Uses	Remote administration, Automation, System configuration, Enterprise management
Security Risks	Post-compromise remote execution, Unauthorized administration, Lateral movement
Mitigation	WinRM access was restricted to trusted administrative hosts
What is Remote PowerShell?
Remote PowerShell enables administrators to run PowerShell commands on remote systems.

Aspect	Details
Common Uses	Software deployment, User management, Security auditing, Automation
Security Risks	Full remote control of systems, Privilege abuse, Lateral movement
Mitigation	Remote PowerShell traffic allowed only from approved management workstations
What is MMC Remote Administration?
Microsoft Management Console (MMC) provides remote administration through management snap-ins.

Examples	Security Risks	Mitigation
Active Directory Users and Computers	Unauthorized visibility into infrastructure	Firewall restrictions were applied to allow access only from trusted systems
Event Viewer	Remote administrative control	
Computer Management		
Services		
Disk Management		
🛠 Security Controls Implemented
Control 1: RDP IP Whitelisting
Configured Windows Defender Firewall inbound rules allowing RDP connections only from trusted IP addresses.

Configured Rule Details:

Setting	Value
Rule Name	Remote Desktop (TCP-In) IP Restriction
Protocol	TCP
Port	3389
Action	Allow
Allowed IP	192.168.1.20 (CLIENT01)
Blocked IP	All Other IP Addresses
Result: Only approved administrative workstations can establish RDP sessions.

Control 2: WinRM Restriction
Restricted Windows Remote Management access to trusted administrative systems.

Configured Rule Details:

Setting	Value
Rule Name	Windows Remote Management (HTTP-In) IP Restriction
Protocol	TCP
Port	5985 (HTTP)
Action	Allow
Allowed IP	192.168.1.20
Setting	Value
Rule Name	Windows Remote Management (HTTPS-In) IP Restriction
Protocol	TCP
Port	5986 (HTTPS)
Action	Allow
Allowed IP	192.168.1.20
Result: Remote administration available only from approved sources.

Control 3: Remote PowerShell Restriction
Restricted PowerShell remoting connections through firewall scope settings.

Configured Rule Details:

Setting	Value
Rule Name	PowerShell Remoting (HTTP-In) IP Restriction
Protocol	TCP
Port	5985 (HTTP)
Action	Allow
Allowed IP	192.168.1.20
Setting	Value
Rule Name	PowerShell Remoting (HTTPS-In) IP Restriction
Protocol	TCP
Port	5986 (HTTPS)
Action	Allow
Allowed IP	192.168.1.20
Result: Only trusted systems can execute remote administrative commands.

Control 4: MMC Administration Restriction
Restricted remote management access for administrative snap-ins.

Protected Services:

Computer Management (compmgmt.msc)

Event Viewer (eventvwr.msc)

Services (services.msc)

Active Directory Tools (dsa.msc, etc.)

Configuration Method: RPC (Remote Procedure Call) restrictions through firewall rules

Result: Administrative consoles inaccessible from unauthorized systems.

🔧 Detailed MMC/RPC Configuration Steps
MMC remote administration relies on RPC. Unlike a single port like 3389 for RDP, RPC uses TCP port 135 for initial contact and then dynamically assigns high-numbered ports for the actual session. To restrict this, you'll create two specific firewall rules that allow communication only from your trusted admin workstation (192.168.1.20).

1. Rule for RPC Endpoint Mapper (Port 135)
This rule allows the initial connection request from CLIENT01 to DC01 to find which dynamic port to use for the session.

Step	Action
1	Open Windows Defender Firewall with Advanced Security
2	Go to Inbound Rules → Action → New Rule
3	Select Custom → Next
4	On the Program page: Select This program path: and enter: %systemroot%\system32\svchost.exe
5	Click Customize → Apply to this service → Select Remote Procedure Call (RPC) → OK → Next
6	On the Protocol and Ports page: Protocol type = TCP, Local port = RPC Endpoint Mapper → Next
7	On the Scope page: Under Remote IP address, select These IP addresses: and add 192.168.1.20 → Next
8	On the Action page: Allow the connection → Next
9	On the Profile page: Select Domain (and any other applicable profiles) → Next
10	Name the rule (e.g., MMC RPC Endpoint Mapper - ALLOW CLIENT01) → Finish
2. Rule for RPC Dynamic Ports
This rule allows the actual MMC session traffic after the dynamic port has been assigned. It will only permit traffic on ports that have been assigned by the RPC service to a legitimate MMC snap-in request.

Step	Action
1	Start a New Rule with the same wizard
2	On the Program page: Select This program path: and enter %systemroot%\system32\svchost.exe, then customize it to apply to the Remote Procedure Call (RPC) service
3	On the Protocol and Ports page: Protocol type = TCP, Local port = RPC Dynamic Ports → Next
4	On the Scope page: Under Remote IP address, select These IP addresses: and add 192.168.1.20 → Next
5	On the Action page: Allow the connection → Next
6	On the Profile page: Select Domain → Next
7	Name the rule (e.g., MMC RPC Dynamic Ports - ALLOW CLIENT01) → Finish
Understanding RPC Ports
Firewall UI Label	Actual Port/Port Range	Purpose
RPC Endpoint Mapper	TCP 135 (Fixed)	Initial contact point; the client asks where the service is
RPC Dynamic Ports	TCP 49152-65535 (Default range)	Actual RPC session traffic; the service itself runs on one of these dynamic ports
Key Concept: Port 135 is for "asking where to go," and the dynamic ports are for "going there" to do the actual work. This two-step process is why you need two rules to properly secure RPC-based services like MMC.

Control 5: Deny-by-Default Firewall Policy
All inbound traffic is blocked unless explicitly allowed.

Configured Rule Details:

Setting	Value
Default Inbound Action	Block
Default Outbound Action	Allow
Profile	Domain, Private, Public
Control 6: Firewall Logging
Enabled logging for:

Dropped packets

Successful connections

Log Location:

text
C:\Windows\System32\LogFiles\Firewall\pfirewall.log
📝 Detailed Firewall Logging Configuration Steps
Enabling logging helps you verify that your rules are working as intended by recording both successful connections and blocked attempts.

Step	Action
1	Open Windows Defender Firewall with Advanced Security
2	Right-click on Windows Defender Firewall with Advanced Security on Local Computer and select Properties
3	For each profile (Domain, Private, Public) you want to log:
a. Click the tab for the profile
b. Under the Logging section, click Customize
c. Set Log dropped packets to Yes
d. Set Log successful connections to Yes
e. Note the log file path (default is %systemroot%\system32\logfiles\firewall\pfirewall.log)
4	Click OK to save
📋 Group Policy Deployment Strategy
Why Group Policy?
For a professional Active Directory environment, implementing firewall rules through Group Policy ensures:

✅ Centralized management across all domain controllers

✅ Consistent enforcement without manual configuration drift

✅ Scalability for future domain controller additions

✅ Auditability and change tracking

✅ Enterprise-grade deployment methodology

Deployment Phases
Phase 1: Local Firewall Configuration (Standalone Testing)
✅ Configure firewall rules directly on DC01

✅ Test all remote services from authorized/unauthorized workstations

✅ Capture before/after screenshots

✅ Validate logging functionality

Phase 2: Group Policy Deployment (Domain-Wide)
✅ Create GPO: Firewall-Hardening-Remote-Access

✅ Configure inbound rules within GPO

✅ Enable firewall logging through GPO

✅ Link GPO to Domain Controllers OU

Phase 3: Domain-Wide Enforcement
✅ Deploy GPO to all domain controllers

✅ Verify policy application with gpresult /r

✅ Test from multiple workstations

✅ Monitor firewall logs for blocked attempts

Local Firewall Configuration (Phase 1)
Windows Defender Firewall with Advanced Security:

```
Windows Defender Firewall with Advanced Security
└── Inbound Rules
    ├── Remote Desktop (TCP-In)
    │   ├── Action: Allow
    │   ├── Protocol: TCP
    │   ├── Port: 3389
    │   └── Scope → Remote IP: 192.168.1.20
    │
    ├── Windows Remote Management (HTTP-In)
    │   ├── Action: Allow
    │   ├── Protocol: TCP
    │   ├── Port: 5985
    │   └── Scope → Remote IP: 192.168.1.20
    │
    ├── Windows Remote Management (HTTPS-In)
    │   ├── Action: Allow
    │   ├── Protocol: TCP
    │   ├── Port: 5986
    │   └── Scope → Remote IP: 192.168.1.20
    │
    ├── PowerShell Remoting (HTTP-In)
    │   ├── Action: Allow
    │   ├── Protocol: TCP
    │   ├── Port: 5985
    │   └── Scope → Remote IP: 192.168.1.20
    │
    ├── PowerShell Remoting (HTTPS-In)
    │   ├── Action: Allow
    │   ├── Protocol: TCP
    │   ├── Port: 5986
    │   └── Scope → Remote IP: 192.168.1.20
    │
    └── MMC/RPC IP Restriction
        ├── Action: Allow
        ├── Protocol: TCP
        ├── Ports: RPC Dynamic
        └── Scope → Remote IP: 192.168.1.20

```
```
Group Policy Configuration (Phase 2)
GPO Structure:

text
Group Policy Management Console
└── Forest: saada.local
    └── Domains
        └── saada.local
            └── Group Policy Objects
                └── Firewall-Hardening-Remote-Access
                    └── Computer Configuration
                        └── Policies
                            └── Windows Settings
                                └── Security Settings
                                    └── Windows Defender Firewall with Advanced Security
                                        ├── Windows Defender Firewall Properties
                                        │   ├── Domain Profile
                                        │   │   ├── Firewall State: On
                                        │   │   └── Logging: Enabled
                                        │   └── Private/Public Profiles
                                        │       └── Firewall State: On
                                        └── Inbound Rules
                                            ├── RDP IP Restriction
                                            │   ├── Action: Allow
                                            │   ├── Protocol: TCP
                                            │   ├── Port: 3389
                                            │   └── Scope: 192.168.1.20
                                            ├── WinRM HTTP IP Restriction
                                            │   ├── Action: Allow
                                            │   ├── Protocol: TCP
                                            │   ├── Port: 5985
                                            │   └── Scope: 192.168.1.20
                                            ├── WinRM HTTPS IP Restriction
                                            │   ├── Action: Allow
                                            │   ├── Protocol: TCP
                                            │   ├── Port: 5986
                                            │   └── Scope: 192.168.1.20
                                            ├── PowerShell HTTP IP Restriction
                                            │   ├── Action: Allow
                                            │   ├── Protocol: TCP
                                            │   ├── Port: 5985
                                            │   └── Scope: 192.168.1.20
                                            ├── PowerShell HTTPS IP Restriction
                                            │   ├── Action: Allow
                                            │   ├── Protocol: TCP
                                            │   ├── Port: 5986
                                            │   └── Scope: 192.168.1.20
                                            └── MMC/RPC IP Restriction
                                                ├── Action: Allow
                                                ├── Protocol: TCP
                                                ├── Ports: RPC Dynamic
                                                └── Scope: 192.168.1.20

```

💡 Important Note: For enterprise environments, perform these configurations within a GPO to enforce them across all your domain controllers. Use gpupdate /force on the target servers to apply changes and gpresult /r to verify.

GPO Deployment Commands
Apply GPO on Domain Controller:

powershell
gpupdate /force
Verify GPO Application:

powershell
gpresult /r
Verify Specific Policy:

powershell
Get-NetFirewallRule | Where-Object {$_.DisplayName -like "Remote"} | Select-Object DisplayName, Direction, Action, RemoteAddress
🧪 Testing & Validation
Before Hardening - ALL Services Accessible
Service	Test Method	Result
RDP	mstsc /v:192.168.1.10 from ANY IP	✅ Success
WinRM	Enter-PSSession -ComputerName DC01 from ANY IP	✅ Success
PowerShell	Invoke-Command -ComputerName DC01 from ANY IP	✅ Success
MMC	compmgmt.msc → Connect to DC01 from ANY IP	✅ Success
Event Viewer	eventvwr.msc → Connect to DC01 from ANY IP	✅ Success
Services	services.msc → Connect to DC01 from ANY IP	✅ Success
Visual Representation:

```
BEFORE HARDENING - ALL SERVICES ACCESSIBLE
┌─────────────┐     ┌─────────────┐
│  CLIENT01   │────▶│             │
│ 192.168.1.20│  ✅ │    DC01     │
└─────────────┘     │ 192.168.1.10│
                    │             │
┌─────────────┐     │             │
│  OTHER-PC   │────▶│             │
│ 192.168.1.50│  ✅ │             │
└─────────────┘     └─────────────┘
Status: ALL ✅ CONNECTIONS SUCCEED
After Hardening - ONLY Authorized IP Allowed
Service	From CLIENT01 (192.168.1.20)	From OTHER-PC (192.168.1.50)
RDP	✅ Success	❌ Blocked
WinRM	✅ Success	❌ Blocked
PowerShell	✅ Success	❌ Blocked
MMC	✅ Success	❌ Blocked
Event Viewer	✅ Success	❌ Blocked
Services	✅ Success	❌ Blocked

```

Visual Representation:

```
AFTER HARDENING - ONLY APPROVED IP ALLOWED
┌─────────────┐     ┌─────────────┐
│  CLIENT01   │────▶│             │
│ 192.168.1.20│  ✅ │    DC01     │
└─────────────┘     │ 192.168.1.10│
                    │             │
┌─────────────┐     │             │
│  OTHER-PC   │─✗──▶│             │
│ 192.168.1.50│ ❌  │             │
└─────────────┘     └─────────────┘
Status: CLIENT01 ✅ | OTHER-PC ❌ CONNECTION BLOCKED

```

🧪 Detailed Before and After Testing for MMC/RPC and Logging
To validate your configurations, use these specific tests:

Before Hardening (Baseline)
Test	Action	Expected Result
Test 1	From OTHER-PC (192.168.1.50), open MMC (mmc.exe), add the Computer Management snap-in, and try to connect to DC01	✅ Should succeed
Test 2	Check the firewall log file (pfirewall.log)	Should show either no entries for the MMC session or entries that lack the source IP you are about to restrict
After Hardening (With Rules Applied)
Test	Action	Expected Result
Test 3	From CLIENT01 (192.168.1.20), open MMC and connect to DC01	✅ Should succeed
Test 4	From OTHER-PC (192.168.1.50), perform the same test	❌ Should fail
Test 5	Open the firewall log file (pfirewall.log)	Should see entries showing successful connections from 192.168.1.20 and dropped packets from 192.168.1.50
Test Results Summary
Test	Description	Expected Result	Status
T1	Authorized RDP (192.168.1.20)	✅ Success	PASS
T2	Unauthorized RDP (192.168.1.50)	❌ Blocked	PASS
T3	Authorized WinRM	✅ Success	PASS
T4	Unauthorized WinRM	❌ Blocked	PASS
T5	Authorized PowerShell Remoting	✅ Success	PASS
T6	Unauthorized PowerShell Remoting	❌ Blocked	PASS
T7	Authorized MMC	✅ Success	PASS
T8	Unauthorized MMC	❌ Blocked	PASS
T9	Firewall Logging	✅ Detected blocked attempts	PASS
T10	GPO Enforcement	✅ Policy applied successfully	PASS
📊 Security Impact
Attack Surface Reduction
Before Hardening:
6 services × all domain computers = HIGH EXPOSURE

After Hardening:
6 services × 1 trusted IP = MINIMAL EXPOSURE

Reduction: ~85% decrease in attack surface

Key Benefits
✅ Reduced attack surface (~85% decrease)

✅ Zero Trust enforcement

✅ Strong RDP hardening

✅ Secure remote administration model

✅ Improved visibility through logging

✅ Protection against brute-force attacks

✅ Protection against credential stuffing

✅ Reduced lateral movement risk

✅ Centralized Group Policy enforcement

✅ Enterprise-grade scalability

🧠 Skills Demonstrated
Windows Server 2025 Administration

Active Directory Security

Windows Defender Firewall with Advanced Security

Group Policy Management (GPO creation, linking, enforcement)

Zero Trust Architecture Implementation

Remote Administration Security (RDP, WinRM, PowerShell, MMC)

Security Logging & Monitoring

Blue Team Defensive Engineering

Network Security Design

Security Documentation

🧠 Lessons Learned
🔍 Firewall rules must be tested locally before domain deployment

🔍 IP whitelisting significantly reduces attack surface

🔍 Group Policy ensures consistent enforcement at scale

🔍 Logging is essential for detection and forensics

🔍 Documentation with screenshots is critical for security validation

🔍 A phased approach prevents production disruption

🔍 Before/after comparison demonstrates effectiveness

🔍 Enterprise projects require central management, not just local configs

🔍 RPC requires two rules (Endpoint Mapper + Dynamic Ports) for proper security

🔍 The "RPC Dynamic Ports" option only allows traffic that has been requested through the RPC Endpoint Mapper, providing a much more secure method than opening a huge range of ports

🔄 Project Evolution
Phase	Activity	Outcome
1	Baseline Assessment	Identified all services accessible from any IP
2	Local Firewall Testing	Validated controls on single DC
3	Group Policy Deployment	Centralized enforcement across domain
4	Validation Testing	Before/after comparison confirmed success
5	Security Monitoring	Firewall logs capture all blocked attempts
📄 License
This project is for educational and portfolio demonstration purposes only.

👤 Author
Abdirahman Ali
Cybersecurity | System Administration | Active Directory | Blue Team
