# Requirement Document — Trip Planner App (từ bản mẫu "Đà Nẵng · Hội An")

> Tài liệu này mô tả lại toàn bộ đặc điểm, requirement và chức năng của bản app hiện tại (1 file HTML tĩnh, dữ liệu cứng cho 1 chuyến đi cụ thể) để làm **input đầu vào** cho quy trình BA → Dev → Test xây lại thành một app hoàn chỉnh hơn (đa chuyến đi, nhiều người dùng, có backend...). Không phải spec kỹ thuật của bản build lại — là bản ghi "app hiện tại làm được gì" + đề xuất khoảng trống cần lấp ở bản sau.

## 1. Bối cảnh & mục tiêu sản phẩm

- **Loại sản phẩm:** Web app lập kế hoạch chuyến đi cá nhân (trip planner / itinerary tracker) kiêm sổ chi tiêu, dùng chung cho 1 cặp đôi/nhóm nhỏ đi chung.
- **Hiện trạng kỹ thuật:** 1 file `index.html` tĩnh (HTML + CSS + vanilla JS, ~900 dòng), không backend, không build step. Dữ liệu chuyến đi (lịch trình, quán ăn, vé máy bay...) được viết cứng trong JS. Lưu trạng thái chỉnh sửa của người dùng bằng `localStorage` của trình duyệt.
- **Hosting hiện tại:** GitHub Pages (public repo) + bản sao trên Claude Artifact.
- **Giới hạn cốt lõi cần giải quyết ở bản sau:** dữ liệu chỉ tồn tại trên 1 trình duyệt/thiết bị, không đồng bộ nhiều người/nhiều máy, không có tài khoản, mỗi chuyến đi phải sửa code để tạo bản mới. Xem mục 8.

## 2. Actor / Người dùng

| Actor | Mô tả | Quyền hạn hiện tại |
|---|---|---|
| **Người đi chuyến** (2 người dùng chung 1 link) | Cặp đôi lên kế hoạch và đi chuyến thực tế | Xem + sửa toàn bộ nội dung, không phân quyền, không đăng nhập |

> Bản hiện tại **không phân biệt vai trò** — ai có link cũng sửa/xoá được mọi thứ. Đây là điểm BA cần quyết định lại cho bản sau (có cần vai trò chủ chuyến đi / thành viên / khách xem-only không).

## 3. Kiến trúc thông tin (Information Architecture)

App có **5 tab** điều hướng bằng thanh dưới cùng (bottom nav, cố định, luôn hiển thị):

1. **Lịch trình** (mặc định khi mở app)
2. **Ăn uống**
3. **Muốn đi**
4. **Vé bay**
5. **Ngân sách**

Header cố định phía trên (không đổi giữa các tab): tên chuyến đi, số ngày, tên người tham gia, khoảng ngày đi, và 1 dòng trạng thái lưu ("Đang lưu…" / "Đã lưu.").

## 4. Chức năng theo từng màn hình

### 4.1 Tab Lịch trình

- **Chọn ngày:** dải "chip" cuộn ngang, mỗi chip hiển thị ngày/thứ + số thứ tự ngày (Ngày 1..N). Chạm để chuyển ngày, chip đang chọn được tô nổi bật.
- **Mỗi ngày gồm:** tiêu đề ngày, mô tả ngắn, và danh sách sự kiện dạng timeline (chấm tròn nối theo trục dọc).
- **Mỗi sự kiện (event) có:**
  - **Giờ:** input giờ dạng native (`<input type="time">`) — chạm vào mở bánh xe chọn giờ:phút của hệ điều hành, không phải gõ chữ tự do.
  - **Tên điểm:** text sửa được trực tiếp (chạm vào, gõ, rời khỏi ô là tự lưu).
  - **Ghi chú:** text sửa được trực tiếp (chỉ hiện nếu có ghi chú gốc hoặc người dùng đã thêm).
  - **Badge "Dự trù: [giá]":** hiển thị chi phí ước tính tĩnh do người tạo lịch trình ghi sẵn (không sửa được ở đây, mang tính tham khảo — khác với "đã chi thực tế" ở tab Ngân sách).
  - **Cảnh báo (flag):** hộp cảnh báo màu nổi bật cho các điểm cần lưu ý/cần chốt thêm (ví dụ: giờ hoạt động đặc biệt, thiếu thông tin) — do người tạo gắn sẵn, hiển thị tĩnh.
  - **Địa chỉ Google Maps:** ô text sửa được (chạm vào, gõ địa chỉ/tên địa điểm) + nút ghim bên cạnh — bấm nút mở Google Maps (tab mới) tìm kiếm đúng theo nội dung đang hiển thị trong ô tại thời điểm bấm.
  - **Nút xoá:** xoá sự kiện khỏi lịch trình (có hộp thoại xác nhận).
  - **Checkbox "Đã đi":** đánh dấu đã hoàn thành điểm này.
