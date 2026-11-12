# Design Inspiration Sources — kho tham chiếu section/pattern THẬT

> Dùng khi Design Agent **không có ảnh/Figma reference** (Phase 1 — Option 3 "agent tự design").
> Mở các nguồn dưới để lấy pattern THẬT từ website đang chạy, **trước khi** tự generate design system.
> Mục tiêu: tránh layout "AI slop" generic — tham chiếu section thật rồi mới chọn/biến tấu.

## ⚠️ Quy tắc dùng
- Đây là **nguồn tham khảo bố cục/pattern**, KHÔNG phải copy nguyên si.
- Ưu tiên vẫn là: `taste-skill-v2.md` §4 Anti-Slop > ui-ux-pro-max defaults > nguồn inspo ở đây.
- Lấy pattern (cấu trúc hero, cách xếp CTA, kiểu footer...) → map vào `design-tokens.md` của project (màu/type/spacing của mình), KHÔNG bê màu/brand của site mẫu.
- Không cần API key, không login. Chỉ xem để tham chiếu.

## Nguồn theo nhu cầu

### 1. Section tổng hợp (ưu tiên mở đầu tiên)
- **unsection.com** — 5.400+ section thật chia theo loại: Hero · Testimonials · Feature · CTA · Footer · Navbar · Pricing · FAQ · Team.
  - Dùng khi: cần dựng cả một phần trang (hero, bảng giá, FAQ...). Mở đúng tab loại section đang thiếu.

### 2. Component chuyên sâu (khi cần 1 phần cụ thể)
- **navbar.gallery** — navbar thật chia theo kiểu: Static · Dropdown · Mega Menu · Side Bar · Search Bar · Announcement · Full Screen · Breadcrumbs.
  - Dùng khi: cần thanh menu đầu trang gọn/rõ/dễ bấm.
- **cta.gallery** — nút & khối CTA tăng chuyển đổi: Button · Call-to-Buy · Download · Form · Modal/Pop-up · Newsletter · Pricing/Subscription.
  - Dùng khi: cần nút đăng ký/mua/tải khiến người xem muốn bấm.
- **footer.design** — footer lọc theo phong cách: Typographic · Small Type · Illustrative · Grid · Flat · Animated · Cards · Bright · Dark.
  - Dùng khi: cần chân trang có link/liên hệ/MXH trông hoàn thiện.

### 3. Layout đặc thù
- **bentogrids.com** — bố cục lưới Bento (chia ô to/nhỏ) để khoe nhiều tính năng trên 1 màn hình. Lọc: Graphic Design · Web Design · Animation · nền tối/sáng.
  - Dùng khi: feature showcase / landing cần trình bày nhiều tính năng mà vẫn gọn.

### 4. Nice-to-have (tuỳ project)
- **404s.design** — trang lỗi 404 sáng tạo. Chỉ mở khi project có trang lỗi và muốn giữ chân khách + đúng màu thương hiệu.

## Motion (chỉ khi MOTION_INTENSITY cao — đối chiếu taste-skill §5 trước)
- **60fps.design** — kho micro-interaction quay từ app mobile thật (button, bottom sheet, badge, blur, bounce...). Hợp **mobile/react-native** hơn web.
  - Dùng khi: muốn nút/menu/thẻ có chuyển động mượt như app xịn. KHÔNG lạm dụng — tuân §5 Animation Protocol (forbidden patterns, reduced-motion).

## KHÔNG wire vào flow (chỉ inspo rời, tự chịu trách nhiệm)
- **motionsites.ai** — kho prompt copy-paste vào Claude để gen landing page có motion.
  - ⚠️ Cái này **chồng vai trò** với chính Design Agent + loop code. Dán prompt ngoài vào dễ phá taste-skill anti-slop đã dựng. Chỉ xem làm cảm hứng; KHÔNG dùng prompt của nó thay cho design-spec/tokens của project.
