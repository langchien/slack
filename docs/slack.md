# Bách Khoa Toàn Thư Tính Năng Slack (Feature Reference Bible)

> **Mục đích tài liệu**: Tài liệu này đóng vai trò là **từ điển tham khảo toàn diện** về tất cả chức năng, tiểu tiết hành vi và trải nghiệm người dùng của Slack chính gốc. Dùng làm cơ sở **đối chiếu, tra cứu và đánh giá** độ hoàn thiện của sản phẩm trong quá trình phát triển (không bắt buộc phải clone 100% mọi tính năng).

---

## 1. Hệ Thống Phân Loại & Tiêu Chí Đánh Giá

Mọi tính năng trong tài liệu được phân cấp theo 2 trục:

### 1.1 Trục Tầm ảnh hưởng & Độ nổi bật (Feature Tier)
* **Tier 1 — Core / Iconic (Biểu tượng & Cốt lõi)**: Định hình bản sắc của Slack. Thiếu những tính năng này thì không còn là Slack (Kênh, Luồng thảo luận Threads, Reacji emoji, Thanh tìm kiếm nhanh `Ctrl+K`, Trạng thái hiện diện).
* **Tier 2 — Major / Standard (Tính năng lớn tiêu chuẩn)**: Các khối chức năng chuẩn mực trong giao tiếp hiện đại (Huddles, Canvas, Tìm kiếm nâng cao, Đính kèm file, Cài đặt thông báo chi tiết).
* **Tier 3 — Minor / Productivity (Tiện ích tăng năng suất)**: Tăng tốc độ làm việc (Hẹn giờ gửi tin, Lưu tin nhắn Saved items, Nhóm kênh tùy chỉnh Custom sections, Nhắc hẹn `/remind`).
* **Tier 4 — Micro & Subtle / "Slack Magic" (Tiểu tiết tinh tế)**: Những chi tiết hành vi cực nhỏ nhưng tạo nên cảm giác "mượt mà", thông minh và cuốn hút đặc trưng của Slack (Phím `Up` để sửa tin, vạch đỏ chia tin chưa đọc, âm thanh gõ cửa/chime, link unfurl).
* **Tier 5 — Enterprise & Platform (Mở rộng & Nền tảng)**: Quản trị cấp doanh nghiệp (Slack Connect đa tổ chức, Workflow Builder, Phân quyền khách mời Guest, Audit log).

### 1.2 Trục Mức độ quan trọng (Priority)
* **P0 (Critical)**: Bắt buộc phải có để hệ thống hoạt động được như một ứng dụng chat.
* **P1 (High)**: Rất quan trọng để tạo ra trải nghiệm người dùng chuẩn Slack.
* **P2 (Medium)**: Nâng cao trải nghiệm, giúp ứng dụng chuyên nghiệp và đầy đủ.
* **P3 (Low / Nice-to-have)**: Các tính năng thứ yếu hoặc mang tính mở rộng.

---

## 2. Danh Mục Tính Năng Chi Tiết Theo Phân Hệ

---

### PHÂN HỆ I: WORKSPACE, TỔ CHỨC & DANH TÍNH (IDENTITY & MEMBERSHIP)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 1.1 | **Multi-Workspace Switcher** | Thanh dọc bên cùng tay trái chuyển đổi nhanh giữa các workspace; hiển thị icon, badge số thông báo chưa đọc, phím tắt `Ctrl+[1-9]`. | Tier 1 | P0 |
| 1.2 | **Workspace Profile & Roles** | Hệ thống vai trò: Primary Owner, Owner, Admin, Full Member, Multi-channel Guest, Single-channel Guest. | Tier 1 | P0 |
| 1.3 | **Profile Fields Tùy Biến** | Hiển thị: Avatar, Full name, Display name, Title/Chức danh, Pronouns, Múi giờ địa phương, Số điện thoại, Tên âm thanh cách đọc tên. | Tier 2 | P1 |
| 1.4 | **Múi giờ & Đồng hồ cục bộ** | Hiển thị giờ hiện tại của thành viên theo múi giờ họ đặt khi xem profile hoặc hover vào avatar (tránh làm phiền người khác ngoài giờ). | Tier 3 | P2 |
| 1.5 | **Workspace Join Flow** | Mời qua email, tạo invite link có thời hạn/giới hạn lượt, hoặc tự động join theo đuôi domain email công ty (Google Workspace / Microsoft 365). | Tier 2 | P1 |
| 1.6 | **Slack Connect (Cross-org)** | Kênh chia sẻ giữa 2 hoặc nhiều công ty/workspace khác nhau mà không cần tạo tài khoản ở workspace đối tác. | Tier 5 | P3 |

