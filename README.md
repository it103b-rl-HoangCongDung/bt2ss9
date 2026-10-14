# BÁO CÁO PHÂN TÍCH VÀ KHẮC PHỤC LỖI SAO CHÉP NÔNG ĐỐI TƯỢNG HỒ SƠ PHÒNG KHÁCH SẠN

> 👤 **Học viên:** Hoàng Công Dũng | **Mã SV:** PTIT-HCM-009
> 🏫 **Môn học:** it103b-rl-HoangCongDung

---

## 📊 Sơ đồ thiết kế hệ thống (Flowchart)

> 💡 *Sơ đồ dưới đây được render tự động trực tiếp trên GitHub bằng Mermaid. Bạn cũng có thể tải file **`bt2.drawio`** trong repository này để mở và chỉnh sửa trực tiếp trên [Draw.io (diagrams.net)](https://app.diagrams.net).* 

```mermaid
graph TD
  Start([Bắt đầu chương trình]) --> Init[Khởi tạo bookingReservation với dữ liệu gốc]
  Init --> CheckPrice[Truy cập roomPrice qua biến priceKey động]
  CheckPrice --> CheckHour{Giờ check-in < 12?}
  CheckHour -- Đúng --> AddSurcharge[Tính earlySurcharge và gán vào bookingReservation]
  CheckHour -- Sai --> SkipSurcharge[Giữ nguyên earlySurcharge = 0]
  AddSurcharge --> CalcTotal[Tính tổng tiền totalAmount]
  SkipSurcharge --> CalcTotal
  CalcTotal --> CheckVoucher{Mã voucher hợp lệ?}
  CheckVoucher -- Không hợp lệ --> FixDelete[Dùng toán tử delete để xóa hoàn toàn key discountCode]
  CheckVoucher -- Hợp lệ --> KeepVoucher[Giữ nguyên voucher]
  FixDelete --> Display[Vòng lặp for...in với bookingReservation[key]]
  KeepVoucher --> Display
  Display --> VerifyTest[Kiểm tra lại bằng 'discountCode' in bookingReservation trả về false]
  VerifyTest --> End([Kết thúc hoàn thành bài tập])
```

---

## Nhiệm vụ 1: Phân tích lỗi sai AS-IS và Lập bảng Test Cases

Sau khi xem xét kỹ mã nguồn hiện tại trong tệp app.js của hệ thống PMS khách sạn, mình nhận thấy có 3 lỗi logic nghiêm trọng ảnh hưởng trực tiếp đến kết quả tính toán hóa đơn và hiển thị thông tin đặt phòng:

Thứ nhất, việc dùng dấu chấm để truy cập thuộc tính qua biến động thông qua câu lệnh bookingReservation.priceKey sẽ trả về giá trị undefined. Nguyên nhân là vì JavaScript hiểu nhầm priceKey là tên một khóa tĩnh nằm trong đối tượng chứ không phải là một biến chứa tên khóa.

Thứ thứ hai, khi mã voucher không hợp lệ, việc gán giá trị bookingReservation.discountCode = undefined chỉ làm giá trị của thuộc tính đó chuyển thành undefined chứ hoàn toàn không xóa bỏ thuộc tính khỏi đối tượng. Điều này dẫn đến việc key này vẫn bị lặp qua khi duyệt đối tượng.

Thứ ba, trong vòng lặp for...in, lập trình viên đang dùng cú pháp bookingReservation.key thay vì dùng dấu ngoặc vuông bookingReservation[key]. Cách viết sai này khiến chương trình cố gắng tìm một thuộc tính có tên chính xác là chữ key thay vì lấy giá trị của biến key đang lặp.

- Lỗi truy cập biến động: Sử dụng dot notation (.priceKey) thay vì bracket notation ([priceKey]).
- Lỗi xử lý voucher không hợp lệ: Gán undefined làm khóa vẫn tồn tại trong bộ nhớ thay vì dùng toán tử delete.
- Lỗi duyệt vòng lặp đối tượng: In nhầm bookingReservation.key thay vì truy xuất động bookingReservation[key].

| Trường hợp kiểm thử | Dữ liệu đầu vào | Kết quả sai thực tế | Kết quả đúng mong đợi |
| --- | --- | --- | --- |
| TC-01: Truy cập giá phòng qua biến động | priceKey = 'roomPrice' | basePrice nhận giá trị undefined, dẫn đến tính toán totalAmount bị NaN | basePrice lấy đúng giá trị 1500000 từ thuộc tính roomPrice |
| TC-02: Xử lý khi mã voucher không hợp lệ | isVoucherValid = false | Thuộc tính discountCode vẫn tồn tại với giá trị là undefined | Thuộc tính discountCode bị xóa hoàn toàn khỏi bộ nhớ đối tượng |
| TC-03: Hiển thị danh sách thuộc tính qua vòng lặp | bookingReservation chứa 6 thuộc tính | Console in ra toàn bộ giá trị là undefined do lỗi .key | Console in chính xác tên khóa và giá trị tương ứng của từng thuộc tính |

## Nhiệm vụ 2: Sửa lỗi mã nguồn tệp solution_ex2.js

Để giải quyết triệt để các vấn đề trên, mình đã tiến hành viết lại toàn bộ mã nguồn vào tệp solution_ex2.js theo đúng chuẩn cú pháp Javascript hiện đại, an toàn và tối ưu bộ nhớ:

Mình sửa lại cách truy cập biến giá phòng thành bookingReservation[priceKey]. Sau đó, khi voucher không hợp lệ, mình dùng toán tử delete bookingReservation.discountCode để dọn sạch sẽ thuộc tính này khỏi đối tượng. Cuối cùng, trong vòng lặp for...in, mình dùng cú pháp bookingReservation[key] để in đúng giá trị và thêm dòng kiểm tra xác thực bằng toán tử in theo yêu cầu đề bài.

- Đảm bảo mã nguồn chạy mượt mà không phát sinh lỗi NaN hay undefined ngoài ý muốn.
- Tuân thủ nghiêm ngặt quy định không dùng các cấu trúc nâng cao chưa được học trong môn.

---

## 📁 Danh sách tệp tin nộp bài trong Repository
- 📝 `bt2.docx`: Báo cáo tài liệu phân tích nghiệp vụ hoàn chỉnh.
- 🎨 `bt2.drawio`: File thiết kế sơ đồ chuẩn theo quy định đề bài (mở trực tiếp bằng [Draw.io](https://app.diagrams.net) hoặc Lucidchart).
- 💻 `solution_ex2.js`: Mã nguồn chương trình.
