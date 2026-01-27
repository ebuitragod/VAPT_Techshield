# Critical founds:

## Vulnerabilities:

Critical: 
```
smb-vuln-ms17-010: VULNERABLE
Remote Code Execution vulnerability in Microsoft SMBV1 servers
CVE-2017-0143 - Risk factor: HIGH
```

High:
```
http-slowloris-check: LIKELY VULNERABLE
CVE-2007-6750 - Denial of Service
```

Open Ports:

`445/tcp (SMB)`: Vulnerable to EternalBlue

`135/tcp, 139/tcp`: Legacy Windows services with several vulnerabilities

`2869/tcp, 5357/tcp, 10243/tcp`: exposed HTTPAPI 

