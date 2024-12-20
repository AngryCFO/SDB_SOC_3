# Домашнее задание к занятию "Защита сети" - `Александра Бужор`

---

### Подготовка к выполнению заданий
Подготовка защищаемой системы:
- установите Suricata,
- установите Fail2Ban.

Подготовка системы злоумышленника: 
- установите nmap и thc-hydra либо скачайте и установите Kali linux.

Обе системы должны находится в одной подсети.

### Задание 1

Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

sudo nmap -sA < ip-адрес >

sudo nmap -sT < ip-адрес >

sudo nmap -sS < ip-адрес >

sudo nmap -sV < ip-адрес >

По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.

В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.

### Решение:

```bash
root@vm1:/home/udjin# service suricata status
● suricata.service - Suricata IDS/IDP daemon
     Loaded: loaded (/lib/systemd/system/suricata.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-03-07 13:32:09 UTC; 15s ago
       Docs: man:suricata(8)
             man:suricatasc(8)
             https://suricata.io/documentation/
    Process: 4617 ExecStart=/usr/bin/suricata -D --af-packet -c /etc/suricata/suricata.yaml --pidfile /run/suricata.pid (code=exited, status=0/SUCCESS)
   Main PID: 4618 (Suricata-Main)
      Tasks: 8 (limit: 2226)
     Memory: 42.0M
        CPU: 452ms
     CGroup: /system.slice/suricata.service
             └─4618 /usr/bin/suricata -D --af-packet -c /etc/suricata/suricata.yaml --pidfile /run/suricata.pid

Mar 07 13:32:09 vm1 systemd[1]: Starting suricata.service - Suricata IDS/IDP daemon...
Mar 07 13:32:09 vm1 suricata[4617]: i: suricata: This is Suricata version 7.0.0 RELEASE running in SYSTEM mode
Mar 07 13:32:09 vm1 systemd[1]: Started suricata.service - Suricata IDS/IDP daemon.
root@vm1:/home/udjin# service fail2ban status
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-03-07 13:23:09 UTC; 9min ago
       Docs: man:fail2ban(1)
   Main PID: 2037 (fail2ban-server)
      Tasks: 5 (limit: 2226)
     Memory: 12.5M
        CPU: 625ms
     CGroup: /system.slice/fail2ban.service
             └─2037 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

Mar 07 13:23:09 vm1 systemd[1]: Started fail2ban.service - Fail2Ban Service.
Mar 07 13:23:09 vm1 fail2ban-server[2037]: 2024-03-07 13:23:09,419 fail2ban.configreader   [2037]: WARNING 'allowipv6' not defined in 'Definition'. Using default one: 'auto'
Mar 07 13:23:09 vm1 fail2ban-server[2037]: Server ready
```

Завел новый  фильтр в /etc/fail2ban/filter.d/

suricata.conf
```
[INCLUDES]
before = common.conf

[DEFAULT]
_daemon = suricata

[Definition]
datepattern = ^%%m/%%d/%%Y-%%H:%%M:%%S
failregex = <HOST>:[0-9]+ ->
ignoreregex =
```

Завел правило в /etc/fail2ban/jail.d

suricata.local
```
[suricata]
enabled = true
filter = suricata
logpath = /var/log/suricata/fast.log
findtime = 3h
banaction_allports = nftables[type=allports]
maxretry = 2
bantime = 24
```

```bash
┌──(root㉿kali)-[/home/kali]
└─# nmap -sV 192.168.1.82
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-07 18:46 MSK
Nmap scan report for 192.168.1.82
Host is up (0.0017s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.3p1 Ubuntu 1ubuntu3.2 (Ubuntu Linux; protocol 2.0)
MAC Address: 08:00:27:A2:E9:84 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds
                                                                                                                                                                                                                  
┌──(root㉿kali)-[/home/kali]
└─# nmap -sA 192.168.1.82
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-07 18:48 MSK
Nmap scan report for 192.168.1.82
Host is up (0.00054s latency).
All 1000 scanned ports on 192.168.1.82 are in ignored states.
Not shown: 1000 unfiltered tcp ports (reset)
MAC Address: 08:00:27:A2:E9:84 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds
                                                                                                                                                                                                                  
┌──(root㉿kali)-[/home/kali]
└─# nmap -sT 192.168.1.82
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-07 18:48 MSK
Nmap scan report for 192.168.1.82
Host is up (0.0025s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:A2:E9:84 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.25 seconds
                                                                                                                                                                                                                  
┌──(root㉿kali)-[/home/kali]
└─# nmap -sS 192.168.1.82
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-07 18:48 MSK
Nmap scan report for 192.168.1.82
Host is up (0.00085s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:A2:E9:84 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.30 seconds
```

