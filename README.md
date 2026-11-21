# Tarot & Tử Vi Agent Office – backend điều phối

Luồng: **Messenger / Zalo → Lễ Tân → (Điều Phối Lịch) → QC → anh duyệt trong văn phòng 3D → gửi khách → Lưu Trữ ghi hồ sơ**.
Nhân vật trong văn phòng 3D đứng dậy, đi lại, nghỉ ngơi theo đúng việc agent đang làm.

## Chạy thử
1. Cài Node 20.6+; chạy `npm install`.
2. Sao `.env.example` thành `.env`, điền khoá (xem bên dưới).
3. `npm start` rồi mở `http://localhost:3000`, bấm **Kết nối server**, dán `ADMIN_TOKEN`.
4. Webhook cần địa chỉ **HTTPS công khai**. Khi thử ở máy: `cloudflared tunnel --url http://localhost:3000` (hoặc ngrok).

## Messenger
- developers.facebook.com → tạo app → thêm sản phẩm Messenger → gắn Page.
- Webhook: `https://<domain>/webhook/messenger`, Verify Token = `FB_VERIFY_TOKEN`, đăng ký trường `messages`.
- `FB_APP_SECRET` (Settings → Basic), `FB_PAGE_TOKEN` (Page access token).
- Khách thật nhắn được khi app ở chế độ Live và đã có quyền `pages_messaging` (cần App Review). Trước đó chỉ tài khoản tester/admin của app nhắn được.
- Chỉ được trả lời trong 24 giờ kể từ tin cuối của khách. Ngoài cửa sổ, server báo ❗ để anh nhắn tay.
- Đối chiếu số phiên bản Graph API (`FB_GRAPH_VERSION`) với tài liệu Meta hiện hành.

## Zalo OA
- developers.zalo.me → tạo app, liên kết với OA, cấp quyền gửi tin và quản lý tin nhắn người dùng. Liên kết app qua API có thể cần gói OA trả phí.
- Xác thực domain: tải file HTML Zalo cấp, đặt vào thư mục `public/`.
- Webhook: `https://<domain>/webhook/zalo`, bật sự kiện `user_send_text`. Lấy **OA Secret Key** ở phần cài đặt webhook (khác App Secret).
- Token OA sống khoảng 25 giờ, refresh token chỉ dùng 1 lần. Dán cặp token đầu tiên vào `.env`; server tự làm mới và lưu lại trong `data/db.json`.
- Chỉ được gửi tin tư vấn trong 48 giờ kể từ tin cuối của khách.
- Công thức chữ ký (`sha256(appId + body + timestamp + OASecretKey)`) lấy từ cộng đồng Zalo, không phải trang spec chính thức. Hãy gửi một tin thật và xem server có trả 401 hay không.

## An toàn
- Mặc định `APPROVAL_MODE=all`: không tin nào tới khách nếu anh chưa bấm duyệt. Tin bị gắn cờ luôn chờ duyệt.
- Chữ ký webhook được kiểm tra trước mọi xử lý; `ADMIN_TOKEN` bảo vệ giao diện duyệt.
- Nội dung khách được coi là dữ liệu, luật cứng (sinh tử, cờ bạc, hứa chắc chắn) chạy bằng regex ngoài model.
- `data/db.json` chứa tin nhắn khách: giữ ngoài git, quyền 600. Xoá dữ liệu một khách: `DELETE /api/customers/:id`.
- Bài đăng Sáng Tạo (`POST /api/content`) không dùng dữ liệu khách. Xin phép khách trước khi dùng bất kỳ câu chuyện thật nào.

## Còn thiếu (làm tiếp)
- Chốt lịch tự động: hiện anh xác nhận bằng `POST /api/bookings` sau khi khách đồng ý. Nối Google Calendar thay `freeSlots()`.
- Ảnh/voice: hiện chỉ xử lý tin nhắn chữ.
- Đổi file JSON sang SQLite/Postgres khi khách đông.
