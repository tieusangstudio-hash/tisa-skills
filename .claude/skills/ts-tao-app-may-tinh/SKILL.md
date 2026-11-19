---
name: ts-tao-app-may-tinh
description: "Gọi bằng /ts-tao-app-may-tinh. Biến ý tưởng tiếng Việt thành app cài đặt thật cho MacBook (.dmg) và Windows (.msi/.exe): giao diện đẹp tiếng Việt, sáng/tối, thanh menu, cài đặt, tự cập nhật, bộ cài chuyên nghiệp. Dựng trên Tauri 2 (113k sao GitHub) + mẫu dannysmith/tauri-template (React, shadcn/ui, Tailwind). Từ khóa: tạo app máy tính, app macbook, app windows, phần mềm cài đặt, file dmg, file exe, desktop app, đóng gói app."
---

# /ts-tao-app-may-tinh — Ý tưởng → app Mac + Windows có bộ cài

Người dùng chỉ đưa ý tưởng. AI làm toàn bộ: đặc tả, giao diện, mã, icon, bộ cài, hướng dẫn.
Khác `/ts-tao-app` (web app 1 file HTML): skill này ra **phần mềm cài vào máy** như app thương mại.

## Bộ khung đã cài sẵn (đừng tải lại)

| Thứ | Chỗ | Ghi chú |
|---|---|---|
| Mẫu gốc | `~/tisa-app-factory/tauri-template/` | **Không sửa mẫu gốc.** Mỗi app mới copy ra thư mục riêng |
| Rust | `~/.cargo` (rustup) | Mỗi lệnh shell: `source ~/.cargo/env` trước |
| Node/npm | đã có | Cài thư viện bằng `npm ci` |
| Tài liệu mẫu | `docs/developer/*.md` trong mẫu | Đọc `architecture-guide.md`, `ui-patterns.md`, `i18n-patterns.md`, `releases.md` trước khi code |

Mẫu đã có sẵn: bảng lệnh Cmd+K, menu hệ thống, cửa sổ Cài đặt lưu bằng Rust, sáng/tối theo máy,
thanh bên co giãn, thông báo, tự cập nhật qua GitHub Releases, nhật ký lỗi, chống mở 2 bản, nhớ vị trí cửa sổ.

## Quy trình

### Bước 1 — Hỏi tối đa 6 câu (A/B/C cho dễ chọn)
1. Tên app (tiếng Việt) + 1 câu app làm gì.
2. Ai dùng: (A) mình tôi · (B) nhân viên · (C) học viên/khách (phát hành rộng).
3. Màn hình chính trông thế nào: (A) bảng danh sách · (B) bảng điều khiển số liệu · (C) trình soạn/xử lý file · (D) nằm trên thanh menu.
4. Dữ liệu lưu: (A) trên máy (mặc định — SQLite/file JSON trong thư mục app) · (B) Google Sheets/mạng.
5. Cần AI / kết nối dịch vụ ngoài không? (khóa API để phía Rust hoặc file `.env`, **không** nhúng vào giao diện).
6. Màu/logo thương hiệu (mặc định: TISA — cam/trắng, logo trong `raw/assets/` nếu có).
Thiếu câu nào → chọn cách đơn giản nhất, ghi là giả định.

### Bước 2 — Đặc tả 1 trang → chờ người dùng gõ "duyệt"
Mục tiêu · màn hình · tính năng Bắt buộc / Nên có / Để sau · dữ liệu · điều app KHÔNG làm · tiêu chí nghiệm thu.

### Bước 3 — Tạo dự án từ mẫu
```bash
APP=ten-app-khong-dau
rsync -a --exclude node_modules --exclude .git --exclude src-tauri/target \
  ~/tisa-app-factory/tauri-template/ ~/tisa-app-factory/apps/$APP/
cd ~/tisa-app-factory/apps/$APP && git init -q && npm ci
```
Rồi làm đúng các bước của `.claude/skills/init/SKILL.md` trong dự án (đổi tên app, `identifier` dạng
`com.hocvientisa.<ten-app>`, publisher "Học Viện TISA", copyright năm hiện tại, mô tả).

### Bước 4 — Tiếng Việt là mặc định
- Tạo `locales/vi.json` dịch đủ mọi khóa của `locales/en.json` (giọng tự nhiên, ngắn: "Cài đặt", "Giao diện", "Thoát {{appName}}").
- `src/i18n/config.ts`: thêm `vi`, đặt `lng: 'vi'`, `fallbackLng: 'vi'`; có thể bỏ `ar`, `fr`.
- Font hỗ trợ dấu tiếng Việt (Be Vietnam Pro hoặc Inter, nhúng file font vào `src/assets`, không tải từ mạng).
- Mọi chữ hiển thị đi qua `t('...')`, không viết cứng.