- **Popup khi tích "Đã đi":** ngay khi tích (chỉ khi chuyển từ chưa tích → tích, không hiện lại khi bỏ tích), hiện popup dạng bottom-sheet cho phép:
  - Sửa lại **tên điểm**
  - Sửa lại **giờ**
  - Nhập **giá đã chi thực tế** cho điểm này
  - 2 lựa chọn: **"Lưu vào Ngân sách"** (ghi tên/giờ mới vào lịch trình + ghi/khớp giá vào đúng danh mục "Ngày N" tương ứng bên tab Ngân sách — nếu điểm đã có khoản chi từ trước thì cập nhật đè, không tạo trùng) hoặc **"Bỏ qua"** (đóng popup, không lưu gì, ô "Đã đi" vẫn giữ trạng thái đã tích).
  - Có thể đóng popup bằng cách chạm ra ngoài vùng nội dung (tương đương Bỏ qua).
- **Thêm sự kiện mới:** nút "+ Thêm mục" cuối mỗi ngày, tạo 1 sự kiện trống (giờ để trống, tên "chạm để sửa"), tự focus vào ô tên để gõ ngay.

### 4.2 Tab Ăn uống / Tab Muốn đi (2 tab có cấu trúc giống hệt nhau)

- Danh sách địa điểm **nhóm theo thành phố** (Hội An / Đà Nẵng), mỗi nhóm có tiêu đề nhóm riêng.
- **Mỗi thẻ địa điểm có:**
  - Tên (sửa được trực tiếp), badge "Nổi bật" tuỳ chọn cho địa điểm được đánh dấu nên đi.
  - Địa chỉ, giờ mở cửa, khoảng giá — hiển thị tĩnh (không sửa trực tiếp, chỉ có trong dữ liệu gốc).
  - Ghi chú — sửa được trực tiếp (điều kiện hiện giống mục 4.1).
  - Ô địa chỉ Google Maps sửa được + nút mở Maps — logic giống hệt tab Lịch trình. Giá trị mặc định của ô này được ghép tự động từ tên + địa chỉ + thành phố.
  - Checkbox "Đã đi" — **không** kèm popup nhập giá (khác với tab Lịch trình).
  - Nút xoá.
- **Thêm địa điểm mới:** nút "+ Thêm địa điểm" cuối mỗi nhóm thành phố.

### 4.3 Tab Vé bay

- Nội dung **hoàn toàn tĩnh, không sửa được**:
  - Vé chặng đi & chặng về: sân bay đi/đến, ngày bay, giờ bay, giá vé ước tính — trình bày dạng "vé máy bay" (ticket UI) có đường viền đứt kiểu xé vé.
  - Thẻ thuê xe: loại xe, thời gian/địa điểm nhận-trả, giá thuê, ghi chú tiền cọc.

### 4.4 Tab Ngân sách

