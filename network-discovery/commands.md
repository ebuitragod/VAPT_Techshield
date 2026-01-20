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

nmap -sn -PS22,80,443 192.168.54.20/24 -oA hosts_tcp_ping

nmap -sn -PR 192.168.54.20/24 -oA hosts_arp
```

Services
```
nmap -sV -sC -p 22,80,135,139,443,445,554,2869,3306,3389,10243 192.168.54.20 -oA service_scan
```

Running basic vulnerability scripts:
```
nmap --script vuln 192.168.57.20
```
![nmap-script-vuln](nmap-script-vuln.png)


