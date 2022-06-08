---
layout: post
title: Under The Wire
---

This post is an overview of my time working through the wargame "century" found in [underthewire](https://underthewire.tech/)

---

![](/assets/image/attachmentsutw-bg.png)

My PowerShell skills are... _lacking_. I very often need to look up How to do X, how to do Y,
you get the idea. So I've decided to change that by working through UnderTheWire wargames, an 
interactive way to get familiar with PowerShell. When I was starting out in the linux world OverTheWire 
wargames were a tremendous help, I am confident the same will apply here. I am starting with 
Century the first of the available games.

## Century - 0 -> 1
login to the machine

### Creds
century1:century1

connect with ssh

```powershell
ssh century1@century.underthewire.tech
```

all further ssh connections will be in this format (I think, if different I will elaborate) so I won't repeat these for each challenge

## Century - 1 -> 2
The password for Century2 is the build version of the instance of PowerShell installed on this system.

After some time fumbling looking for the wrong thing I found:

```powershell
PS C:\users\century1\desktop> $PSVersionTable

Name                           Value                                               
----                           -----                                               
PSVersion                      5.1.14393.3866                                     
PSEdition                      Desktop                                             
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}                             
BuildVersion                   10.0.14393.3866                                     
CLRVersion                     4.0.30319.42000                                     
WSManStackVersion              3.0                                                 
PSRemotingProtocolVersion      2.3                                                 
SerializationVersion           1.1.0.1    
```

```powershell
BuildVersion                   10.0.14393.3866
```

century2:10.0.14393.3866

## Century - 2 -> 3
The password for Century3 is the name of the built-in cmdlet that performs the wget like function within PowerShell PLUS the name of the file on the desktop.

we get the filename with a simple `dir`
filename: 443

I think the command hinted at is `Invoke-WebRequest` but whether it requires it asforementioned or in the shorthand `iwr` will require trial and error

The former was correct so with converting to lowercase the credentials are
century3:invoke-webrequest443

## Century - 3 -> 4
The password for Century4 is the number of files on the desktop.

with some stack-overflow browsing I have my answer

```powershell
Write-Host ( Get-ChildItem C:\users\century3\desktop | Measure-Object).Count;
```

century4:123
## Century - 4 -> 5
The password for Century5 is the name of the file within a directory on the desktop that has spaces in its name.

I think I am misunderstanding this one somehow. I thought the file inside the directory should have spaces in it's name so I came up with the command

```powershell
Get-ChildItem -Path .\ -Recurse | Where-Object {$_.Name -like "* *"}
```

but this only returns the directory `Not Me`
reading again I think maybe it means a file within `Not Me` but this directory
with some trial and error there is a file in `OpenMe` called `61580` which turned out to be the password, but since neither have whitespace in their name I am at a total loss as to what the intended solution was here.

regardless I got through, here are the creds
century5:61580

## Century - 5 -> 6
The password for Century6 is the short name of the domain in which this system resides in PLUS the name of the file on the desktop.

running:

```powershell
PS C:\users\century5\desktop> get-addomain                                                                                                      
AllowedDNSSuffixes                 : {}
ChildDomains                       : {}
ComputersContainer                 : CN=Computers,DC=underthewire,DC=tech
DeletedObjectsContainer            : CN=Deleted Objects,DC=underthewire,DC=tech
DistinguishedName                  : DC=underthewire,DC=tech
DNSRoot                            : underthewire.tech
DomainControllersContainer         : OU=Domain Controllers,DC=underthewire,DC=tech
DomainMode                         : Windows2016Domain
DomainSID                          : S-1-5-21-758131494-606461608-3556270690
ForeignSecurityPrincipalsContainer : CN=ForeignSecurityPrincipals,DC=underthewire,DC=tech
Forest                             : underthewire.tech
InfrastructureMaster               : utw.underthewire.tech
LastLogonReplicationInterval       : 
LinkedGroupPolicyObjects           : {cn={ECB4A7C0-B4E1-41B1-9E89-161CFA679999},cn=policies,cn=system,DC=underthewire,DC=tech, CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=underthewire,DC=tech}
LostAndFoundContainer              : CN=LostAndFound,DC=underthewire,DC=tech
ManagedBy                          : 
Name                               : underthewire
NetBIOSName                        : underthewire
ObjectClass                        : domainDNS
ObjectGUID                         : bdccf3ad-b495-4d86-a94c-60f0d832e6f0
ParentDomain                       : 
PDCEmulator                        : utw.underthewire.tech
PublicKeyRequiredPasswordRolling   : True
QuotasContainer                    : CN=NTDS Quotas,DC=underthewire,DC=tech
ReadOnlyReplicaDirectoryServers    : {}
ReplicaDirectoryServers            : {utw.underthewire.tech}
RIDMaster                          : utw.underthewire.tech
SubordinateReferences              : {DC=ForestDnsZones,DC=underthewire,DC=tech, DC=DomainDnsZones,DC=underthewire,DC=tech, CN=Configuration,DC=underthewire,DC=tech}
SystemsContainer                   : CN=System,DC=underthewire,DC=tech
UsersContainer                     : CN=Users,DC=underthewire,DC=tech
```
```
Name                               : underthewire
```

I believe what we are looking for is `underthewire` grabbing the filename on the desktop of `3347` I try

century6:underthewire3347

which is correct

## Century - 6 -> 7
The password for Century7 is the number of folders on the desktop.
I believe this should just be a tweak on our earlier command to count directories

```powershell
Write-Host ( Get-ChildItem . -Attributes Directory | Measure-Object).Count; 
```

I actually repeated without `-Attributes Directory`  and it gave the same number, so filtering was really necessary but still good practice

century7:197

## Century - 7 -> 8
The password for Century8 is in a readme file somewhere within the contacts, desktop, documents, downloads, favorites, music, or videos folder in the user’s profile.

I think here I can reuse the recursive filename search technique I used when looking for the whitespace

running:

```powershell
Get-ChildItem -Path .\ -Recurse | Where-Object {$_.Name -like "*readme*"}
```

we find it immediately inside of the downloads folder

century8:7points

## Century - 8 -> 9
The password for Century9 is the number of unique entries within the file on the desktop.

with some quick research I put together a command to get an array of the unique content that can then be put into the count structure I've used before

```powershell
Get-Content .\unique.txt | Select-Object -unique
```

```powershell
Write-Host (Get-Content .\unique.txt | Select-Object -unique | Measure-Object).Count
```

we get 696 unqiue entries, trying this as a password we get in

century9:696

## Century - 9 -> 10
The password for Century10 is the 161st word within the file on the desktop.

My first thought is that I can do this somehow through get content, so I will try to figure it out by reading the help page of the command, at least for a while before I have to resort to google :)

