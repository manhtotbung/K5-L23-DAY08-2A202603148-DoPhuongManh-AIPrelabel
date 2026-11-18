# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603148 - Đỗ Phương Mạnh

Bản bổ sung ngày 18/11/2026, dùng cùng guideline v1.0. Các quy tắc dưới đây khớp mã quyết định trong submission/qc/calibration_decisions.csv. Các thay đổi nhãn cần được áp dụng trong CVAT và xuất lại trước khi nhận release theo v1.1.

## 1. Câu hỏi mở đã chốt

| Câu hỏi ở mục 6 của v1.0 | Quy tắc bổ sung | Ảnh ví dụ | case_id |
|---|---|---|---|
| 1. Xe đạp điện và xe máy điện | Xe có động cơ điện thuộc motorbike, xe chỉ đạp chân thuộc bicycle. Dựa vào bộ phận truyền động, động cơ hoặc pin nhìn thấy. Khi nguồn động lực bị che, chọn lớp có căn cứ nhất và can_review. Hai ảnh ví dụ minh hoạ việc kiểm dấu hiệu phương tiện, chưa được xác nhận có xe điện. | img_009, img_014 | CAL-07 |
| 2. Cụm xe đỗ dày gần camera | Vật có phần nhìn thấy đủ ít nhất khoảng 20% và box cao ít nhất 20 px được gán riêng. Cụm ít nhất 5 vật nhỏ dưới 20 px hoặc mờ đến mức không tách được cá thể được bao vùng ignore sát cụm. Vật lớn, rõ được rà riêng dù có phần box giao vùng ignore. | img_024, img_026, img_018 | CAL-05 |
| 3. Hàng hoá nhô khỏi xe | Box giữ thùng, giá và giỏ gắn cố định. Hàng rời nhô quá khoảng 20% chiều dài xe được loại theo v1.0. Với mask, giữ cấu kiện cố định và loại hàng rời theo đường biên nhìn thấy. Chốt can_review của box 118 img_009 khi áp dụng. | img_009, img_003 | CAL-03 |
| 4. Người bị mui che | Gán person khi phần thấy đạt khoảng 20% và box cao ít nhất 20 px. Box bao các mảnh người còn thấy. Khi sát ngưỡng và chưa chốt được thì can_review, không mặc định tick mọi người khó vẽ. Ở img_035, box 573 thấy đầu và thân trên, giữ person và bỏ can_review khi áp dụng. | img_035, img_010 | CAL-09 |
| 5. Kính xe và khung hở, mask nhiều mảnh | Người có mask riêng. Mask xe loại phần người phía trước và khoảng hở lớn hơn khoảng 5% diện tích xe. Giữ kính thuộc xe. Các mảnh xe bị che vẫn cùng một đối tượng, kiểm tra từng mảnh và loại mảnh lạc. Ảnh mask Part 5 đang minh hoạ lỗi, cần sửa các vùng dính trước khi nhận là nhãn mask cuối. | img_013, img_010, img_003 | CAL-04 |
| 6. Box chồng nhau ở ảnh đêm | Các box cùng một vật thì giữ một box đủ phần thấy. Hai vật khác nhau được giữ riêng dù chồng box. Vệt đèn đơn thuần không được coi là một vật. Ca không xác định được theo quy tắc được can_review và ghi vùng cần làm rõ. | img_024, img_029 | CAL-10 |
| 7. Điểm conf để tự động chấp nhận | Không đặt mức conf để chấp nhận tự động. Rà từng vật và hình học trên ảnh gốc. conf dùng xếp thứ tự duyệt, giữ nguyên điểm máy khi đổi lớp hoặc box. | img_008, img_004 | CAL-08 |

## 2. Ca khó mới và cách áp dụng

