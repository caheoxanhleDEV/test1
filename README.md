# phimhayvietsub — Full-stack source

## Chạy local
1. Cài Node.js 20+ và npm.
2. Sao chép `.env.example` thành `.env` và thay `JWT_SECRET` bằng chuỗi ngẫu nhiên dài ít nhất 32 ký tự. Server tự đọc `.env` nhờ dotenv.
3. Chạy `npm install`.
4. Chạy `npm start`, mở `http://localhost:3000`.
5. Kiểm tra `http://localhost:3000/api/health`; nếu hiện JSON có `"ok":true` thì backend đang chạy.

### Sửa lỗi “Không kết nối được máy chủ”
- Không mở `public/index.html` trực tiếp bằng `file://`; hãy chạy `npm start` và truy cập URL server.
- Nếu chỉ host giao diện trên GitHub Pages, API Express/SQLite không chạy trên GitHub Pages. Hãy deploy toàn bộ backend lên dịch vụ hỗ trợ Node.js và database, rồi sửa `public/config.js`: `window.PHVS_API_URL = 'https://TEN-BACKEND-CUA-BAN/api';` (không có dấu `/` cuối).
- Khi frontend và backend khác host, đặt `CORS_ORIGIN` trong biến môi trường backend thành origin frontend của bạn. Không dùng `*` cho môi trường production; chỉ cho phép domain của bạn.
- Frontend hiển thị trạng thái kết nối ở góc dưới màn hình; nếu offline, kiểm tra backend logs và `/api/health`.

## Kiến trúc
- `public/`: frontend HTML/CSS/JS.
- `server.js`: REST API, xác thực JWT, phân quyền, SSE realtime.
- `data/phimhayvietsub.sqlite`: database trung tâm SQLite tự tạo khi chạy; lưu bền trên máy chủ.
- PC/mobile dùng chung API/database khi website được triển khai trên cùng một server. LocalStorage chỉ giữ token đăng nhập, không giữ kho phim.
- SSE phát sự kiện `movies`, `genres`, `authors`, `users`, `comments`; frontend tự tải lại danh sách phim khi nhận sự kiện.
- Backup tự động tạo snapshot định kỳ và lưu tối đa 30 bản. API backup OWNER-only.

## Tạo OWNER đầu tiên
Đăng ký tài khoản từ website, sau đó chạy lệnh SQL trực tiếp trên SQLite để nâng role: `UPDATE users SET role='OWNER', verified=1 WHERE username='TEN_USERNAME';`. Dùng công cụ SQLite trên máy chủ, không mở quyền OWNER cho người dùng công khai.

## API chính
- `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`
- `GET /api/movies`, `POST /api/movies`, `PATCH /api/movies/:id`, `DELETE /api/movies/:id`
- `POST /api/movies/:id/episodes`, `GET /api/genres`, `POST /api/genres`, `GET /api/authors`, `POST /api/authors`
- `GET /api/admin/pending`, `POST /api/admin/movies/:id/review`, `GET /api/admin/users`, `PATCH /api/admin/users/:id`
- `GET /api/admin/backup`, `POST /api/admin/restore` (khôi phục dữ liệu nội dung, không ghi đè mật khẩu/tài khoản), `GET /api/events` (SSE)

## Quan trọng trước khi public production
Đây là nền tảng full-stack có database/API/realtime và các luồng chính, chưa phải triển khai production hoàn chỉnh. Cần HTTPS, cấu hình reverse proxy/SSE, kiểm thử bảo mật, quy trình restore được kiểm duyệt, object storage/CDN cho video và ảnh, OAuth Google nếu cần, và migration được quản lý khi schema thay đổi. SQLite phù hợp triển khai đơn máy chủ; khi cần nhiều server/scale ngang hãy chuyển sang PostgreSQL và Redis/pub-sub. Upload file nhị phân chưa được bật trong bản này; hiện poster/video nhận URL. Không dùng dữ liệu mẫu để suy ra số liệu thật.

## Bản vá bổ sung
- Chuẩn hóa dữ liệu hồ sơ `/api/auth/me` sang camelCase để frontend dùng thống nhất.
- Kiểm tra username/email, mật khẩu tối thiểu 8 ký tự và dữ liệu profile.
- Khôi phục backup nội dung trong transaction; tài khoản và mật khẩu không bị ghi đè.
- Poster URL hiển thị thành ảnh; tập phim phát video nếu có URL hợp lệ.
- Nếu token đăng nhập hết hạn/không hợp lệ, tự xóa token lỗi khỏi trình duyệt.

## Tính năng được gộp thêm từ yêu cầu prototype xem phim
- Giữ nguyên giao diện dark neon hiện tại, bổ sung đánh giá/bình luận/like/lưu phim và lịch sử xem qua API trung tâm.
- Hồ sơ tài khoản: sửa username, email, tên hiển thị, avatar URL; đổi mật khẩu; xem phim đã đăng và số người theo dõi.
- Trợ lý hướng dẫn theo vai trò và các luồng website (nội dung hướng dẫn cục bộ, không phải AI cloud).
- Bổ sung schema và API cho timeline marker theo giây, giao nhiệm vụ lồng tiếng, trạng thái nhiệm vụ, thông báo, support ticket và dự án đạo diễn.
- Thêm API profile, favorites, follow, history, notifications, director projects, assignments và markers.

## Các API mở rộng
- `GET /api/account/profile`, `PATCH /api/account`, `POST /api/account/password`
- `GET /api/movies/:id/social`, `POST /api/movies/:id/like`, `POST /api/movies/:id/favorite`, `POST /api/movies/:id/history`
- `POST /api/users/:id/follow`, `GET /api/notifications`, `PATCH /api/notifications/:id/read`
- `GET/POST /api/episodes/:id/markers`, `GET /api/director/projects`, `POST /api/episodes/:id/assignments`, `PATCH /api/assignments/:id`, `GET /api/my/assignments`, `POST /api/support`

## Phạm vi còn cần triển khai trước production
- Google OAuth, upload file nhị phân/object storage, HLS transcoding/CDN, phụ đề/đa server, email OTP, PWA, kiểm thử E2E và giao diện quản lý đầy đủ cho mọi API chưa hoàn tất trong gói này.
- SQLite và SSE chia sẻ dữ liệu khi mọi thiết bị cùng truy cập một máy chủ; không đồng bộ được nếu chỉ mở HTML cục bộ hoặc triển khai nhiều instance độc lập.
- Đặt OWNER đầu tiên qua console/database an toàn theo hướng dẫn bên trên; không cho phép tự đăng ký OWNER qua form.
