# Host Header – Defensive Test Cheat Sheet (Lab‑Only)

> **Scope & Ethics**  
> Dùng *chỉ* trong lab hoặc khi bạn **được ủy quyền** kiểm thử bảo mật. Không áp dụng vào hệ thống mà bạn không sở hữu/không được phép. Nội dung bên dưới phục vụ **phòng thủ** và đánh giá an toàn.

## Cách sử dụng nhanh
Sau mỗi phép thử, quan sát 3 thứ:
1) **Phản hồi**: mã trạng thái, redirect (`Location`), body.  
2) **Phản xạ**: giá trị `Host` có xuất hiện trong HTML/JSON/email/template không.  
3) **Log/trace**: chênh giữa proxy/LB và app (tên miền nào được dùng).

---

## 1 Chấp nhận Host “lạ” (default vhost)
- **Mục tiêu:** Xem ứng dụng có chấp nhận `Host` ngoài whitelist không.
- **Thử (lab):** Gửi yêu cầu với `Host` là một miền không hợp lệ (vd. `not-your-domain.test`).
- **Dấu hiệu dương tính:** Server trả 200/302 “bình thường”, nội dung/redirect phản xạ miền lạ.

## 2 Ambiguity: URL tuyệt đối vs. `Host`
- **Mục tiêu:** Kiểm tra ưu tiên parse khi request-line là URL tuyệt đối nhưng `Host` khác.
- **Thử (lab):** Dùng URL tuyệt đối trong request-line đồng thời đặt `Host` khác tên miền đó.
- **Dấu hiệu dương tính:** Backend/redirect/cookie domain dựa vào miền “sai”.

## 3 `Host` trùng lặp/không chuẩn
- **Mục tiêu:** Một số thành phần chọn `Host` đầu/cuối khi có nhiều/không chuẩn.
- **Thử (lab):** Tạo điều kiện có **nhiều giá trị** `Host` hoặc `Host` có ký tự khoảng trắng lạ.
- **Dấu hiệu dương tính:** Proxy và app hiểu **khác** nhau về miền đích.

## 4 Header ghi đè host qua proxy
- **Mục tiêu:** Backend có tin `X-Forwarded-Host`/`Forwarded` không.
- **Thử (lab):** Đặt `X-Forwarded-Host` thành miền “lạ” khi đi qua proxy/layer tin cậy.
- **Dấu hiệu dương tính:** Link tuyệt đối/redirect/email token sinh theo miền bị ghi đè.

## 5 Password‑reset poisoning (lab‑only)
- **Mục tiêu:** App dựng URL email từ `Host` không tin cậy.
- **Thử (lab):** Kích hoạt quy trình “quên mật khẩu” khi `Host` bị đổi (hoặc bị ghi đè như #4).
- **Dấu hiệu dương tính:** Link trong email trỏ đến miền lạ (rủi ro chiếm đoạt token).

## 6 Web cache poisoning qua `Host`
- **Mục tiêu:** Cache key thiếu thuộc tính `Host` → phát tán phản hồi độc.
- **Thử (lab):** Truy cập tài nguyên có khả năng được cache khi `Host` là miền lạ.
- **Dấu hiệu dương tính:** Yêu cầu hợp lệ sau đó nhận nội dung đã “nhiễm”/phản xạ miền lạ.

## 7 Open‑redirect/URL generation phụ thuộc `Host`
- **Mục tiêu:** 30x hoặc các URL tuyệt đối được dựng từ `Host` không chuẩn hóa.
- **Thử (lab):** Truy cập endpoint tạo redirect/ký tự điều hướng khi `Host` thay đổi.
- **Dấu hiệu dương tính:** Header `Location`/link tuyệt đối chứa miền lạ.

## 8 Routing/SSRF qua `Host`
- **Mục tiêu:** LB/vhost định tuyến dựa trên `Host` → có thể “lọt” vào service nội bộ.
- **Thử (lab):** Đặt `Host` tương ứng với tên vhost/service nội bộ (trong lab).
- **Dấu hiệu dương tính:** Log/trace cho thấy request tới backend nội bộ trái phép.

## 9 Biến thể cú pháp: cổng, hoa/thường, IDN/punycode
- **Mục tiêu:** So khớp lỏng lẻo có thể coi các biến thể là “cùng miền”.
- **Thử (lab):** Đổi chữ hoa/thường, thêm `:80/:443`, hoặc dùng tên miền IDN/punycode tương tự.
- **Dấu hiệu dương tính:** Cookie/redirect/CSRF origin lệch sang biến thể.

## 10 Header xấu & length/whitespace edge‑cases
- **Mục tiêu:** Một số parser dừng sớm/ghép header sai → bất nhất giữa tầng.
- **Thử (lab):** Dùng khoảng trắng lạ/độ dài bất thường quanh `Host` (trong phạm vi tiêu chuẩn an toàn).
- **Dấu hiệu dương tính:** 4xx/5xx lạ, hoặc proxy/app không khớp về miền đích.

---

## Dấu hiệu dương tính “mạnh”
- Response/redirect/email **phản xạ** giá trị `Host` tùy ý.  
- Cache phân phát nội dung sinh từ `Host` lạ cho người dùng khác.  
- Proxy log ≠ App log về tên miền đích (**dị tuyến**).

## Phòng thủ nhanh
- **Whitelist domain** ở **proxy + app** (vd. `server_name`, `ALLOWED_HOSTS`).  
- **Chuẩn hóa/vô hiệu** `X-Forwarded-Host`/`Forwarded`; chỉ tin proxy tin cậy.  
- Không dựng URL tuyệt đối từ `Host`; hoặc **ghim/whitelist** miền cho email/reset.  
- **Cache key** phải bao gồm `Host`; tránh phản xạ `Host` vào nội dung.  
- **Enforce chuẩn**: đúng **một** `Host`, khớp authority; từ chối request lệch chuẩn.

---

## Tài liệu tham khảo (đọc thêm)
- PortSwigger: *Host header attacks* (tổng quan kỹ thuật & biện pháp phòng thủ).  
- OWASP: *Testing for Host Header Injection* & cấu hình best practices theo framework/proxy.

> Nếu bạn muốn, mình có thể chuyển nội dung thành **checklist cho Dev/DevOps/Pentest** hoặc gợi ý cấu hình cho stack cụ thể (Nginx + Django/Express) theo hướng phòng thủ.