- **Đây là nơi duy nhất ghi nhận tiền đã chi thực tế của cả chuyến** (đã bỏ khái niệm "dự trù/ước tính" ở tab này — chỉ còn số tiền thật đã/sẽ chi).
- **1 thẻ tổng** ở đầu trang: "Đã chi thực tế" = tổng tất cả danh mục bên dưới, cập nhật theo thời gian thực khi gõ.
- **Danh mục cố định:**
  - **Đi lại** — các khoản vé máy bay, thuê xe.
  - **Khách sạn** — từng đêm khách sạn/homestay.
  - **Ngày 1 → Ngày N** — 1 danh mục riêng cho mỗi ngày của chuyến đi, dùng để ghi chi tiêu ăn uống/hoạt động phát sinh trong ngày đó. Có thể được điền sẵn tự động từ các khoản "Dự trù" đã gắn ở sự kiện lịch trình ngày đó (xem 4.1), hoặc thêm thủ công, hoặc tự thêm qua popup ở 4.1.
- **Mỗi khoản trong danh mục có:** tên (sửa được trực tiếp), số tiền (ô nhập số, để trống = 0/chưa có giá), nút xoá.
- **Thêm khoản mới:** nút "+ Thêm khoản" trong mỗi danh mục.
- Subtotal mỗi danh mục và tổng cuối cùng tự tính lại ngay khi người dùng gõ số, không cần bấm lưu riêng.

## 5. Cơ chế xuyên suốt (Cross-cutting)

- **Sửa trực tiếp tại chỗ (inline edit):** hầu hết text (tên, ghi chú, địa chỉ Maps, tên khoản ngân sách) là vùng `contenteditable` — chạm vào gõ trực tiếp, lưu khi rời khỏi ô (blur), không có nút "Lưu" riêng.
- **Lưu tự động:** mọi thay đổi (tick đã đi, sửa text, sửa số tiền, thêm/xoá mục) được gộp vào 1 object trạng thái và ghi xuống `localStorage` của trình duyệt sau 350ms debounce; có dòng chữ báo "Đang lưu…/Đã lưu." ở đầu trang.
- **Không có backend:** không có API, không có database — toàn bộ là client-side. Dữ liệu **không đồng bộ** giữa các thiết bị/trình duyệt khác nhau.
- **Cơ chế lưu thay đổi trên nền dữ liệu gốc (không sửa trực tiếp mảng gốc):**
  - Text override theo từng field-id (vd. `d4-1-title`).
  - Đánh dấu "đã xoá" (soft-delete) cho các mục gốc thay vì xoá thật khỏi mảng.
  - Danh sách mục "tự thêm" riêng theo từng section (mỗi tab/ngày/danh mục ngân sách có 1 danh sách mục thêm tay).
  - Trạng thái "đã đi" theo từng id.
- **Tích hợp Google Maps:** chỉ là **link tìm kiếm** (`google.com/maps/search/?query=...`) dựng từ text người dùng nhập — **không** dùng Google Maps API thật (không autocomplete, không geocoding, không hiển thị bản đồ nhúng, không tính khoảng cách).
- **Không có xác thực người dùng**, không phân quyền, ai có link cũng sửa được.
- **Ngôn ngữ:** tiếng Việt duy nhất, không có i18n.
- **Giao diện:** chỉ có chế độ sáng (`color-scheme: light only`), không có dark mode.

## 6. Data model (được suy ra từ dữ liệu cứng trong code)

```
Trip (ngầm định, chỉ có 1 chuyến)
├─ title, travelers[], dateRange

Day
├─ key, label ("Ngày 1"), date, title, sub
└─ events: Event[]

Event
├─ id, time, title
├─ note? , cost? (chỉ hiển thị, không phải số thật đã chi)
├─ flag? (cảnh báo tĩnh)
├─ map? (địa chỉ tìm kiếm mặc định)
└─ flight? (boolean — style vé máy bay riêng trên timeline)

Place (dùng chung cho Food & Wishlist)
├─ id, name, addr?, hours?, price?, note?, top? (badge nổi bật)

FlightTicket (tĩnh, không phải data-driven — hardcode trong HTML)
├─ route, date, time, estimatedPrice

CarRental (tĩnh)
├─ model, pickupInfo, returnInfo, price, depositNote

BudgetCategory
├─ id, label
└─ items: BudgetItem[]

BudgetItem
├─ id, label, amount

PersistedState (per-browser, trong localStorage, key "trip-edits")
├─ visited: { [id]: boolean }
├─ text: { [fieldKey]: string }       // override cho mọi field sửa tay
├─ removed: { [id]: boolean }         // soft-delete mục gốc
└─ custom: { [section]: Item[] }      // mục tự thêm theo từng khu vực
```

