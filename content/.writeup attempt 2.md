> [!info] 📦 Box Info: CCTV
> **OS:** 🐧 Linux
> **Difficulty:** 🟢 Easy
> **Release Date:** 7 March 2026
> **Points:** 20

---
### getting foothold:

Начинаем с nmap скана:
```
nmap -sC -sV -vv 10.129.4.42
```
![[Pasted image 20260309163428.png]]

2 открытых порта. Добавляем домен в хосты. Переходим на сайт. 
![[Pasted image 20260309163634.png]]
Ввожу следующие креды:
```
admin:admin
```
И попадаю на такую панель:
 
![[Pasted image 20260309163819.png]]

Вправом вверхнем углу видим версию сервиса. Ищем уязвимости по данной версии и находим следующее:

https://github.com/ZoneMinder/zoneminder/security/advisories/GHSA-qm8h-3xvf-m7j3

![[Pasted image 20260310230213.png]]

Это blind sql injeciton. Я использовал sqlmap для автоматизации процесса. И нашел следующие креды:

mark - $2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIFQcURnm5N.rhlULwM0jrtbm

Это `bcrypt` хэш. Запускаем hashcat и брутим хеш по рокъю вордлисту:

```
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

Получаем пароль:
```
mark:$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.:opensesame
```

Подключаем по secure shell-у в аккаунт марка. Скачиваем linpeas.sh на уязвимую машину. Запускаем:

```
./linpeas.sh
```

find 5000 active port that sends superadmin creds...

USERNAME=sa_mark;PASSWORD=X1l9fx1ZjS7RZb;CMD=disk-info

authenticate with su_mark -> get user.txt

---
### accessing root:

В home директории видим пдф файл, скачиваем его. Нам говорят, что появилась какая-то новая платформа:

![[Pasted image 20260310230742.png]]

Видим странный порт 8765:
![[Pasted image 20260310230659.png]]

Немного походя по директория. Я нашел вот такой сервис, со следующей версией.

![[Pasted image 20260310230817.png]]

Делаем port-forwarding технику. Для того, чтобы с я мог со своего компа открыть сервис работающий на коробке на интересном порту:

```
ssh -L 8765:127.0.0.1:8765 sa_mark@10.129.5.172
```

Также я нашел креды для admin-а в наш сервис.

![[Pasted image 20260310230927.png]]

Перед нами какая-то система для управления камерами.

![[Pasted image 20260310230859.png]]

По данной версии я нашел CVE. Сначала я пробовал эксплуатировать все в ручную, но у меня не вышло... Пришлось использовать metasploit.

![[Pasted image 20260310230834.png]]

https://www.rapid7.com/db/modules/exploit/linux/http/motioneye_auth_rce_cve_2025_60787/

https://www.exploit-db.com/exploits/52481

Запускаем метасплоит. Находим данную CVE-шку. Вводим необходимые параметры и начинаем атаку:

```
> msfconsole

```
![[Pasted image 20260310231116.png]]
![[Pasted image 20260310231135.png]]
![[Pasted image 20260310231149.png]]

Получаем root сессию и получаем наш флаг.
![[Pasted image 20260310231206.png]]