---

### PHÂN HỆ II: KÊNH & NHÓM HỘI THOẠI (CHANNELS & DMS)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 2.1 | **Public Channel (`#`)** | Kênh công khai, bất kỳ ai trong workspace đều có thể tìm thấy, đọc lịch sử và tham gia. | Tier 1 | P0 |
| 2.2 | **Private Channel (`🔒`)** | Kênh riêng tư, chỉ người được mời mới thấy và truy cập. Không thể chuyển từ private về public. | Tier 1 | P0 |
| 2.3 | **Direct Message (1:1 DM)** | Nhắn tin trực tiếp giữa 2 cá nhân, hiển thị trạng thái online/offline trực tiếp cạnh tên. | Tier 1 | P0 |
| 2.4 | **Group DM (Tối đa 9 người)** | Nhắn tin nhóm riêng lẻ không đặt tên; có thể convert thành Private Channel bất kỳ lúc nào nếu nhóm phát triển lớn. | Tier 2 | P1 |
| 2.5 | **Channel Details Header** | Tiêu đề hiển thị: Tên kênh, nút Star, Topic (chủ đề ngắn có thể click sửa), Member avatars & count, Search trong kênh. | Tier 1 | P0 |
| 2.6 | **Channel Description & Topic** | Phân biệt rõ ràng giữa **Topic** (chủ đề hiện tại của kênh, hiện ở thanh bar) và **Description** (mục đích tồn tại của kênh). | Tier 3 | P1 |
| 2.7 | **Custom Sidebar Sections** | Người dùng tự tạo các nhóm kênh trong thanh sidebar (ví dụ: "Dự án A", "Giải trí", "Quan trọng"), kéo-thả sắp xếp. | Tier 3 | P2 |
| 2.8 | **Starred Channels / DMs** | Đánh dấu sao các kênh/DM quan trọng để ghim lên đầu sidebar vào section "Starred". | Tier 2 | P1 |
| 2.9 | **Mute Channel** | Tắt chuông và không làm đậm tên kênh khi có tin mới; kênh bị mờ đi và dồn xuống đáy danh sách. | Tier 3 | P1 |
| 2.10 | **Archive & Delete Channel** | Lưu trữ kênh (read-only, giữ lịch sử tìm kiếm, có thể khôi phục) hoặc xóa hoàn toàn kênh (chỉ admin). | Tier 2 | P1 |
| 2.11 | **Channel Posting Permissions** | Admin có thể cài đặt kênh chỉ cho phép thông báo (chỉ admin/chủ kênh được đăng bài, thành viên chỉ được reply trong thread). | Tier 3 | P2 |

---

