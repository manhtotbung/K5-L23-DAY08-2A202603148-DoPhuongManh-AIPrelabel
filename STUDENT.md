# Thông tin học viên

- Họ và tên: Đỗ Phương Mạnh
- MSHV: 2A202603148
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: manhtotbung
- Repo bài nộp (URL): https://github.com/manhtotbung/K5-L23-DAY08-2A202603148-DoPhuongManh-AIPrelabel
- Commit chốt bài (hash): cd2eb7ef7afd1fb17c7b4f0706a7ef081bb52f2c
- Máy: Windows 11 Pro, 16 GB RAM, không GPU (CPU, torch 2.15.0+cpu)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt

- Máy, nguồn `submission/qc/review_stats.json`. Accepted 416/511 = 81,4%, edited 27/511 = 5,3%, deleted 68/511 = 13,3%. Added 126/569 = 22,1% đối tượng sau duyệt. Các vùng quanh bẫy được loại khỏi bảng error profile theo script.
- So với 4 ảnh gold, precision của máy 50/66 = 75,8%, recall 50/61 = 82,0%.
- Bắt bẫy 8/12 = 66,7%, thấp hơn mức đạt ít nhất 10/12. Còn ba box lỏng và một vật sót.
- Thời gian từ `submission/time_log.csv`. Gán tay 30 phút / 4 ảnh = 7,50 phút/ảnh. Review 26,53 phút / 24 ảnh = 1,105 phút/ảnh, gồm đối chiếu ảnh, sửa XML và nạp lại CVAT.
- QC bản trước calibration, nguồn `submission/qc/qc_review.csv` và `qc_sample.csv`. 7/8 = 87,5% ảnh có lỗi, 11/224 = 4,9% đối tượng hoặc vị trí vật bị ghi lỗi. Mẫu gồm 3 ca khó và 5 ảnh ngẫu nhiên, seed 42. Đã đối chiếu khớp `qc_report_auto.md`, cận trên Wilson 95% của tỉ lệ ảnh lỗi là 98% từ 7/8 ảnh lỗi.
- **Quyết định là Rework**, khớp `qc_report_auto.md`. Lượt QC trước sửa có 7/8 ảnh lỗi và bắt bẫy 8/12, đều chưa đạt. Đã sửa calibration trong CVAT, có `qc_fixed.json` và build lại release đủ 24 ảnh, 585 đối tượng. Vòng sửa mới xử lý 3/11 = 27,3% lỗi QC ở img_008, còn 8/11 = 72,7% lỗi chưa sửa ở img_003, img_009, img_017, img_018, img_024 và img_035. Hai thay đổi can_review ở img_009 và img_035 ghi nhận ca đã chốt quy tắc. Cần sửa 8 lỗi còn lại và QC lại trước khi cân nhắc Release.
- Calibration có 10 quyết định, gồm 3 `loi_may` và 7 `cau_hoi_mo`, dẫn ảnh và có quy tắc trong `guideline_v1.1_calibrated.md`.
