# Ai là triệu phú Chibi

Website trò chơi giáo dục Lịch sử & Địa lý cho trẻ 10–13 tuổi.
HTML/CSS/JS thuần, không cần build. Đăng nhập bằng email + mật khẩu qua Supabase Auth (cùng cách làm với app 0.1%).

## Cấu trúc thư mục

```
index.html            Toàn bộ giao diện + logic game
config.js             ← CHỈ CẦN SỬA FILE NÀY: URL và anon key của Supabase
data/                 11 ngân hàng câu hỏi JSON (01_… đến 11_…)
supabase/schema.sql   Bảng profiles + chính sách bảo mật (RLS), chạy 1 lần
README.md
```

---

## Cài đặt đăng nhập email & lấy lại mật khẩu (khoảng 15 phút)

### Bước 1 — Tạo project Supabase
1. Vào https://supabase.com → **New project** (nên tạo project **riêng**, không dùng chung project của app 0.1%,
   để tài khoản hai app không lẫn vào nhau). Region: **Southeast Asia (Singapore)**.
2. **SQL Editor → New query**, dán toàn bộ `supabase/schema.sql` → **Run**.

### Bước 2 — Điền `config.js`
**Project Settings → API**, chép 2 giá trị vào `config.js`:
```js
window.CHIBI_CONFIG = {
  supabaseUrl: "https://abcdxyz.supabase.co",   // URL ĐẦY ĐỦ, không chỉ dán mã project
  supabaseAnonKey: "eyJhbGciOi..."              // khóa "anon public"
};
```
Khóa anon được phép công khai; dữ liệu đã được RLS bảo vệ (mỗi tài khoản chỉ đọc/ghi hồ sơ của chính mình).
**Không bao giờ** dán khóa `service_role` vào đây.

### Bước 3 — Đưa lên Vercel
- Tạo repo mới trên GitHub → **Add file → Upload files** → kéo các tệp *bên trong* thư mục
  (index.html, config.js, data/, supabase/, README.md) vào → Commit.
- https://vercel.com/new → Import repo đó → Framework Preset: **Other**, không cần Build Command → Deploy.
- Ghi lại địa chỉ web, ví dụ `https://chibi-quiz.vercel.app`.

### Bước 4 — Khai báo địa chỉ web cho Supabase (bắt buộc để liên kết trong email hoạt động)
**Authentication → URL Configuration**
- **Site URL**: `https://chibi-quiz.vercel.app`
- **Redirect URLs**: thêm `https://chibi-quiz.vercel.app/**` (và `http://localhost:8000/**` nếu chạy thử trên máy)

### Bước 5 — Việt hóa email (tùy chọn nhưng nên làm)
**Authentication → Email Templates**

*Confirm signup* — Tiêu đề: `Xác nhận tài khoản Ai là triệu phú Chibi`
```html
<h2>Chào mừng nhà thám hiểm mới!</h2>
<p>Bố mẹ vừa tạo tài khoản Ai là triệu phú Chibi cho bé. Bấm vào nút dưới đây để kích hoạt:</p>
<p><a href="{{ .ConfirmationURL }}">Kích hoạt tài khoản</a></p>
<p>Nếu bạn không tạo tài khoản này, hãy bỏ qua email.</p>
```

*Reset Password* — Tiêu đề: `Đặt lại mật khẩu Ai là triệu phú Chibi`
```html
<h2>Đặt lại mật khẩu</h2>
<p>Có người vừa yêu cầu đặt lại mật khẩu cho tài khoản này. Bấm vào nút dưới đây để tạo mật khẩu mới:</p>
<p><a href="{{ .ConfirmationURL }}">Đặt mật khẩu mới</a></p>
<p>Liên kết chỉ dùng được một lần và sẽ hết hạn sau 1 giờ. Nếu không phải bạn yêu cầu, hãy bỏ qua email.</p>
```

### Bước 6 — Email gửi số lượng lớn (khi có nhiều người dùng)
Máy chủ email mặc định của Supabase chỉ gửi được rất ít email mỗi giờ, đủ để thử nghiệm.
Khi dùng thật, vào **Project Settings → Authentication → SMTP Settings** và nối với một dịch vụ gửi email
(ví dụ Resend, Brevo, SendGrid, hoặc Gmail SMTP với mật khẩu ứng dụng).

