---
title: Shop Campaigns - Hướng dẫn test trên prod
nav_order: 6
---
# Shop Campaigns: hướng dẫn test trên môi trường prod

Tài liệu cho người test không cần biết kỹ thuật. Làm theo từng bước, so với ảnh mẫu, ghi lại chỗ nào khác. Bản dành cho dev: [Hướng dẫn test trên dev](shop-campaigns-testing-guide.html).

**Prod là shop thật, dữ liệu thật.** Đọc kỹ mục 3 và mục 4 trước khi bấm bất cứ nút nào.

## 1. Tính năng này là gì

**Shop Campaigns** là quảng cáo merchant chạy ngay trong Shopify (Shopify tính tiền). Trước đây GoProfit không thấy khoản chi này, nên lợi nhuận bị báo cao hơn thực tế. Từ bản này GoProfit coi Shop Campaigns là **một kênh quảng cáo thứ 5**, cạnh Facebook, Google, TikTok và Pinterest.

Những chỗ thay đổi trong app:

| Màn hình | Thay đổi |
|---|---|
| Settings → Marketing integration | Có thêm dòng **Shop Campaigns** với công tắc bật/tắt và một dòng trạng thái |
| Marketing | Bảng Channel performance có thêm dòng **Shop Campaigns**, mở ra thấy từng chiến dịch |
| Home → Profit breakdown | Mở dòng **Ad spend** thấy thêm dòng **Shop Campaigns** |
| Profit & loss | Mở dòng **Advertising spend** thấy thêm dòng **Shop Campaigns** |

Khác với dev, trên prod:

- Tính năng đã **mở cho mọi shop** (feature flag `shop-campaigns` ở trạng thái **On**). Shop nào cũng thấy công tắc trong Settings.
- Mặc định công tắc **BẬT**. GoProfit tự lấy dữ liệu Shop Campaigns, merchant không cần làm gì. Merchant có thể tự tắt, khi đó Marketing không còn dòng Shop Campaigns.

Ảnh mẫu bên dưới chụp trên **dev**: **khung cam có số** là chỗ thay đổi cần nhìn, giao diện prod giống nhưng số liệu khác. Viền đỏ quanh app và dải vàng **CS MODE** ở trên cùng là của extension Open Shop, không phải của app.

## 2. Chuẩn bị

1. Đăng nhập Shopify Admin của store test, mở app **GoProfit Analytics** (bản prod, không phải Profit App dev).
2. Muốn xem shop khác: cài extension Chrome **GODISOFT Open Shop**, chọn môi trường **Prod**, nhập **ShopId**, bấm mở. Chờ dải vàng hiện `CS MODE: <domain> (<ShopId>)` rồi mới bắt đầu test. Thoát CS MODE cũng làm trong extension.
3. Chỉ dùng CS MODE để **xem**. Trong CS MODE múi giờ và đơn vị tiền ở một số chỗ có thể theo store gốc. Nếu chỉ lệch giờ hoặc tiền tệ thì ghi chú lại, chưa coi là lỗi.

## 3. Quy định an toàn trên prod

- **Không bấm công tắc Shop Campaigns của shop merchant thật.** Bật/tắt chỉ làm trên store của team hoặc store bạn tự tạo (mục 4).
- Mọi bước ở UC1–UC6 và UC8 chỉ **xem**.
- Không sửa feature flag trên trang admin. Chỉ xem, bấm **Hủy** để đóng.
- Không chụp ảnh lộ ShopId hay domain của merchant thật rồi gửi ra ngoài team. Che trước khi gửi.

## 4. Shop và dữ liệu test

Prod **không có** shop mẫu được sao chép sẵn như dev, và **không có cách tạo dữ liệu giả**. Chi tiêu Shop Campaigns chỉ có khi một shop thật chạy quảng cáo trong Shopify. Vì vậy cần chuẩn bị theo bảng sau. Cột ShopId để người test tự điền.

| Mã | Loại shop | ShopId | Dùng để test |
|---|---|---|---|
| P1 | Shop thật **có chi tiêu** Shop Campaigns (USD). Chỉ xem | _tự điền_ | UC1–UC4 |
| P2 | Store của team, **không có** chi tiêu | _tự điền_ | UC1 (trạng thái "no activity"), UC7 |
| P3 | Shop dùng tiền khác USD, có chi tiêu (nếu tìm được) | _tự điền_ | UC5 |
| P4 | Shop **chưa cấp quyền đọc báo cáo** (`read_reports`) cho app (nếu tìm được) | _tự điền_ | UC6 |
| P5 | Shop đã **tắt** công tắc Shop Campaigns (nếu tìm được) | _tự điền_ | UC8 |