### PHÂN HỆ III: TIN NHẮN & SOẠN THẢO (MESSAGING & RICH TEXT)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 3.1 | **WYSIWYG & Markdown kép** | Hỗ trợ gõ trực tiếp cú pháp markdown (`**bold**`, `_italic_`, `~strike~`, `code`, `> quote`) và thanh toolbar định dạng rich text. | Tier 1 | P0 |
| 3.2 | **Multi-line Code Blocks** | Khối code với cú pháp ```` ``` ````, tự nhận diện thụt đầu dòng, có nút Copy riêng và giữ nguyên định dạng font monospace. | Tier 1 | P1 |
| 3.3 | **Bullet & Numbered Lists** | Tự động thụt lề, hỗ trợ danh sách nhiều cấp (Tab / Shift+Tab để lùi/tiến cấp danh sách). | Tier 2 | P1 |
| 3.4 | **Linkify & Custom Link Text** | Tự động biến URL thành hyperlink; hỗ trợ gán text vào link qua phím tắt `Ctrl+K` trong khung soạn thảo. | Tier 2 | P1 |
| 3.5 | **Blockquotes (`>`)** | Hiển thị trích dẫn với đường viền dọc bên trái và chữ hơi nghiêng. | Tier 2 | P1 |
| 3.6 | **Auto-save Drafts** | Tự động lưu bản nháp tin nhắn đang gõ cho từng kênh/DM; mục "Drafts & Sent" ở thanh sidebar gom lại toàn bộ bản nháp chưa gửi. | Tier 2 | P1 |
| 3.7 | **Scheduled Send (Hẹn giờ gửi)** | Nút mũi tên cạnh nút gửi: chọn thời gian gửi tin trong tương lai (9:00 AM sáng mai, hoặc ngày giờ tuỳ chọn). | Tier 3 | P2 |
| 3.8 | **Edit & Delete Message** | Chỉnh sửa nội dung tin nhắn (gắn nhãn `(edited)` kèm timestamp); xóa tin nhắn (soft delete, kèm xác nhận). | Tier 1 | P0 |
| 3.9 | **Link Unfurl (Rich Preview)** | Tự động lấy OpenGraph metadata của đường link (ảnh, tiêu đề, mô tả tóm tắt) và hiển thị thẻ preview dưới tin nhắn; có nút `x` để gỡ preview. | Tier 2 | P1 |
| 3.10 | **Collapsible Long Messages** | Tin nhắn quá dài (> 700 ký tự hoặc quá nhiều dòng) tự động co lại với nút "Show more" / "Show less". | Tier 4 | P2 |

---

### PHÂN HỆ IV: PHẢN HỒI, LUỒNG & TƯƠNG TÁC (THREADS, REACJI & MENTIONS)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 4.1 | **Thread (Luồng trả lời)** | Nhấp "Reply in thread" mở khung thread trượt ra ở cột bên phải. Giữ cho kênh chính không bị trôi tin nhắn. | Tier 1 | P0 |
| 4.2 | **Broadcast to Channel (`Also send to #channel`)** | Checkbox khi trả lời thread cho phép bắn tin nhắn đó ra luồng chính của kênh để mọi người cùng thấy. | Tier 2 | P1 |
| 4.3 | **Thread Notification & Follow/Unfollow** | Người tham gia thread tự động nhận thông báo; có nút "Unfollow thread" nếu không muốn nhận ping nữa. | Tier 2 | P1 |
| 4.4 | **Centralized "Threads" View** | Mục "Threads" trên đỉnh sidebar gom toàn bộ các luồng hội thoại người dùng đang theo dõi có tin mới. | Tier 1 | P0 |
| 4.5 | **Emoji Reactions (Reacji)** | Thả cảm xúc emoji vào tin nhắn; hover hiển thị danh sách ai đã thả; click vào icon để cộng dồn hoặc bỏ reaction. | Tier 1 | P0 |
| 4.6 | **Hover Action Bar** | Thanh công cụ nổi xuất hiện khi hover lên tin nhắn: 3 emoji hay dùng nhất, nút Add reaction, Reply in thread, Share/Forward, Bookmark, More actions (`...`). | Tier 1 | P0 |
| 4.7 | **Custom Emoji** | Cho phép tải lên emoji riêng của workspace (`:party-parrot:`, logo công ty,...), tổ chức theo category. | Tier 3 | P2 |
| 4.8 | **Emoji Skin Tone Selector** | Cho phép chọn tông màu da mặc định cho các emoji cử chỉ bàn tay/khuôn mặt. | Tier 4 | P3 |
| 4.9 | **Mentions Cụ Thể (`@username`)** | Nhắc tên cá nhân, người được nhắc nhận thông báo nổi bật màu vàng nhạt trên nền tin nhắn. | Tier 1 | P0 |
| 4.10 | **Nhắc Nhóm (`@channel`, `@here`, `@everyone`)** | • `@channel`: Báo mọi thành viên trong kênh.<br>• `@here`: Chỉ báo những ai đang online.<br>• `@everyone`: Chỉ dùng ở kênh `#general`, báo toàn bộ workspace. | Tier 1 | P1 |
| 4.11 | **User Group Mentions (`@team-frontend`)** | Tạo nhóm người dùng để nhắc tên cả nhóm cùng lúc. | Tier 3 | P2 |
| 4.12 | **Pin Messages** | Ghim các tin nhắn quan trọng vào kênh; mở danh sách Pinned Messages ở panel bên phải; ai vào kênh cũng xem được. | Tier 2 | P1 |
| 4.13 | **Bookmarks & Saved Items** | Đánh dấu "Save for later" tin nhắn riêng của cá nhân; mở lại qua tab "Later" trên sidebar. | Tier 2 | P1 |
| 4.14 | **Message Forwarding / Share** | Chia sẻ tin nhắn sang một kênh hoặc DM khác kèm trích dẫn nguyên văn và lời bình luận thêm. | Tier 2 | P1 |

