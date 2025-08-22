## Host Header – Test

Tool: Host Header Inchecktion (Burp Suite)

#### Check for flawed validation
```
GET /example HTTP/1.1
Host: vulnerable-website.com:bad-stuff-here
```
```
GET /example HTTP/1.1
Host: hacked-subdomain.vulnerable-website.com
```

#### Send ambiguous requests
Inject duplicate Host headers
```
GET /example HTTP/1.1
Host: vulnerable-website.com
Host: bad-stuff-here
```
Supply an absolute URL
```
GET https://vulnerable-website.com/ HTTP/1.1
Host: bad-stuff-here
```
Add line wrapping
```
GET /example HTTP/1.1
    Host: bad-stuff-here
Host: vulnerable-website.com
```

#### Inject host override headers
```
GET /example HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: bad-stuff-here
```
```
    X-Host
    X-Forwarded-Server
    X-HTTP-Host-Override
    Forwarded
```

## Exploit