hmm not that helpful immediately (or I am oblivious) but I know it returns an array of content, maybe I can simply index it?

with

```powershell
(get-content .\Word_File.txt)[161]
```

I am returned `i` so clearly I am indexing the characters instead of the words, If I can make it return the words in the array this approach should work. I seen a delimiter option in the help command, maybe this is the delimiter to split upon.

```powershell
(get-content .\Word_File.txt -Delimiter " ")[161]
```

this returns `nonapplicabness` exactly what I wanted. Now provided these arrays are not zero indexed (for some reason I have a feeling this might be the case?) we should get into the next level

century10:nonapplicabness

yes it seems I was wrong the array is zero indexed... repeating the command with an index of 160

```powershell
(get-content .\Word_File.txt -Delimiter " ")[160]
```

we get `pierid`
century10:pierid

## Century 10 -> 11
The password for Century11 is the 10th and 8th word of the Windows Update service description combined PLUS the name of the file on the desktop.

with the techniques I've developed so far, all I need is how to get the Windows Update service description, I resorted to google here 

with some trial and erro of a few commands I arrived at

```powershell
(get-wmiobject win32_service | where-object {$_.Name -like "*wuauserv*"} | Select Description).Description
```

a little dirty but does the trick

now to index this for the words we want

upon trying this I realised I forgot to split the string into an array

```powershell
(get-wmiobject win32_service | where-object {$_.Name -like "*wuauserv*"} | Select Description).Description.Split(" ")
```

indexing for the 8th element (7th index) we get `updates`
indexing for the 10th element (9th index) we get `windows`

combining with the filename `110` from the desktop we get

centruy11:windowsupdates110

## Century 11-> 12
The password for Century12 is the name of the hidden file within the contacts, desktop, documents, downloads, favorites, music, or videos folder in the user’s profile.

we can do this with a command similar to the recursive search from before, adding the    -Hidden argument

```powershell
Get-ChildItem -Path .\ -Recurse -Hidden 
```

quite a bit of permission denied but we see the file `secret_sauce` in downloads

century12:secret_sauce

## Century 12-> 13
The password for Century13 is the description of the computer designated as a Domain Controller within this domain PLUS the name of the file on the desktop.

filename: \_things

it took quite a while to find the right command, eventually I settled on 

```powershell
Get-adcomputer -filter * -Property Description
```
we see a description of i\_authenticate
so we have
century13:i_authenticate_things
## Century 13-> 14
The password for Century14 is the number of words within the file on the desktop.

we can do similar to before, split into array -> put into count structure used above

```powershell
Write-Host (get-content .\countmywords -delimiter " " | measure-object).count
```

755

century14:755
## Century 14-> 15
The password for Century15 is the number of times the word “polo” appears within the file on the desktop.

I used a sligthly different approach to counting here to mix things up

```powershell
(get-content .\countpolos -delimiter " " | select-string -pattern "^polo").length
```

## Final Thoughts
A good wargame, I definitely learned some tricks around manipulating objects in powershell that I didn't know when I started. I look forward to the next stage.