| case_id | Ảnh | Vấn đề quan sát | Quyết định | Mục bổ sung v1.0 |
|---|---|---|---|---|
| CAL-01 | img_004, img_013 | YOLO26 gọi tuk-tuk là car hoặc tách thành nhiều box. | Gán một three_wheeler cho mỗi xe ba bánh, gồm mui và khung gắn liền xe. Đổi ngưỡng chỉ lọc điểm tin cậy, không sửa lớp. | Bổ sung mục 2 và 4.1 về tuk-tuk và khung mui. |
| CAL-02 | img_008 | Ba xe bán trái cây hai bánh bị gán three_wheeler, các box 109, 110, 111 đang can_review. | Xe đẩy tay hai bánh có một trục, không có bộ phận đạp hoặc động cơ, nằm ngoài các lớp phương tiện của taxonomy. Xóa ba box 109, 110, 111. Không tạo ignore_region cho xe đẩy. | Bổ sung mục 2 về xe đẩy tay ngoài taxonomy. Cần áp dụng trong CVAT trước khi nhận bản release v1.1. |
| CAL-03 | img_009, img_003 | Phạm vi xe và hàng hoá trong box hoặc mask chưa cụ thể. | Box giữ thùng, giá và giỏ gắn cố định. Hàng rời nhô quá khoảng 20% chiều dài xe được loại theo v1.0. Với mask, giữ cấu kiện cố định và loại hàng rời theo đường biên nhìn thấy. Chốt can_review của box 118 img_009 khi áp dụng. | Bổ sung mục 4.1.5 và 4.2 về giá cố định và hàng rời. |
| CAL-04 | img_013, img_010, img_003 | SAM dính người lái hoặc người ngồi trên xe, lấp nền qua khung và tách mask thành nhiều mảnh. | Người có mask riêng. Mask xe loại phần người phía trước và khoảng hở lớn hơn khoảng 5% diện tích xe. Giữ kính thuộc xe. Các mảnh xe bị che vẫn cùng một đối tượng, kiểm tra từng mảnh và loại mảnh lạc. Ảnh mask Part 5 đang minh hoạ lỗi, cần sửa các vùng dính trước khi nhận là nhãn mask cuối. | Bổ sung mục 4.2 về người lái, lỗ hở và đối tượng nhiều mảnh. |
| CAL-05 | img_024, img_026, img_018 | Cụm xe gần và xa bị chồng dày, chưa rõ khi nào dùng ignore_region. | Vật có phần nhìn thấy đủ ít nhất khoảng 20% và box cao ít nhất 20 px được gán riêng. Cụm ít nhất 5 vật nhỏ dưới 20 px hoặc mờ đến mức không tách được cá thể được bao vùng ignore sát cụm. Vật lớn, rõ được rà riêng dù có phần box giao vùng ignore. | Bổ sung mục 4.1.2 và 4.1.6, kiểm kích thước ở ảnh gốc. |
| CAL-06 | img_005, img_021, img_028 | Ba bẫy box lỏng đều lọt, chỉ kiểm có box và đúng lớp chưa đủ. | Kiểm tra lần lượt cạnh trái, trên, phải, dưới của từng box so với phần vật nhìn thấy. Giữ giới hạn sai số khoảng 2 px trên ảnh 1024 px của v1.0. Đánh dấu đã kiểm đủ bốn cạnh trong quy trình review. | Bổ sung checklist hình học mục 4.1.1, phân biệt lỗi thực hiện với thiếu quy tắc. |
| CAL-07 | img_009, img_014 | V1.0 để mở lớp cho xe đạp điện và xe máy điện. | Xe có động cơ điện thuộc motorbike, xe chỉ đạp chân thuộc bicycle. Dựa vào bộ phận truyền động, động cơ hoặc pin nhìn thấy. Khi nguồn động lực bị che, chọn lớp có căn cứ nhất và can_review. Hai ảnh ví dụ minh hoạ việc kiểm dấu hiệu phương tiện, chưa được xác nhận có xe điện. | Bổ sung mục 2 và 4.3 cho xe chạy điện, không suy lớp từ dáng người lái. |
| CAL-08 | img_008, img_004 | YOLOE gán xe đẩy trái cây là three_wheeler ở conf 0.90, YOLO26 gán tuk-tuk là car. | Không đặt mức conf để chấp nhận tự động. Rà từng vật và hình học trên ảnh gốc. conf dùng xếp thứ tự duyệt, giữ nguyên điểm máy khi đổi lớp hoặc box. | Bổ sung mục 3, trả lời câu hỏi mở 6.7 về conf. |
| CAL-09 | img_035, img_010 | Người bị mui và ghế che, chưa rõ dùng can_review và ước lượng phần nhìn thấy. | Gán person khi phần thấy đạt khoảng 20% và box cao ít nhất 20 px. Box bao các mảnh người còn thấy. Khi sát ngưỡng và chưa chốt được thì can_review, không mặc định tick mọi người khó vẽ. Ở img_035, box 573 thấy đầu và thân trên, giữ person và bỏ can_review khi áp dụng. | Bổ sung mục 4.1.2 và 4.3 cho người bị mui che. |
| CAL-10 | img_024, img_029 | Nhãn ảnh đêm chồng nhau hoặc có vệt đèn, chưa rõ gộp box thế nào. | Các box cùng một vật thì giữ một box đủ phần thấy. Hai vật khác nhau được giữ riêng dù chồng box. Vệt đèn đơn thuần không được coi là một vật. Ca không xác định được theo quy tắc được can_review và ghi vùng cần làm rõ. | Bổ sung mục 4.1 và 4.3 về vật thực, box trùng và chuyển động nhòe. |

Phân biệt nguyên nhân. CAL-02, CAL-03, CAL-04, CAL-05, CAL-07, CAL-09 và CAL-10 cụ thể hoá các câu hỏi hoặc ca còn mơ hồ. CAL-01, CAL-06 và CAL-08 nhắc cách áp dụng quy tắc sẵn có khi máy gán sai hoặc người review để lọt lỗi. Các ảnh mask Part 5 lưu kết quả thử còn lỗi, cần chỉnh vùng dính trước khi nhận là mask cuối.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do và ảnh ví dụ |
|---|---|---|
| v1.0 | Bản giảng viên | Bộ quy tắc nền. |
| v1.1 | Làm rõ tuk-tuk và xe đẩy tay hai bánh ngoài taxonomy. | CAL-01, CAL-02, img_004, img_008, img_013. |
| v1.1 | Chốt hàng rời, cấu kiện cố định, người dính mask, lỗ hở và các mảnh cùng vật. | CAL-03, CAL-04, img_009, img_003, img_010, img_013. |
| v1.1 | Bổ sung cách duyệt cụm dày, xe điện, người bị che, ảnh đêm, bốn cạnh và conf. | CAL-05 đến CAL-10, img_026, img_018, img_009, img_014, img_035, img_024, img_029, img_005, img_021, img_028, img_008. |
