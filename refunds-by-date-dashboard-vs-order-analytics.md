---
title: Refund - Dashboard/P&L khác Order Analytics thế nào
nav_order: 4
---
# Refund: vì sao Dashboard/P&L và Order Analytics hiển thị khác nhau

## Phần 1. Cách lấy ngày của 2 báo cáo

Hai báo cáo trả lời **hai câu hỏi khác nhau**, nên dùng **hai mốc ngày khác nhau** cho cùng một khoản refund:

| | Dashboard / Profit & Loss | Orders → Order Analytics |
|---|---|---|
| Câu hỏi | Trong ngày này, tiền hoàn lại khách là bao nhiêu? | Các đơn **đặt** trong khoảng này hiện ra sao? |
| Mốc ngày để lọc | **Ngày refund** (thời điểm refund được tạo, theo giờ shop) | **Ngày tạo đơn** |
| Refund của đơn cũ | Tính vào **ngày refund** | Không hiện, vì đơn không thuộc khoảng ngày đã lọc |

Hệ quả: đơn đặt tháng trước, refund hôm nay → Dashboard hôm nay có Returns, còn Order Analytics lọc hôm nay **không thấy đơn đó**.

Quy ước số liệu Returns trên Dashboard/P&L: chỉ tính **tiền sản phẩm**, **đã bỏ VAT**. Tiền ship hoàn không nằm trong Returns, mà trừ vào dòng Shipping.

## Phần 2. Ví dụ: một đơn, hai báo cáo

**Đơn A** (shop dùng GBP, giá đã gồm VAT 20%). Đặt ngày **12/09**, refund toàn bộ ngày **07/10 lúc 22:48** (giờ shop).

### 2.1. Dữ liệu gốc của đơn (từng line)

| Giờ shop | Sự kiện | Line | Số lượng | Tiền (gồm VAT) | VAT | Tiền (không VAT) |
|---|---|---|---|---|---|---|
| 12/09 09:41 | Đặt hàng | Sản phẩm X | 1 | £52.99 | £8.83 | £44.16 |
| 12/09 09:41 | Đặt hàng | Phí ship | - | £5.99 | £1.00 | £4.99 |
| 07/10 22:48 | Refund | Sản phẩm X | -1 | -£52.99 | -£8.83 | -£44.16 |
| 07/10 22:48 | Refund | Phí ship | - | -£5.99 | -£1.00 | -£4.99 |

### 2.2. Dashboard / P&L hiển thị gì

| Ngày xem | Doanh thu sản phẩm | Ship thu | Returns | Giải thích |
|---|---|---|---|---|
| **12/09** | £44.16 | £4.99 | 0 | Ngày đặt hàng: ghi nhận doanh thu |
| **07/10** | 0 | -£4.99 | **£44.16** | Ngày refund: ghi nhận Returns sản phẩm (không VAT). Ship hoàn trừ vào dòng Shipping, không nằm trong Returns |

### 2.3. Order Analytics hiển thị gì

| Khoảng ngày lọc | Đơn A có trong danh sách? | Cột Refunds của Đơn A | Giải thích |
|---|---|---|---|
| **12/09 → 12/09** | Có | £0 | Đơn đặt ngày 12/09 nên có mặt. Refund xảy ra ngày 07/10, ngoài khoảng lọc nên chưa tính |
| **12/09 → 31/10** | Có | £44.16 | Refund nằm trong khoảng lọc |
| **07/10 → 07/10** | **Không** | - | Đơn không được đặt trong khoảng này, dù có refund ngày 07/10 |

### 2.4. Cách đọc kết quả

- Dashboard 07/10 hiện Returns £44.16, nhưng Order Analytics lọc 07/10 không có đơn nào hoàn: **không phải lỗi dữ liệu**. Đơn nằm ở ngày 12/09 trong Order Analytics.
- Muốn tìm đơn đứng sau một khoản Returns, cần tra theo **ngày refund**, không phải ngày tạo đơn.

*Ghi chú: bảng 2.3 theo cách tính hiện tại của hệ thống (cột Refunds chỉ cộng refund nằm trong khoảng ngày lọc). Một số shop cấu hình cũ có thể hiện tổng refund của cả đời đơn.*
