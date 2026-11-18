# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 224 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 11/224 = 4.9% | 224 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 7/8 = 87.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 416/511 = 81.4% / 27/511 = 5.3% / 68/511 = 13.3% | 511 box máy |
| Máy: added | 126/569 = 22.1% | 569 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 4 | img_003.jpg, img_008.jpg |
| sai_vi_tri_kich_thuoc | 4 | img_009.jpg, img_017.jpg, img_018.jpg |
| bo_sot | 2 | img_018.jpg, img_024.jpg |

## Giới hạn thống kê
- 7 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **98%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 87.5% > 2%
- bắt bẫy 67% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