---

### PHÂN HỆ V: HIỆN DIỆN & TRẠNG THÁI (PRESENCE & STATUS)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 5.1 | **Real-time Presence Indicator** | • Chấm xanh tròn đặc: Online.<br>• Vòng tròn xanh rỗng: Away (sau 10-15 phút không tương tác).<br>• Biểu tượng mặt trăng zZz: Do Not Disturb. | Tier 1 | P0 |
| 5.2 | **Manual Presence Setting** | Người dùng có thể tự chuyển trạng thái của mình sang "Set yourself as away" mà không phụ thuộc vào inactivity timer. | Tier 3 | P1 |
| 5.3 | **Custom Status + Emoji** | Trạng thái hiển thị chữ kèm icon emoji (ví dụ: 🌴 *On vacation*, 🍔 *Out to lunch*, 🤒 *Sick*). Hiện cạnh tên ở mọi nơi. | Tier 2 | P1 |
| 5.4 | **Status Auto-Expiry (Hết hạn trạng thái)** | Đặt thời gian tự động xóa trạng thái (Sau 30 phút, 1 giờ, Hôm nay, Tuần này, hoặc chọn giờ chính xác). | Tier 3 | P2 |
| 5.5 | **Do Not Disturb (DND) Schedule** | Cài đặt giờ không làm phiền cố định (ví dụ: từ 22h00 đến 08h00 hàng ngày); tắt mọi chuông báo và đẩy thông báo vào hàng đợi. | Tier 2 | P1 |
| 5.6 | **Urgent Bypass DND** | Khi gửi tin nhắn cho người đang bật DND, người gửi có tuỳ chọn "Notify anyway" trong trường hợp cực kỳ khẩn cấp. | Tier 4 | P2 |

---

### PHÂN HỆ VI: THÔNG BÁO & HUY HIỆU (NOTIFICATIONS & BADGES)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 6.1 | **Unread Indicator (In đậm)** | Kênh có tin nhắn chưa đọc sẽ được in đậm chữ trắng trên sidebar; không làm phiền bằng âm thanh nếu không bị mention. | Tier 1 | P0 |
| 6.2 | **Mention Badge (Số đếm đỏ)** | Huy hiệu số màu đỏ trên sidebar và taskbar khi có tin nhắn trực tiếp DM hoặc được `@mention`. | Tier 1 | P0 |
| 6.3 | **Granular Notification Scope** | Cài đặt nhận thông báo: "All new messages", "Direct messages, mentions & keywords", hoặc "Nothing". Áp dụng toàn cục hoặc override cho từng kênh. | Tier 2 | P1 |
| 6.4 | **Custom Keyword Alerts** | Người dùng nhập danh sách từ khóa quan trọng (ví dụ: tên dự án, "lỗi", "deploy"); khi ai đó gõ từ này thì người dùng nhận thông báo như được mention. | Tier 3 | P2 |
| 6.5 | **Cross-device Smart Notifications** | Khi đang mở Slack trên Desktop, thông báo di động bị hoãn lại 1-2 phút; chỉ gửi push notification tới điện thoại khi người dùng rời máy tính. | Tier 4 | P2 |
| 6.6 | **Desktop Audio Alerts** | Âm thanh đặc trưng (Knock Brush, Chime, Wow, Ding, Drop, Ta-da) có thể tùy chọn trong Preferences. | Tier 3 | P2 |
| 6.7 | **Activity Tab / Mentions & Reactions** | Tab trung tâm tổng hợp tất cả các thông báo nhắc tên bạn, ai đã thả reaction vào tin nhắn của bạn, ai mời vào kênh. | Tier 1 | P0 |

---