Mã đầy đủ của P1 (shop merchant thật, đã che) hỏi team dev. Shop nào chưa có dữ liệu phù hợp thì ghi "chưa test được" trong báo cáo, **không tự dựng**.

### Muốn có thêm dữ liệu test thì làm thế nào

Chỉ có các cách thật sau:

1. **Chọn shop merchant thật đã có chi tiêu Shop Campaigns** và test ở chế độ xem.
2. **Tự chạy chiến dịch Shop Campaigns** trên store test của bạn, tốn tiền thật, và chờ có chi tiêu. Chỉ làm khi team đồng ý ngân sách.
3. **Dùng store test**, chấp nhận dữ liệu bằng 0: đủ để test UC1 (no activity) và UC7.

Sau khi công tắc bật (mặc định, hoặc sau khi bật lại ở UC7):

- App tự đặt lịch lấy dữ liệu ngay, rồi lặp lại mỗi **150 phút**. Trong lúc chờ, dòng trạng thái là `Enabled · collecting data`.
- Lần đầu app lấy lịch sử từ đầu thời gian gói cước cho phép đến hết hôm qua (theo múi giờ shop). Shop chưa có dữ liệu thì chưa thấy gì ngay, đó là bình thường.
- Shop thiếu quyền `read_reports` sẽ không có số. Merchant mở app và đồng ý cấp thêm quyền thì mới lấy được.

### Shop nào được thấy tính năng: màn admin

Trang admin prod: `https://admin.app.goprofit.io` → menu **Feature Flags**.

1. Dòng **shop-campaigns** (khung 1) hiện State là **On**. Ảnh mẫu bên dưới là của dev nên ghi Allowlist.

![Admin danh sách feature flag (ảnh mẫu dev)](assets/images/shop-campaigns-testing/08-admin-feature-flags-list.png)

2. State có 3 giá trị: **Off** là tắt với mọi shop, **Allowlist** là chỉ bật cho các shop trong danh sách, **On** là bật cho mọi shop. Trên prod danh sách Allowlist để trống vì flag đang là On.

![Admin chi tiết flag (ảnh mẫu dev)](assets/images/shop-campaigns-testing/09-admin-shop-campaigns-allowlist.png)

Người test chỉ xem. Thay đổi flag nhờ team dev.

## 5. Các use case

### UC1. Xem trạng thái trong Settings

**Shop:** P2, P1, P4 · **Đường đi:** Settings → Marketing integration → kéo xuống cuối khối Ad platforms.

Kết quả đúng:

- Có dòng **Shop Campaigns**, chữ "Billed by Shopify.", công tắc ở bên phải.
- Dòng trạng thái thay đổi theo shop:

| Shop | Dòng trạng thái mong đợi |
|---|---|
| P2 | `Enabled · no Shop Campaigns activity yet` |
| P1 | `Enabled · updated <ngày gần nhất>` |
| P4 | `Enabled · collecting data` (không có chữ nào nói về quyền) |

Shop chưa có chi tiêu (ảnh mẫu dev):

![Settings shop chưa có chi tiêu](assets/images/shop-campaigns-testing/01-settings-dev-store-no-activity.png)

Shop có chi tiêu (1 = dòng Shop Campaigns và công tắc):

![Settings shop có chi tiêu](assets/images/shop-campaigns-testing/02-settings-s1-enabled.png)

### UC2. Màn Marketing có dòng Shop Campaigns

**Shop:** P1 · **Đường đi:** menu Marketing → bảng **Channel performance**.

1. Tìm dòng **Shop Campaigns** (khung 1).
2. Bấm mũi tên bên trái để mở. Bên dưới hiện từng chiến dịch (khung 2).

Kết quả đúng:

- Số của dòng Shop Campaigns bằng tổng các chiến dịch con.
- Dòng **Total** cộng tất cả kênh, gồm cả Shop Campaigns.
- Không có biểu tượng cảnh báo cạnh Shop Campaigns.

![Marketing](assets/images/shop-campaigns-testing/03-marketing-s1-shop-campaigns-row.png)

### UC3. Home: chi phí quảng cáo có Shop Campaigns