### Bước 5 — Giao diện ấn tượng
- Dùng component shadcn/ui có sẵn (`src/components/ui`); thêm bằng `npx shadcn@latest add <tên>`.
- Đọc skill `frontend-design` / `ui-ux-pro-max` để chọn hướng thẩm mỹ; màu đặt trong `src/theme-variables.css` cho cả sáng và tối.
- Trạng thái rỗng có hướng dẫn, lỗi nói bằng tiếng Việt dễ hiểu, nút chính to rõ, phím tắt hiện trong menu.
- Icon: tạo 1 ảnh vuông 1024×1024 PNG rồi `npx tauri icon duong-dan.png` (tự ra .icns, .ico và mọi cỡ).

### Bước 6 — Xây theo lát, chạy thử từng lát
- `npm run tauri:dev` để xem app thật; chụp màn hình gửi người dùng sau mỗi lát.
- Logic nặng/đụng file hệ thống → lệnh Rust trong `src-tauri/src/commands/` (xem `tauri-commands.md`), giao diện gọi qua bindings sinh bằng tauri-specta.
- Trước khi giao: `npm run typecheck && npm run lint && npm run test:run` (và `npm run check:all` nếu đã duyệt cài ast-grep).

### Bước 7 — Đóng gói bộ cài
**Mac (làm ngay trên máy này):**
```bash
source ~/.cargo/env && npx tauri build --bundles dmg
# ra: src-tauri/target/release/bundle/dmg/<Tên>_<ver>_aarch64.dmg
```
Muốn chạy cả Mac Intel: `rustup target add x86_64-apple-darwin` rồi `npx tauri build --target universal-apple-darwin --bundles dmg`.

**Windows (.msi / .exe):** máy Mac không dựng bản Windows ổn định → dùng GitHub Actions có sẵn trong mẫu
(`.github/workflows/release.yml`, chạy `tauri-apps/tauri-action` chính chủ). Đẩy code lên repo GitHub **riêng tư**,
tạo tag `v1.0.0` → Actions tự dựng `.dmg` + `.msi` (thêm `nsis` vào `--bundles` nếu muốn file `.exe`) và đặt vào Releases nháp.
**Cần người dùng duyệt** trước khi tạo repo, đẩy code, hay phát hành.

**Tự cập nhật:** `npx tauri signer generate -w ~/.tauri/<app>.key` → dán khóa công khai vào `plugins.updater.pubkey`,
khóa riêng vào GitHub Secrets `TAURI_PRIVATE_KEY`. Khóa riêng **không bao giờ** ghi vào repo/wiki/output.

### Bước 8 — Ký số (báo rõ cho người dùng)
- Chưa ký: Mac báo "không xác định nhà phát triển" (chuột phải → Mở), Windows SmartScreen báo "Windows protected your PC" (More info → Run anyway). Ghi cách mở vào `Hướng dẫn cài đặt.txt`.
- Phát hành rộng cho học viên/khách → nên có Apple Developer (99 USD/năm, ký + notarize) và chứng chỉ ký mã Windows (hoặc Azure Trusted Signing). Đây là việc tốn tiền → chỉ đề xuất, chờ chủ quyết.

### Bước 9 — Giao hàng & lưu bộ não
Thư mục `output/YYYY-MM-DD-<ten-app>/` gồm: file `.md` (câu hỏi gốc, đặc tả, tính năng, cách build lại, dòng `🔗 Kết nối:`),
bộ cài `.dmg` (và `.msi` nếu đã dựng), ảnh giao diện, `Hướng dẫn cài đặt.txt`. Mã nguồn để ở `~/tisa-app-factory/apps/<ten-app>/`.
Cập nhật `index.md`, bản đồ chủ đề phù hợp và `log.md` theo `CLAUDE.md`.

## Khi nào chọn công nghệ khác
- App chỉ cho Mac, cần cảm giác Apple thuần (thanh menu, widget, đụng phần cứng như TISA Cool) → SwiftUI.
- Cần chạy trên điện thoại → web app (`/ts-tao-app`) hoặc Tauri mobile (để sau).

## Bài học từ TISA Clean (17/11/2026)
- Bỏ "quick pane" của mẫu: xóa `commands/quick_pane.rs`, `quick-pane.html`, `src/quick-pane-main.tsx`, `components/quick-pane`, ShortcutPicker, plugin `tauri-nspanel` + `global-shortcut`, mục `quick-pane` trong vite.config và capabilities.
- Số lớn (dung lượng, thời gian) gửi sang giao diện dùng `f64`, vì specta không xuất được `u64`. Kiểu chỉ dùng trong sự kiện phải khai báo `.typ::<T>()` trong `bindings.rs`.
- Việc quét nặng chạy bằng `rayon::ThreadPoolBuilder` riêng, để các lệnh khác không bị kẹt chờ.
- Updater chưa có khóa: đặt `plugins.updater.active=false`, `createUpdaterArtifacts=false`, bỏ `check()` trong App.tsx.
- Mở URL `x-apple.systempreferences:` cần quyền `opener:allow-open-url` có scope riêng.
- Thử giao diện bằng computer-use: nếu người dùng đang ở Space toàn màn hình thì không bấm được. Chụp cửa sổ app bằng `screencapture -l <window_id>`, không chụp cả màn hình.
- `mkfile -n` tạo file "rỗng ruột" (không chiếm chỗ thật). Tạo file thử bằng `mkfile` không có `-n`.