### PHÂN HỆ VII: ĐIỀU HƯỚNG & TÌM KIẾM (NAVIGATION & SEARCH)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 7.1 | **Quick Switcher (`Ctrl+K` / `Cmd+K`)** | Cửa sổ tìm kiếm nhanh siêu tốc: gõ tên kênh, người dùng, tài liệu để nhảy ngay đến đó mà không cần dùng chuột. | Tier 1 | P0 |
| 7.2 | **Full-Text Message Search** | Tìm kiếm toàn văn trong tất cả các kênh người dùng có quyền đọc, tô sáng từ khóa tìm thấy. | Tier 1 | P0 |
| 7.3 | **Search Filter Modifiers** | Hỗ trợ cú pháp tìm kiếm chuyên sâu:<br>• `from:@user`<br>• `in:#channel`<br>• `has:link`, `has:star`, `has:reaction`<br>• `before:date`, `after:date`, `during:month` | Tier 2 | P1 |
| 7.4 | **Tab Phân Loại Kết Quả Tìm Kiếm** | Tách kết quả thành các tab: **Messages**, **Files**, **Channels**, **People** để dễ lọc. | Tier 2 | P1 |
| 7.5 | **Lịch sử Điều hướng (History Back/Forward)** | Nút mũi tên Quay lại (`Ctrl+[`) và Tiến tới (`Ctrl+]`) giống trình duyệt web, giúp quay lại kênh/vị trí vừa đọc. | Tier 4 | P1 |

---

### PHÂN HỆ VIII: QUẢN LÝ TỆP TIN & MEDIA (FILE SHARING & PREVIEWS)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 8.1 | **Drag & Drop Upload** | Kéo thả tệp tin trực tiếp vào khung chat hoặc bất kỳ đâu trong cửa sổ Slack để tải lên. | Tier 1 | P0 |
| 8.2 | **Multi-file Upload** | Tải lên nhiều file cùng lúc, gom lại thành một lưới ảnh hoặc danh sách file gọn gàng trong 1 tin nhắn. | Tier 2 | P1 |
| 8.3 | **Image Lightbox & Zoom** | Click vào ảnh mở modal toàn màn hình, xem ở độ phân giải gốc, tải xuống hoặc duyệt qua các ảnh khác trong kênh. | Tier 2 | P1 |
| 8.4 | **PDF / Document Preview** | Xem trước tài liệu PDF trực tiếp trong ứng dụng mà không cần tải về máy. | Tier 3 | P2 |
| 8.5 | **Code Snippet Tool** | Tạo tệp snippet code riêng có syntax highlighting theo ngôn ngữ lập trình, số dòng và cho phép người khác tải về hoặc copy. | Tier 3 | P2 |
| 8.6 | **Files Browser** | Trang quản lý tập trung tất cả các file đã chia sẻ trong kênh hoặc toàn bộ workspace, lọc theo loại file và người gửi. | Tier 3 | P2 |

---

### PHÂN HỆ IX: REAL-TIME AUDIO & VIDEO (HUDDLES & CLIPS)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 9.1 | **Slack Huddle (Phòng thoại nhanh)** | Công tắc góc dưới bên trái của kênh/DM để bật phòng thoại nhanh; âm thanh êm dịu khi có người tham gia/rời phòng. | Tier 2 | P1 |
| 9.2 | **Huddle Screen Sharing & Co-draw** | Chia sẻ màn hình trong Huddle, cho phép người nghe vẽ trực tiếp (annotation) lên màn hình người chia sẻ. | Tier 2 | P2 |
| 9.3 | **Huddle In-call Chat & Emoji** | Khung chat tạm thời bên trong huddle và thả reaction emoji bay lên màn hình. | Tier 3 | P2 |
| 9.4 | **Audio Clips** | Ghi âm tin nhắn thoại ngắn (voice note) với biểu đồ sóng âm (waveform), có thể nghe lại và tua tốc độ (1x, 1.5x, 2x). | Tier 3 | P2 |
| 9.5 | **Video Clips** | Ghi hình camera hoặc màn hình ngắn gửi trực tiếp vào kênh; tự động tạo phụ đề (transcription). | Tier 3 | P3 |

---

### PHÂN HỆ X: TÀI LIỆU & GHI CHÚ NỘI BỘ (CANVAS & LISTS)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 10.1 | **Channel Canvas** | Mỗi kênh có 1 trang tài liệu Canvas gắn kèm ở góc trên bên phải để ghi chú quy trình, checklist, links quan trọng của kênh. | Tier 2 | P2 |
| 10.2 | **Standalone Canvas** | Tạo trang tài liệu cá nhân hoặc cộng tác tương tự Notion, nhúng bảng, checklist và mention thành viên. | Tier 3 | P3 |
| 10.3 | **Slack Lists** | Tính năng quản lý công việc dạng bảng kanban/list đơn giản gắn liền với kênh chat. | Tier 3 | P3 |

---

