# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 511; đối tượng sau duyệt: 569; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 416 | 416/511 = 81.4% | 511 box máy |
| edited (sửa) | 27 | 27/511 = 5.3% | 511 box máy |
| deleted (xoá) | 68 | 68/511 = 13.3% | 511 box máy |
| added (vẽ thêm) | 126 | 126/569 = 22.1% | 569 đối tượng sau duyệt |

Trong số deleted + added, **12 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 114.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 239 | 12 | 18 | 31 |
| bicycle | 5 | 0 | 3 | 6 |
| motorbike | 115 | 5 | 9 | 65 |
| car | 46 | 0 | 8 | 12 |
| bus | 3 | 0 | 0 | 5 |
| truck | 1 | 0 | 5 | 3 |
| three_wheeler | 7 | 10 | 25 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 61 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 50 | 16 | 11 | 75.8% (KTC 95%: 64–84%) | 82.0% (KTC 95%: 71–90%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - extra: 3/3
  - loose: 0/3
  - wrong_class: 3/3
  - lọt: img_005.jpg (loose), img_021.jpg (loose), img_028.jpg (loose), img_024.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