---

## Luồng tài khoản (khi đã điền config.js)

- **Tạo tài khoản**: biệt danh của bé, **email của bố mẹ**, mật khẩu (≥ 6 ký tự, nhập 2 lần),
  ô “Tôi là phụ huynh và đồng ý”. Supabase gửi email xác nhận → bấm liên kết → đăng nhập.
  Nhiều con dùng chung một Gmail: dùng `bome+be1@gmail.com`, `bome+be2@gmail.com`… thư vẫn về một hộp thư.
- **Đăng nhập**: email + mật khẩu. Phiên đăng nhập được ghi nhớ, tự gia hạn.
- **Quên mật khẩu?** → nhập email đã đăng ký → nhận email → bấm liên kết → trang “Đặt mật khẩu mới”.
  Thông báo luôn giống nhau dù email có đăng ký hay không, để không lộ ai có tài khoản.
- **Đổi mật khẩu** khi đang đăng nhập: Hồ sơ → “Đổi mật khẩu”.
- **Tiến trình đồng bộ** (avatar, XP, sao, huy hiệu, lịch sử) vào bảng `profiles`, đăng nhập máy nào cũng thấy.
  Lần đầu đăng nhập, tiến trình chơi khách trên máy được chuyển vào tài khoản.
- Vẫn có **“Chơi thử không cần tài khoản”**.

Khi `config.js` để trống (ví dụ bản xem trước trên Claude), website tự chuyển sang **chế độ tài khoản trên thiết bị**:
Góc phụ huynh có mã PIN, bố mẹ tạo tên đăng nhập + mật khẩu cho con, đặt lại mật khẩu thủ công; không gửi email,
không đồng bộ giữa các máy.

Xóa hẳn một tài khoản: Supabase → **Authentication → Users** → chọn → Delete (hồ sơ tự xóa theo, nhờ `on delete cascade`).

---

## Chạy thử trên máy

```bash
cd chibi
python3 -m http.server 8000
# mở http://localhost:8000
```

## Thêm hoặc thay ngân hàng câu hỏi

1. Đặt tệp vào `data/` đúng tên như cấu hình, ví dụ `data/05_The_gioi_co_dai.json`.
2. Danh sách chủ đề là hằng `TOPICS` trong `index.html`; thêm chủ đề = thêm một dòng `{file, name, place, icon, bg}`.
3. Tệp thiếu hoặc rỗng tự hiện “Sắp mở khóa”.

Mỗi bản ghi: `id, chu_de, linh_vuc, muc_do, cau_hoi, A, B, C, D, dap_an, giai_thich`.
Khi nạp, game kiểm tra thiếu trường, `dap_an` ngoài A–D, phương án trùng, `muc_do` sai, ID trùng; bản ghi lỗi bị bỏ qua.

## Luật chơi

**Ai là triệu phú Chibi** — 15 câu, câu *i* lấy mức độ *i* (dữ liệu có mức 1–15). Chọn → “Chốt đáp án” → hồi hộp → công bố
→ “Góc khám phá”. Sai là kết thúc; mốc an toàn câu 5 (2.000.000) và câu 10 (22.000.000); được dừng nhận thưởng giả lập.
Trợ giúp (1 lần/lượt): 50:50, Khán giả nhí, Hỏi cú thông thái, Đổi câu.

**Chơi tự do** — 10/20/30/50/toàn bộ câu, khoảng mức độ, đồng hồ tùy chọn. Đúng = 1 điểm, sai vẫn chơi tiếp;
cuối lượt ôn câu sai.

Không có bảng xếp hạng ở bất kỳ đâu.

## Chưa có (cần thêm máy chủ thời gian thực)

Kết bạn bằng mã, đấu 1 vs 1, phòng nhóm 2–6 người, thử thách bất đồng bộ, chấm điểm phía server.
Màn hình “Bạn bè” ghi rõ **chưa khả dụng**. Hướng làm tiếp: Supabase Realtime + Edge Functions trên cùng project.
