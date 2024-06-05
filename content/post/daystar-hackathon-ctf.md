+++
title = 'Daystar Hackathon CTF'
date = 2024-06-05T16:47:16-04:00
+++

# Daystar Hackathon CTF writeups

![leaderboard](/images/daystar/daystarleaderboard.png)
*My team CyberKimathi participated in the hackathon held at Daystar University. Being just a 2 hrs CTF and to be determined by time, we solved a couple of challenges, toping the competition. Here is a writeup of some:*

Challenge 1: Threat Intelligence
> A Malicious actor has been using the IP 64.176.194.7 to spread malware, can you identify the filename to the Powershell script used in the malicious activity. (Enclose the name within)

Using [VirusTotal](https://www.virustotal.com/), I searched the IP address of the malicious actor. I saw they are labelled AS-CHOOPA had been flagged as malicious by a couple vendors as malicious. Checking the Relations, you find the file they have been using and since you are asked for the powershell file, hence it is **gtyt.ps1.txt**
![VirusTotal](/images/daystar/daystarVT.png)

Challenge 2: Forensics
> Within the dump below you will hear the echoes of the past, can you decipher them and retreive your flag? See you on the other side.

You are given a link to download a file which turns out to be DOS/MBR boot. From the challenge's name 'Echos of the Past', I thought of deleted files. I decided to use autopsy for the challenge. Looking through the files after analyzing, a directory /android-9.0-r2/data/data/com.example.android.notepad/databases/ has interesting db files. Viewing the wal file, it has a secret note:
![Secret Note](/images/daystar/daystarforensicsdbcontents.png)

Decoding it with cyberchef,
![flag](/images/daystar/daystarforensicsdeciphered.png)

Learned!!