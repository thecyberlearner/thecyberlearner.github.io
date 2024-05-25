+++
title = 'Logjammer'
date = 2024-05-25T01:12:56-04:00
summary = 'This challenge is part of the Sherlocks HackTheBox challenges which are mainly focused on improving your defensive skills.'
+++

# Logjammer
> ![](/images/logjammer.png)

I normally use [gigasheets](http://gigasheet.com/) to analyze event logs but for this challenge some of the logs proved to be quite challenging so I had to swtich to Windows and use event viewer.\

**Task 1 : When did the cyberjunkie user first successfully log into his computer? (UTC)**
> Opening the System.evtx with gigasheet, checking when the user cyberjunkie first appeared you'll get the answer
![](/images/logjammerfirstlogin.png)

**Task 2 : The user tampered with firewall settings on the system. Analyze the firewall event logs to find out the Name of the firewall rule added?**
> Going through the Firewall-Firewall.evtx file, I found the rule they tampered with
![](/images/logjammerfirewallrule.png)

**Task 3 : Whats the direction of the firewall rule?**
> This is where it got tricky for me and I had to swtich to Windows event viewer.  It was easier to use and vieweing the Firewall-Firewall.evtx file, I saw that a rule had been added to the defender list which is the one above. Below it is the direction
![](/images/logjammerfirewallrule.PNG)

**Task 4: The user changed audit policy of the computer. Whats the Subcategory of this changed policy?**
> By analyzing the Security.evtx logs, we can see a audit change log and below it is its subcategory
![](/images/logjammerauditploicychanged.PNG)

**Task 5: The user "cyberjunkie" created a scheduled task. Whats the name of this task?**
> Inspecting the Security.evtx logs, we see a log of id 4698, Other Object Access Events. Checking it further, it is the scheduled task
![](/images/logjammerscheduledtaskname.PNG)

**Task 6: Whats the full path of the file which was scheduled for the task?**
> Just below on the same task is the path of the file to execute
![](/images/logjammerscheduledtaskfile.PNG)

**Task 7: What are the arguments of the command?**
> The commands are right below the file

**Task 8: The antivirus running on the system identified a threat and performed actions on it. Which tool was identified as malware by antivirus?**
> I used gigasheet on this one. Definitely from the given logs, we would identify this from the Windows Defender-Operational.evtx logs. Looking through the logs, I caught this
![](/images/logjammermalware.png)\
Hence the name is SharpHound

**Task 9: Whats the full path of the malware which raised the alert?**
> The path is displayed in the same logs above

**Task 10: What action was taken by the antivirus?**
> By switching to the tab of Action, we find the action taken
![](/images/logjammermalwareaction.png)

**Task 11: The user used Powershell to execute commands. What command was executed by the user?**
> Analyze the Powershell-Operational.evtx logs and you see a log called 'Execute a Remote Command'. This is it hence just go through it
![](/images/logjammerpowershellcommand)

**Task 12: We suspect the user deleted some event logs. Which Event log file was cleared?**
> The trickiest one that made me hit so many dead ends. Though, after going through 'Windows Firewall-Firewall.evtx' logs, a log with id 2006 stated that 'A rule has been deleted in the Windows Defender Firewall exception list'. This is it
![](/images/logjammerruledeleted.PNG)

			*FUN CHALLENGE NGL*