### PHÂN HỆ XI: TỰ ĐỘNG HÓA & BOT (BOTS, SLASH COMMANDS & WORKFLOWS)

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 11.1 | **Slackbot Help & DM** | Kênh DM riêng với Slackbot để nhận thông báo hệ thống, lời chào mừng và lưu nháp ghi chú cá nhân. | Tier 1 | P1 |
| 11.2 | **Slackbot Custom Auto-responses** | Admin cài đặt từ khóa kích hoạt để Slackbot tự động trả lời (ví dụ: ai hỏi "wifi password" -> Slackbot tự trả lời password). | Tier 3 | P2 |
| 11.3 | **Core Slash Commands** | Bộ lệnh gõ tắt kinh điển:<br>• `/remind [who] [what] [when]` (Ví dụ: `/remind me to check PR at 3pm`)<br>• `/shrug` (Tự gõ `¯\_(ツ)_/¯`)<br>• `/mute` / `/unmute`<br>• `/away` / `/active`<br>• `/collapse` và `/expand` (Đóng/mở toàn bộ ảnh/preview) | Tier 3 | P1 |
| 11.4 | **Workflow Builder** | Giao diện kéo thả không cần code để tạo luồng tự động: Form đăng ký xin nghỉ phép, tin nhắn chào mừng thành viên mới, phân luồng ticket. | Tier 5 | P3 |
| 11.5 | **Interactive Message Components (Block Kit)** | Tin nhắn chứa button, dropdown menu, date picker để người dùng thao tác trực tiếp trên chat. | Tier 2 | P1 |
| 11.6 | **Incoming & Outgoing Webhooks** | Nhận payload HTTP POST từ bên ngoài (GitHub, Sentry, Jira) để bắn tin nhắn format đẹp vào kênh. | Tier 2 | P1 |

---

### PHÂN HỆ XII: TIỂU TIẾT TINH TẾ & TRẢI NGHIỆM ĐẶC TRƯNG ("SLACK MAGIC")

> *Đây là những chi tiết làm nên "cái hồn" của Slack — thứ khiến người dùng cảm thấy Slack tinh tế hơn các phần mềm chat doanh nghiệp thông thường.*

| STT | Tên tiểu tiết | Hành vi chi tiết | Tier |
|---|---|---|:---:|
| 12.1 | **Phím `Up Arrow` để sửa nhanh** | Khi khung soạn thảo đang **trống**, ấn mũi tên lên `↑` sẽ tự động chuyển tin nhắn cuối cùng vừa gửi của bạn sang chế độ Edit. | Tier 4 |
| 12.2 | **Vạch đỏ phân tách tin nhắn chưa đọc** | Vạch đỏ mỏng ngang kênh ghi `New Messages` cố định tại vị trí tin nhắn đầu tiên bạn chưa đọc khi bạn mở kênh. Không bị biến mất ngay lập tức cho đến khi bạn rời kênh hoặc đánh dấu đã đọc. | Tier 4 |
| 12.3 | **Debounced Typing Indicator** | Hiển thị *"Alex is typing..."* mượt mà; nếu 3 người cùng gõ thì hiện *"Alex, Sarah and 1 other are typing..."*; tự biến mất sau 3 giây không gõ. | Tier 4 |
| 12.4 | **Nút "Mark as Unread" (`Alt + Click`)** | Giữ phím `Alt` và click vào bất kỳ tin nhắn nào trong lịch sử để đánh dấu kênh chưa đọc bắt đầu từ đúng tin nhắn đó. | Tier 4 |
| 12.5 | **Gom nhóm tin nhắn cùng người gửi (Message Grouping)** | Các tin nhắn liên tiếp của cùng 1 người trong vòng 5-10 phút sẽ không lặp lại Avatar và Tên, chỉ hiện timestamp mờ khi hover để tiết kiệm diện tích. | Tier 4 |
| 12.6 | **Display Density (Compact vs Clean)** | Tùy chọn giao diện:<br>• **Clean**: Hiện avatar to, khoảng cách thoáng đãng.<br>• **Compact**: Ẩn avatar, hiển thị dạng dòng chat IRC với tên cùng dòng, chứa được nhiều thông tin hơn. | Tier 4 |
| 12.7 | **Custom Theme với mã Hex** | Cho phép đổi theme sidebar sang Dark/Light hoặc tự nhập chuỗi mã màu hex tùy chỉnh để chia sẻ màu theme với đồng nghiệp. | Tier 4 |
| 12.8 | **Emoji Shortcode Autocomplete** | Gõ `:smile` hiển thị popup gợi ý emoji kèm phím mũi tên và `Enter` để chèn nhanh. | Tier 4 |
| 12.9 | **Jump to Latest / Jump to Unread** | Khi đang scroll xem tin nhắn cũ mà có tin mới, xuất hiện thanh thông báo nổi *"More unread messages below ↓"* kèm nút nhảy nhanh xuống đáy. | Tier 4 |
| 12.10 | **Âm thanh gõ cửa & Easter Eggs** | Âm thanh gõ cửa nhẹ nhàng khi người khác vào phòng thoại; hiệu ứng pháo hoa khi giải quyết xong todo list hoặc reaction `:tada:`. | Tier 4 |

