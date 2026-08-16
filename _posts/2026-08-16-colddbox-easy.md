---
title: "ColddBox: Easy — RCE WordPress via l'éditeur de thème + privesc GTFOBins"
date: 2026-08-16 10:00:00 +0200
categories: [WriteUp, TryHackMe]
tags: [wordpress, wpscan, gtfobins, privilege-escalation]
---

Machine easy TryHackMe https://tryhackme.com/room/colddboxeasy

### Task 1 : boot2Root
Can you get access and get both **flags**?
Good Luck!.﻿
By Marti from Hixec.
**Doubts and / or help in [Hixec Community](https://discord.gg/bMWXR6Z).**  
_Thumbnail box image credits, designed by [Freepik](https://www.flaticon.com/authors/freepik) from [www.flaticon.es](https://www.flaticon.es)_  

**Answer the questions below**
**user.txt** :
Walktrhought : 
#### Etape 1 : Scan nmap 
nmap -sV -A 10.81.137.160               
![alt](/assets/img/Colddbox/scan_nmap.png)
Can see a web site with wordpress 4.1.31
Try to go scan web
![alt](/assets/img/Colddbox/scan_web.png)
Found Hidden page
![alt](/assets/img/Colddbox/hidden_page.png)
Go to Wpscan :
``wpscan --url 10.81.137.160 -P /usr/share/wordlists/rockyou.txt
![alt](/assets/img/Colddbox/wp_scan.png)
```
Username and password founded : 
C0ldd / 9876543210
```
Now go to coonect on wp-admin page
After thoroughly exploring the website and research, proceed to “Appearance” and then select “Editor”. Look for the 404.php tab on the right-hand side and click to open it, as this is where we will be working next

Put code of Revershell like https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php (don't forget to change port and IP)
And update the file.

Start nc connection in a shell :
```bash
nc -lvnp 1234
```
Now go on your browser and connect to :
`http://IP/wp-content/themes/twentyfifteen/404.php'

YOU GOT SHELL !

![alt](/assets/img/Colddbox/reverse_shell.png)
Upgrade the terminal :
```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@ColddBox-Easy:/$ 
```
We can't cat the user.txt file with www-data user
So we try to search password in wp-config (if it's existed)

![alt](/assets/img/Colddbox/upgrade_shell.png)
And cat the user.txt file : 
RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==

#### **root.txt**
To get access root, do escalation privilege : 
![alt](/assets/img/Colddbox/find_vuln.png)
Found a GTFObin with vim : https://gtfobins.github.io/gtfobins/vim/
![alt](/assets/img/Colddbox/gtfo.png)
And ROOT SHELL SPAWN
![alt](/assets/img/Colddbox/escalation.png)
![alt](/assets/img/Colddbox/flag_root.png)
Root Flag : wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=
