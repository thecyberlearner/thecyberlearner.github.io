+++
title = 'Phantom Check'
date = 2025-07-23T02:48:10-04:00
+++


# Phantom Check Sherlocks Challenge

![](/images/phantomcheck/phantomcheck.png)
*This is a write-up about a challenge I did a while back, Phantom Check which is part of the Sherlocks challenges in HackTheBox. It tests your knowledge in investigating WMI logs. WMI is the infrastructure for management data and operations on Windows-based operating systems*

> Description: Talion suspects that the threat actor carried out anti-virtualization checks to avoid detection in sandboxed environments. Your task is to analyze the event logs and identify the specific techniques used for virtualization detection. Byte Doctor requires evidence of the registry checks or processes the attacker executed to perform these checks.

We are given two event log files, Microsoft-Windows-Powershell and Windows-Powershell-Operational. The Windows-Powershell-Operational will be of great importance to us. I opened these with Event Viewer in Windows and started going through the logs. For most of the questions, I filtered the logs by doing a global search like wmi to get the wmi logs then going through those.

**Task 1: Which WMI class did the attacker use to retrieve model and manufacturer information for virtualization detection?**
> Going through the logs, we can see the following powershell script which they used, and the class is : Win32_ComputerSystem
![](/images/phantomcheck/task1.png)

**Task 2: Which WMI query did the attacker execute to retrieve the current temperature value of the machine?**
> Filtering the wmi logs, you can see the following that used to query the temperature, hence the query: SELECT * FROM MSAcpi_ThermalZoneTemperature
![](/images/phantomcheck/task2.png)

**Task 3: The attacker loaded a PowerShell script to detect virtualization. What is the function name of the script?**
> Going through the logs, you find the function that does all the work, finding if the machine is a VM. Here it is and it's called: Check-VM
![](/images/phantomcheck/task3.png)

**Task 4: Which registry key did the above script query to retrieve service details for virtualization detection?**
> Digging into the function, you find where it queries the service details by calling the registry key: HKLM:\SYSTEM\ControlSet001\Services
![](/images/phantomcheck/task4.png)

**Task 5: The VM detection script can also identify VirtualBox. Which processes is it comparing to determine if the system is running VirtualBox?**
> Going to the virtualbox block which is even commented, you can see the script query vboxservice.exe and vboxtray.exe to identify virtualbox environments.
![](/images/phantomcheck/task5.png)

**Task 6: The VM detection script prints any detection with the prefix 'This is a'. Which two virtualization platforms did the script detect?**
> I switched to the Microsoft-Windows-Powershell logs to see what had happened, and since we know how the script prints the output when it finds a VM, we search for the output here "This is a". I got the following logs hence Hyper-V and VMWare was found.
![](/images/phantomcheck/task6.png)

Easy but awesome stuff :)