Из предоставленного лога видно, что система Suricata успешно обнаружила и зарегистрировала несколько попыток сканирования на различные порты, включая порты, связанные с популярными базами данных и службами (например, SSH, MySQL, PostgreSQL, MSSQL, Oracle SQL и VNC)

```bash
udjin@vm1:~$ sudo tail -f /var/log/suricata/fast.log
03/07/2024-15:46:59.469976  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:3306
03/07/2024-15:46:59.469976  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:3306
03/07/2024-15:46:59.484779  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:1521
03/07/2024-15:46:59.484779  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:1521
03/07/2024-15:46:59.498387  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:5901
03/07/2024-15:46:59.498387  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:5901
03/07/2024-15:46:59.520944  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:1433
03/07/2024-15:46:59.520944  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:1433
03/07/2024-15:46:59.523422  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:5815
03/07/2024-15:46:59.523422  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:5815
03/07/2024-15:46:59.540660  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:5432
03/07/2024-15:46:59.540660  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:43024 -> 192.168.1.82:5432
03/07/2024-15:48:29.320983  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:34860 -> 192.168.1.82:3306
03/07/2024-15:48:29.320983  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:34860 -> 192.168.1.82:3306
03/07/2024-15:48:29.327743  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:42632 -> 192.168.1.82:5432
03/07/2024-15:48:29.327743  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:42632 -> 192.168.1.82:5432
03/07/2024-15:48:29.353712  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:38494 -> 192.168.1.82:5906
03/07/2024-15:48:29.353712  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:38494 -> 192.168.1.82:5906
03/07/2024-15:48:29.371779  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:56344 -> 192.168.1.82:1433
03/07/2024-15:48:29.371779  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:56344 -> 192.168.1.82:1433
03/07/2024-15:48:29.375578  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:58100 -> 192.168.1.82:1521
03/07/2024-15:48:29.375578  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:58100 -> 192.168.1.82:1521
03/07/2024-15:48:29.390318  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:51332 -> 192.168.1.82:5810
03/07/2024-15:48:29.390318  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:51332 -> 192.168.1.82:5810
03/07/2024-15:48:35.392875  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:3306
03/07/2024-15:48:35.392875  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:3306
03/07/2024-15:48:35.430072  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:5432
03/07/2024-15:48:35.430072  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:5432
03/07/2024-15:48:35.475102  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:1521
03/07/2024-15:48:35.475102  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:1521
03/07/2024-15:48:35.493258  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:1433
03/07/2024-15:48:35.493258  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.81:59874 -> 192.168.1.82:1433
03/07/2024-15:58:16.897228  [**] [1:2001219:20] ET SCAN Potential SSH Scan [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37284 -> 192.168.1.82:22
03/07/2024-15:58:16.897228  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37284 -> 192.168.1.82:22
03/07/2024-15:58:16.897228  [**] [1:2001219:20] ET SCAN Potential SSH Scan [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37284 -> 192.168.1.82:22
03/07/2024-15:58:16.899652  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37326 -> 192.168.1.82:22
03/07/2024-15:58:16.897228  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37284 -> 192.168.1.82:22
03/07/2024-15:58:16.902426  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37358 -> 192.168.1.82:22
03/07/2024-15:58:16.899652  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37326 -> 192.168.1.82:22
03/07/2024-15:58:16.902426  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37358 -> 192.168.1.82:22
03/07/2024-15:58:52.850158  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:58852 -> 192.168.1.82:22
03/07/2024-15:58:52.850158  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:58852 -> 192.168.1.82:22
```

Fail2Ban, в свою очередь, отреагировал на эти логи, автоматически заблокировав IP-адрес 192.168.1.81 после обнаружения множественных попыток сканирования. Бан и последующий анбан IP-адреса говорят о том, что Fail2Ban применяет временные меры блокировки (согласно конфигурации правила) для предотвращения потенциальных атак.

