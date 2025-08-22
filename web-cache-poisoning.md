# Web Cache Poisoning — Exploiting Cache *Design* Flaws (PortSwigger, concise)

> Nguồn chính: PortSwigger Web Security Academy — “Exploiting cache design flaws”. Bản tóm tắt súc tích để ôn tập và làm lab.

## Mấu chốt
- **Điều kiện dễ vỡ**: Ứng dụng **xử lý unkeyed input** (thường là header/cookie) để tạo **response cacheable** → cache trở thành kênh phát tán tấn công.
- **Đòn bẩy quen thuộc**: `X-Forwarded-Host`, `X-Forwarded-Proto/Scheme`, cookie ngôn ngữ, các URL import JS/JSON động, redirect xây từ header.
- **Mục tiêu**: ép cache lưu phản hồi „độc” (XSS, redirect, nạp script/JSON của attacker), sau đó mọi nạn nhân cùng **cache key** đều nhận bản độc.

## Các mô-típ khai thác chính
### 1 Dùng cache poisoning để phát tán **XSS**
- Unkeyed header bị **phản chiếu vào HTML** (meta/canonical/OG) mà **không lọc** → chỉ cần chèn payload vào header rồi **để nó bị cache**.
- Ví dụ: `X-Forwarded-Host` xuất hiện trong thẻ `<meta property="og:image" …>` → có thể bẻ HTML và đưa `<script>`.

### 2 **Import tài nguyên** không an toàn (JS/JSON/CSS)
- Ứng dụng **dựng URL import** từ unkeyed input (thường `X-Forwarded-Host`) → attacker thay domain sang máy của mình.
- Nếu phản hồi chứa URL độc được **cache**, trình duyệt nạn nhân sẽ **tải & chạy** file của attacker.

### 3 **Cookie-handling** không an toàn
- Ứng dụng quyết định **nội dung** (vd. bản dịch) dựa trên **cookie**, nhưng cache **không đưa Cookie vào key** → một người vô tình/ác ý có thể „đổi” nội dung cho toàn bộ người dùng. (Hiếm gặp hơn header-based, nhưng có).

### 4 **Ghép nhiều header** để mở đường khai thác
- Một header đơn lẻ **chưa khai thác được**; nhưng ghép lại (vd. `X-Forwarded-Scheme` kích hoạt **redirect sang HTTPS**, còn `X-Forwarded-Host` chèn **host độc** vào **Location**) → tạo **response cacheable** trỏ tới tài nguyên độc (điển hình các lab “multiple headers”).

### 5 **Phản hồi lộ thông tin** giúp canh thời điểm/nhắm mục tiêu
- `Age`, `Cache-Control: max-age=…`, thông tin purge → bật mí **nhịp cache** để gửi đúng lúc, không cần spam.
- `Vary` tiết lộ **header nào được đưa vào cache key** (vd. `User-Agent`) → tấn công **có chủ đích** theo từng phân khúc UA/ngôn ngữ.

### 6 **DOM-based** qua dữ liệu bị đầu độc
- Không chỉ file JS: app thường **fetch JSON** rồi xử lý trên client. Nếu code DOM xử lý **không an toàn**, chỉ cần đầu độc cache để nạn nhân **nạp JSON độc** (có thể cần **CORS: ACAO:***).

### 7 **Chaining** nhiều kỹ thuật
- Thực tế thường phải **xâu chuỗi**: multi-header + import không an toàn + DOM sink + thông tin TTL/Vary → tạo đường khai thác đầy đủ.

## Tín hiệu & đòn bẩy quan trọng
- **Dấu hiệu cache**: `X-Cache: hit/miss`, `Age`, `Cache-Control`, `Via`, `Vary`.
- **Unkeyed inputs hay gặp**: `X-Forwarded-Host`, `X-Forwarded-Proto/Scheme`, một số cookie cấu hình, query „phụ” (utm).
- **Khóa kiến thức**: unkeyed input **ảnh hưởng nội dung**, nhưng **không** nằm trong cache key → điều kiện „poisonable”.

## Checklist kiểm thử siêu nhanh (tập trung *design flaws*)
1. **Chọn endpoint cacheable** (trang HTML phổ biến, file JS/JSON quan trọng). Kiểm `Age`/`X-Cache`/`Cache-Control`.
2. **So sánh baseline** vs. yêu cầu có `X-Forwarded-Host` / `X-Forwarded-Scheme`:
   - Quan sát **Location** (redirect), **URL tuyệt đối** trong HTML, **URL import** JS/JSON.
3. Nếu đơn lẻ chưa „ăn”, **kết hợp nhiều header** (điển hình: Scheme + Host) để tạo **response cacheable** chứa URL/HTML do mình kiểm soát.
4. **Prime & poison**: dùng **cache-buster** để học hành vi; khi chắc key, **bỏ cache-buster** và gửi yêu cầu chứa giá trị độc để **ghi vào cache**.
5. **Nếu có `Vary`**, bắt chước phân khúc (UA/Language) của nạn nhân/công chúng mục tiêu.
6. **Xác nhận**: yêu cầu lặp lại thấy `X-Cache: hit`, nạn nhân tải tài nguyên độc (hoặc HTML đã bị chèn).

## Gợi ý phòng ngừa (từ nội dung chương này)
- Đừng dùng **dữ liệu không tin cậy** (nhất là unkeyed header) để dựng **URL/HTML** trong response cacheable.
- Chỉ tin các `X-Forwarded-*`/`Forwarded` do **proxy tin cậy** set; ở proxy hãy **xóa** giá trị client gửi rồi **set lại**.
- Hạn chế lộ `Age`/TTL; dùng `Vary` có chủ đích; khi nghi ngờ, **không cache** hoặc **đưa đủ trường vào key**.
