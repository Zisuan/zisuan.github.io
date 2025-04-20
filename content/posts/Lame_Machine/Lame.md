---
title: "HackTheBox - Lame (Retired Machine) Walk Through"
date: 2025-04-20
hero: /images/sections/posts/htb.png
description: HackTheBox Retired Machine Basic Write Up
menu:
  sidebar:
    name: HTB Retired Machine - Lame
    identifier: Lame
    weight: 30
author:
  name: Zi Suan
  image: /images/author/profile.png
---

- **Machine Name**: `Lame`
- **Difficulty**: `Easy`

## Reconnaissance

Reconnaissance is a critical first step to identify exposed services and potential attack vectors on the target system.

### Nmap Scan

Guided Question: How many of the nmap top 1000 TCP ports are open on the remote host?

This can be found out by running the command:

```
nmap --top-ports=1000 ip {target ip address}
```

<br>
{{< img src="/posts/Lame_Machine/nmap.png" title="nmap scan" >}}
<br>
From the results, we can see that there are 4 open ports:

File Transfer Protocol (FTP) - Port 21
<br>
Secure Shell (SSH) - Port 22
<br>
NetBIOS SMB - Port 139
<br>
Server Message Block (SMB) - Port 445

### Service Enumeration

Guided Question: What version of VSFTPd is running on Lame?

We can further scan for the respective version of these services.

This can be found out by running the command:

```
nmap -sV {target ip address} -p21
```

<br>
{{< img src="/posts/Lame_Machine/ftp_version.png" title="ftp service version scan" >}}
<br>

From the result, we can see that FTP is running on vsftpd 2.3.4.

Guided Question: There is a famous backdoor in VSFTPd version 2.3.4, and a Metasploit module to exploit it. Does that exploit work here?

We can use Metasploit to check if there is any known vulnerability on vsftpd 2.3.4.

{{< img src="/posts/Lame_Machine/vsftpd.png" title="metasploit search" >}}
<br>
Indeed, an exploit can be found for vsftpd 2.3.4.

Before running the exploit, we have to configure some settings on the module.

These settings can be found by running the command: show options

{{< img src="/posts/Lame_Machine/msf_options.png" title="exploit module options" >}}
<br>
We have to set the target host to our target machine by running the command: set RHOSTS {target ip address}.

Since the target port is already set to port 21, we do not have to configure the target port.

Following which, we will try to run the exploit by running the command: exploit

{{< img src="/posts/Lame_Machine/exploit.png" title="exploit module" >}}
<br>
Unfortunately, there was no session created even after the exploit is completed. This might be due to the vulnerability being patched or not exploitable in this context. This also means that we have to find another way into the machine.

Next, we can look into the SMB ports.

Guided Question: What version of Samba is running on Lame? Give the numbers up to but not including "-Debian".

This can be found out by running the command:

```
nmap -p 445 -sC -sV {target ip address}
```

<br>
{{< img src="/posts/Lame_Machine/smb.png" title="smb version scan" >}}
<br>

From the result, we can see that SMB is running on Samba 3.0.20.

Guided Question: What 2007 CVE allows for remote code execution in this version of Samba via shell metacharacters involving the SamrChangePassword function when the "username map script" option is enabled in smb.conf?

Similarly, we can use the same steps as before to search for any vulnerabilities on this version.

{{< img src="/posts/Lame_Machine/samba.png" title="Samba metasploit search" >}}
<br>
From the result, there is only 1 known exploit that we can try.

Likewise, we have to configure the module settings before running the exploit.

{{< img src="/posts/Lame_Machine/exploit_smb.png" title="Samba exploit" >}}
<br>
The result shows that we have successfully exploited this vulnerability to gain command shell as a root user with root privileges.

## Post Exploitation

<br>

Using the command shell, we can use the command 'ls' to list files in the current directory and command 'cd' to traverse to the directory.

Running "ls" we can see that there is a home directory which we will traverse to using "cd /home" followed by another "ls" to list files in it.

A user makis can be found, by traversing into the user's home directory, a user.txt file can be found.

Running the command: "cat user.txt" will show us the content of the file.

{{< img src="/posts/Lame_Machine/userflag.png" title="user flag" width="50%" height="50%" class="center" >}}
</br>
**Voila! We got our User flag! 🏴‍☠️**

Using the same command shell, we can traverse around the files to find the root's home directory to look for the root flag.

cd root > ls > cat root.txt

{{< img src="/posts/Lame_Machine/rootflag.png" title="root flag" width="50%" height="50%">}}
</br>
**👑 Root Flag!**

Overall, This retired machine offers a solid introduction to service enumeration and basic exploitation using publicly available vulnerabilities. It also reinforces foundational techniques making it an excellent starting point for newcomers to penetration testing.