**Shop:** P1 · **Đường đi:** Home → khối **Profit breakdown** → bấm dòng **Ad spend**.

Kết quả đúng: dòng Ad spend (khung 1) mở ra có thêm dòng **Shop Campaigns** (khung 2). Ad spend bằng Facebook + Google + TikTok + Pinterest + Shop Campaigns. Với cùng khoảng ngày, số Shop Campaigns ở đây khớp số ở màn Marketing.

![Home](assets/images/shop-campaigns-testing/04-home-s1-ad-spend-breakdown.png)

### UC4. Profit & loss: dòng Shop Campaigns theo tháng

**Shop:** P1 · **Đường đi:** Profit & loss → bấm dòng **Advertising spend**.

Kết quả đúng:

- Advertising spend (khung 1) mở ra có thêm dòng **Shop Campaigns** (khung 2).
- Advertising spend của mỗi tháng bằng tổng các dòng con.
- Tháng nào shop không có kênh quảng cáo nào khác thì Advertising spend **chỉ gồm Shop Campaigns**.

![Profit & loss](assets/images/shop-campaigns-testing/05-profit-loss-s1-advertising-spend.png)

### UC5. Shop dùng tiền khác USD

**Shop:** P3 (nếu có) · Lặp lại UC2 và UC4.

Kết quả đúng:

- Số tiền hiện bằng đơn vị tiền của shop (ví dụ `473,18 lei`), không phải `$`.
- Màn Marketing và Profit & loss có thể lệch nhau vài xu do quy đổi tỷ giá theo ngày. Đây là hạn chế đã biết, không cần báo.

### UC6. Shop chưa cấp quyền

**Shop:** P4 (nếu có).

1. Vào Settings: dòng Shop Campaigns ghi `Enabled · collecting data` (ảnh dưới, khung 1).
2. Vào Marketing: nếu khoảng ngày đang chọn kéo dài quá thời điểm dữ liệu đã lấy được, dòng Shop Campaigns có **biểu tượng tam giác cảnh báo**. Rê chuột vào thì hiện "May be incomplete". Số có thể là 0 (ảnh dưới, khung 1).

Kết quả đúng: app không báo lỗi, không hiện con số sai. Chỉ cảnh báo là số có thể thiếu.

![Settings shop thiếu quyền](assets/images/shop-campaigns-testing/06-settings-s4-collecting-data.png)

![Marketing shop thiếu quyền](assets/images/shop-campaigns-testing/07-marketing-s4-may-be-incomplete.png)

### UC7. Bật và tắt Shop Campaigns

**Shop:** P2 hoặc store test của bạn. **Không làm trên shop merchant thật.**

1. Settings → tắt công tắc Shop Campaigns. Trang không tải lại. Dòng trạng thái đổi sang tắt.
2. Vào Marketing: dòng Shop Campaigns **biến mất** khỏi bảng.
3. Quay lại Settings → bật công tắc. Dòng trạng thái quay về `Enabled · …`.
4. Vào Marketing: dòng Shop Campaigns **xuất hiện lại**.

Kết quả đúng: mỗi lần bấm chỉ đổi một lần, không nháy qua lại, không có thông báo lỗi. Sau khi bật lại, dòng trạng thái có thể là `collecting data` trong lúc chờ lấy dữ liệu.

### UC8. Shop đã tắt Shop Campaigns

**Shop:** P5 (hoặc P2 sau khi tắt ở UC7 bước 1, chưa bật lại).

Kết quả đúng: Settings **có** dòng Shop Campaigns nhưng công tắc ở trạng thái **tắt**, do merchant đã tắt. Màn Marketing **không có** dòng Shop Campaigns, Ad spend và Advertising spend không có dòng Shop Campaigns. Các số khác không đổi so với trước đây.

## 6. Khi thấy chỗ khác ảnh mẫu

Gửi cho team dev các thông tin sau:

1. Mã shop, ví dụ: `P1`.
2. Use case và bước, ví dụ: `UC3 bước 1`.
3. Ảnh chụp **cả màn hình**, có dải vàng CS MODE nếu đang dùng. Che ShopId và domain của merchant thật.
4. Thời điểm test (giờ Việt Nam).
5. Khoảng ngày đang chọn trên màn hình.

Ảnh mẫu chụp trên dev ngày 14/10/2026, nên trên prod sẽ ra số khác. Cái cần kiểm là **các dòng có xuất hiện và có cộng khớp nhau không**, không phải đúng từng con số trong ảnh.