---

### PHÂN HỆ XIII: QUẢN TRỊ, BẢO MẬT & COMPLIANCE

| STT | Tên tính năng | Mô tả chi tiết hành vi | Tier | Mức độ |
|---|---|---|:---:|:---:|
| 13.1 | **Member Management & Invitation** | Trang danh sách thành viên: duyệt tài khoản chờ, đổi quyền (Admin, Member, Guest), vô hiệu hóa tài khoản (Deactivate). | Tier 1 | P0 |
| 13.2 | **Channel Management Console** | Admin xem được toàn bộ danh sách kênh (kể cả kênh công khai không tham gia), dung lượng chiếm dụng, số thành viên, và trạng thái archive. | Tier 2 | P1 |
| 13.3 | **Session & Device Management** | Xem danh sách các thiết bị đang đăng nhập, địa chỉ IP và nút "Sign out all other sessions". | Tier 2 | P1 |
| 13.4 | **Message Retention Policy** | Cài đặt chính sách lưu trữ: Giữ tin nhắn vĩnh viễn, hoặc tự động xóa tin nhắn cũ hơn 30/90 ngày. | Tier 5 | P3 |
| 13.5 | **Data Export** | Xuất toàn bộ dữ liệu lịch sử chat và file đính kèm dưới dạng file nén JSON/zip (phục vụ backup hoặc compliance). | Tier 5 | P3 |
| 13.6 | **Audit Logs API** | Ghi nhận chi tiết mọi hoạt động bảo mật (đăng nhập, xóa kênh, đổi quyền, tải file) cho quản trị viên. | Tier 5 | P3 |

---

## 3. Ma Trận Đối Chiếu Khi Triển Khai Bản Clone (Benchmark Matrix)

Khi xây dựng một ứng dụng Slack Clone, bảng dưới đây đóng vai trò là thang đo đối chiếu mức độ tương đồng:

| Cấp độ hoàn thiện | Các tính năng đối chiếu cần đạt |
|---|---|
| **Mức 1: MVP Chat Cơ Bản** | • Đăng ký / Đăng nhập / Chọn Workspace.<br>• Kênh Public/Private & Nhắn tin 1:1 DM.<br>• Soạn tin nhắn realtime (Socket.io) kèm Markdown cơ bản.<br>• Phân biệt tin đã đọc / chưa đọc trên Sidebar. |
| **Mức 2: Slack Trải Nghiệm Chuẩn** | • **Luồng Threads** chuẩn (mở panel bên phải, centralized Threads tab).<br>• **Emoji Reactions (Reacji)** đầy đủ.<br>• **Quick Switcher `Ctrl+K`** nhảy kênh thần tốc.<br>• Tìm kiếm tin nhắn theo từ khóa.<br>• Đính kèm tệp tin và ảnh xem trước. |
| **Mức 3: Đạt Độ "Mượt" Slack Magic** | • Đầy đủ các **tiểu tiết Tier 4**: Phím `Up` sửa tin, vạch đỏ New Messages, gom nhóm avatar, Jump to unread.<br>• Typing indicator debounced thông minh.<br>• Trạng thái hiện diện Online/Away tự động.<br>• Ghim tin nhắn (Pins) và Lưu xem sau (Bookmarks). |
| **Mức 4: Slack Nâng Cao** | • Huddles (thoại trực tiếp WebRTC).<br>• Hẹn giờ gửi tin nhắn (Scheduled Send).<br>• Cài đặt thông báo chi tiết theo kênh và DND schedule.<br>• Slash commands (`/remind`, `/shrug`). |
