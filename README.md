# Rentzy — Tổng quan mã nguồn và website

## Giới thiệu
- Rentzy là nền tảng C2C/B2C cho thuê xe, kết nối Chủ xe (Owner) và Khách thuê (Renter).
- Hệ thống hỗ trợ đầy đủ quy trình: tìm kiếm xe, đặt xe, thanh toán cọc, ký hợp đồng, bàn giao/nhận xe, trả xe, đánh giá; đồng thời xử lý nghiệp vụ phát sinh như phạt nguội, bồi thường và rút tiền.

## Kiến trúc
- Monorepo gồm hai phần chính: `frontend` (React + Vite + Tailwind) và `backend` (Node.js + Express + Sequelize).
- Backend kết nối cơ sở dữ liệu MySQL qua Sequelize, quản lý mô hình `User`, `Vehicle`, `Booking`, `Transaction`, `Notification`, `TrafficFineRequest`...
- Tích hợp dịch vụ ngoài:
  - Thanh toán: PayOS (`@payos/node`)
  - Email: Nodemailer
  - Lưu trữ ảnh: Cloudinary hoặc AWS S3 (tùy cấu hình)
  - OCR / AI hỗ trợ: `tesseract.js`, OpenAI (mô tả xe, trợ lý)
- Realtime/WebSocket cho thông báo: `ws` (xem `services/wsService.js` nếu có).

## Công nghệ chính
- Frontend: React 19, Vite 7, Tailwind 4, Radix UI, TanStack Query, React Router, Chart.js, Recharts.
- Backend: Express 5, Sequelize 6, MySQL2, JWT, Multer, Node-Cron, Nodemailer.
- Khác: Date-fns, Axios, Lucide-React, Leaflet/React-Leaflet.

## Cấu trúc thư mục
- `frontend/` mã nguồn giao diện khách hàng và admin
- `backend/` dịch vụ API, logic nghiệp vụ, tích hợp thanh toán/email/ws
- Một số tệp UML minh họa luồng nghiệp vụ: `*.puml` (PlantUML)

## Luồng nghiệp vụ chính
- Thuê xe (Booking):
  - Tìm kiếm & đặt xe → Duyệt yêu cầu → Thanh toán cọc → Ký hợp đồng → Bàn giao xe → Sử dụng → Trả xe → Hoàn tất thanh toán.
  - Lấy lịch đã đặt để khóa ngày: `backend/src/controllers/booking/bookingController.js:16`
  - Chi tiết đơn thuê (bao gồm thông tin xe, hợp đồng, giao dịch): `backend/src/controllers/booking/bookingController.js:96`
- Phạt nguội (Traffic Fine):
  - Owner tạo yêu cầu phạt nguội, đính kèm bằng chứng → Admin duyệt → Renter thanh toán → Admin chuyển tiền cho Owner.
  - Lấy danh sách yêu cầu phạt nguội (admin): `backend/src/controllers/admin/adminTrafficFineController.js:15`
  - Thống kê yêu cầu phạt nguội: `backend/src/controllers/admin/adminTrafficFineController.js:179`
  - Danh sách các khoản chuyển tiền phạt nguội (payouts): `backend/src/controllers/admin/adminTrafficFineController.js:205`
- Thanh toán:
  - Tích hợp PayOS tạo liên kết thanh toán, xử lý giao dịch và bồi thường (COMPENSATION).
  - Kiểm tra phiên bản PayOS: `backend/package.json:5`
- Thông báo:
  - Tạo thông báo đến user khi duyệt, từ chối, cập nhật trạng thái; email gửi qua Nodemailer.

## Thiết lập môi trường
- Yêu cầu: Node.js >= 18, MySQL 8.x (hoặc tương đương), pnpm/npm.
- Biến môi trường (ví dụ):
  - `DATABASE_URL` hoặc chuỗi kết nối MySQL (host, user, password, database)
  - `JWT_SECRET`
  - `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` (gửi email)
  - `PAYOS_CLIENT_ID`, `PAYOS_API_KEY`, `PAYOS_CHECKSUM_KEY` (nếu dùng PayOS)
  - `CLOUDINARY_URL` hoặc thông tin AWS S3 (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_S3_BUCKET`, `AWS_REGION`)
- Không commit thông tin nhạy cảm vào repo.

## Cài đặt & chạy
- Backend:
  - Cài đặt: `cd backend && npm install`
  - Chạy dev: `npm run dev` (khởi chạy `src/server.js`) `backend/package.json:40`
  - Migration bổ sung trường phạt nguội (nếu có): `npm run migrate:traffic-fine` `backend/package.json:41`
- Frontend:
  - Cài đặt: `cd frontend && npm install`
  - Chạy dev: `npm run dev`
  - Build: `npm run build`
  - Preview: `npm run preview`

## Testing & Chất lượng mã
- Frontend: `npm run lint` trong `frontend` để kiểm tra ESLint.
- Backend: Tùy dự án, có thể thêm `jest` hoặc `vitest` nếu cần; hiện chủ yếu là kiểm tra qua API, logs, và cron.

## Ghi chú triển khai
- Cấu hình Reverse Proxy (Nginx) hoặc dịch vụ PaaS (Render/Heroku/Vercel/Netlify) tùy phần frontend/backend.
- HTTPS, CORS, bảo mật JWT và rate-limit cần cấu hình theo môi trường production.

## Liên kết mã tiêu biểu
- Lấy lịch xe đã đặt: `backend/src/controllers/booking/bookingController.js:16`
- Lấy booking theo ID (bao gồm vehicle, renter, transaction, handover, contract): `backend/src/controllers/booking/bookingController.js:96`
- Lấy yêu cầu phạt nguội (admin): `backend/src/controllers/admin/adminTrafficFineController.js:15`
- Lấy thống kê phạt nguội: `backend/src/controllers/admin/adminTrafficFineController.js:179`
- Lấy danh sách payout phạt nguội: `backend/src/controllers/admin/adminTrafficFineController.js:205`

## Đóng góp
- Quy ước code theo chuẩn ESLint/Prettier của frontend; backend theo phong cách Express/Sequelize.
- Pull Request kèm mô tả thay đổi, ảnh chụp/gif UI nếu có; tránh push secrets.

## Liên hệ
- Vấn đề khẩn cấp liên quan đến bảo mật hoặc dữ liệu: vui lòng tạo issue nội bộ và liên hệ quản trị hệ thống.

