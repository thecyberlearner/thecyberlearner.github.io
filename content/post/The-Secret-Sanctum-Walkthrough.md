+++
title = 'The Secret Sanctum Walkthrough'
date = 2024-07-11T05:52:25-04:00
summary = 'This is a walkthrough of a simple web challenge I created for CyberKimathi CTF competition.'
+++

# The Secret Sanctum Walkthrough

![](/images/thesecretsanctum/thesecretsanctum.png)
*This is just a simple challenge I created for the purpose of understanding web applications and some of the most popular vulnerabilities used to attack web applications. This challenge needed understanding of url redirection, sql injections and Cross Site Scripting.*

Visiting the url given, we realize that the application redirects us to another page. Though, in the redirection, their is a string passed. Intercepting this with burpsuite to inspect closely, we find it is a base64 string:
![](/images/thesecretsanctum/thesecretsanctumflag1intercept.png)

Decoding it, we get the flag
![](/images/thesecretsanctum/thesecretsanctumflag1.png)

Proceeding, we find a page that tells us to explore the hidden secrets. I don't know about you but I first think of viewing the page source for comments after hearing this.
![](/images/thesecretsanctum/thesecretsanctumrobotssecret.png)

The comments say that there is always a place they don't want you to find. There is a page in web applications called robots.txt that states pages that they don't want indexed. Visiting it:
![](/images/thesecretsanctum/thesecretsanctumrobots.png)

So we visit the page /batcave.php. We are presented with a login page.
Trying default creds like 'admin':'admin' and 'admin':'password' doesn't work and says incorrect credentials.
![](/images/thesecretsanctum/thesecretsanctumbatcave.png)

But then trying to invoke an error by adding a character to the parameters, like admin' for the username causes an error and the whole page disappers. This means that input is not being filtered and it is vulnerable to sql injection. Try this payload for username and anything for password:
> admin' OR 1=1;--

**Explanation**: the ' closes the parameter, then since 1 is equal to 1, this returns True. Now since the second part of the statement is true and we have used OR to mean if any part of the statement is true the whole is true, and commented everything else, it means no matter what we enter for the password, the login form returns true hence we are logged in. This log us in.

We find a page that insists for secrets.
![](/images/thesecretsanctum/thesecretsanctumsqliexploited.png)

Viewing page source, we get a base64 for the second flag
![](/images/thesecretsanctum/thesecretsanctumflag2.png)

Now looking at the functionality of the page we are on, it has a form that needs you to identify yourself to check if you are a superhero. Entering your name, it returns the statement You truly are a superhero, then your name.\
Checking if it filters user input, enter something like "name>". It does not filter
![](/images/thesecretsanctum/thesecretsanctumxssexplanation.png)

That means the application will suffer from XSS due to this. Now use the payload
```
<script>alert(1);</script>
```

This throws an alert, then discloses the flag which is encoded.
![](/images/thesecretsanctum/thesecretsanctumflag3.png)

Decoding this with [cyberchef](https://gchq.github.io/CyberChef/), it is not base64. Use the magic operation, it discloses the string to be base58. Decoding:
![](/images/thesecretsanctum/thesecretsanctumflag3encoded.png)

FUN isn't it?