## 7. Yêu cầu phi chức năng (Non-functional)

- **Nền tảng mục tiêu:** mobile-first, tối ưu cho màn hình điện thoại (layout giới hạn max-width ~520px), mở qua trình duyệt di động (không phải app native).
- **Hiệu năng:** tải tức thời (không gọi API ngoài trừ Google Fonts và link Google Maps khi bấm), không cần build/deploy phức tạp.
- **Khả dụng:** phụ thuộc hoàn toàn vào static hosting (GitHub Pages) — không có SLA, không có backup dữ liệu người dùng phía server (vì không có server).
- **Bảo mật/riêng tư:** **không có** — link public, ai cũng xem/sửa được; nội dung chứa thông tin cá nhân (tên thật, số tiền chi tiêu thật).
- **Khả năng offline:** không có service worker/cache — cần mạng để tải lần đầu; không phải PWA.
- **Khả năng mở rộng dữ liệu:** để tạo chuyến đi mới hiện phải sao chép file và sửa code tay — không có UI tạo chuyến đi mới.

## 8. Giới hạn hiện tại & đề xuất khoảng trống cho bản build lại (v2)

Đây là danh sách các điểm **BA nên xác nhận lại phạm vi** khi viết requirement chính thức cho bản v2:

1. **Đa người dùng thật sự** — cần tài khoản/đăng nhập, phân quyền (chủ chuyến đi sửa được gì, thành viên được mời sửa được gì, khách xem-only).
2. **Đồng bộ nhiều thiết bị** — cần backend + database thay cho `localStorage`, để 2 người cùng xem/sửa 1 chuyến đi theo thời gian thực hoặc gần thời gian thực (cân nhắc xử lý xung đột khi 2 người sửa cùng lúc — bản hiện tại không xử lý, ai lưu sau ghi đè người lưu trước).
3. **Đa chuyến đi** — cho phép tạo/quản lý nhiều chuyến đi khác nhau trong cùng 1 app, thay vì 1 file = 1 chuyến đi cố định.
4. **Google Maps thật** — cân nhắc dùng Places Autocomplete/Geocoding API thay vì link tìm kiếm suông, để địa chỉ chính xác hơn và có thể hiển thị bản đồ nhúng.
5. **Ngân sách** — cân nhắc thêm: phân loại theo người trả (ai ứng tiền, chia đều/chia riêng), biểu đồ trực quan theo danh mục, xuất báo cáo.
6. **Thông báo/nhắc nhở** — nhắc giờ theo lịch trình, nhắc check-in/check-out khách sạn.
7. **Ảnh/tệp đính kèm** — cho phép gắn ảnh vé, hoá đơn, ảnh địa điểm.
8. **Sao lưu/khôi phục dữ liệu** — export/import, hoặc tự động backup định kỳ.
9. **Khả năng truy cập (accessibility)** — bản hiện tại dùng nhiều `contenteditable` chưa được kiểm tra kỹ với screen reader; cần audit lại nếu v2 hướng tới chuẩn accessibility.
10. **Kiểm thử tự động** — bản hiện tại không có test nào (chỉ kiểm thử thủ công qua preview); v2 nên có test coverage theo từng luồng nêu ở mục 4.

## 9. Tham chiếu

- Bản hiện tại (live): https://phillipvna.github.io/da-nang-hoi-an-trip/
- Mã nguồn: repo GitHub `phillipvna/da-nang-hoi-an-trip`, file duy nhất `index.html`.