```bash
2024-03-07 16:25:19,203 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,204 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,318 fail2ban.actions        [13153]: NOTICE  [suricata] Ban 192.168.1.81
2024-03-07 16:25:19,326 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,333 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,359 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,360 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,418 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,420 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,448 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:19,449 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:19
2024-03-07 16:25:21,306 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:21
2024-03-07 16:25:21,307 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:21
2024-03-07 16:25:21,408 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:21
2024-03-07 16:25:21,410 fail2ban.filter         [13153]: INFO    [suricata] Found 192.168.1.81 - 2024-03-07 16:25:21
2024-03-07 16:25:45,665 fail2ban.actions        [13153]: NOTICE  [suricata] Unban 192.168.1.81
```

---

### Задание 2

Проведите атаку на подбор пароля для службы SSH:

hydra -L users.txt -P pass.txt < ip-адрес > ssh

Настройка hydra:
- создайте два файла: users.txt и pass.txt;
- в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

Дополнительная информация по hydra: https://kali.tools/?p=1847.

Включение защиты SSH для Fail2Ban:
- открыть файл /etc/fail2ban/jail.conf,
- найти секцию ssh,
- установить enabled в true.

Дополнительная информация по Fail2Ban:https://putty.org.ru/articles/fail2ban-ssh.html.

В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.

### Решение:

#### Hydra

Судя по выводу Hydra, в результате попытки брутфорса SSH-сервера по адресу 192.168.1.82, с использованием списков пользователей и паролей из набора Metasploit (mirai_user.txt и mirai_pass.txt), в итоге не было найдено ни одного верного пароля, и процесс завершился с ошибками из-за слишком многих неудачных попыток подключения.

```bash
┌──(root㉿kali)-[/home/kali]
└─# hydra -L /usr/share/wordlists/metasploit/mirai_user.txt -P /usr/share/wordlists/metasploit/mirai_pass.txt 192.168.1.82 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-03-07 18:58:16
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 645 login tries (l:15/p:43), ~41 tries per task
[DATA] attacking ssh://192.168.1.82:22/
[STATUS] 49.00 tries/min, 49 tries in 00:01h, 602 to do in 00:13h, 10 active
[ERROR] all children were disabled due too many connection errors
0 of 1 target completed, 0 valid password found
[INFO] Writing restore file because 2 server scans could not be completed
[ERROR] 1 target was disabled because of too many errors
[ERROR] 1 targets did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-03-07 18:59:40
```

#### Suricata

В логах Suricata видим серию предупреждений о потенциальных попытках сканирования SSH, исходящих от IP 192.168.1.81 в направлении 192.168.1.82. 
Suricata классифицировала это как "Попытка утечки информации" с приоритетом 2. 

```bash
03/07/2024-15:58:16.899652  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37326 -> 192.168.1.82:22
03/07/2024-15:58:16.897228  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37284 -> 192.168.1.82:22
03/07/2024-15:58:16.902426  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37358 -> 192.168.1.82:22
03/07/2024-15:58:16.899652  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37326 -> 192.168.1.82:22
03/07/2024-15:58:16.902426  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:37358 -> 192.168.1.82:22
03/07/2024-15:58:52.850158  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:58852 -> 192.168.1.82:22
03/07/2024-15:58:52.850158  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:58852 -> 192.168.1.82:22
03/07/2024-15:58:53.861663  [**] [1:2003068:7] ET SCAN Potential SSH Scan OUTBOUND [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.81:58854 -> 192.168.1.82:22
```

#### Fail2Ban

Fail2Ban, в свою очередь, заметил множественные неудачные попытки подключения к SSH (порт 22) от IP 192.168.1.81 и зарегистрировал их в своих логах. Атакующий был заблокирован.

```bash
2024-03-07 15:58:17,062 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,062 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,062 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,062 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,062 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,063 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,063 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,063 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,071 fail2ban.filter         [2037]: INFO    [sshd] Found 192.168.1.81 - 2024-03-07 15:58:17
2024-03-07 15:58:17,439 fail2ban.actions        [2037]: NOTICE  [sshd] Ban 192.168.1.81
```
