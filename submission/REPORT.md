# Báo cáo Day 08 — 2A202603148 - Đỗ Phương Mạnh

## 1. Model và recipe (Part 2–3)

| Lớp | Model và cỡ | Ngưỡng dùng khi gộp | Lý do và bằng chứng |
|---|---|---|---|
| person | YOLO26m | 0.30 | r01, r02, r05. Sweep trên 4 ảnh gold chỉ đạt recall tối đa 27/31 = 87,1%, thấp hơn mục tiêu 90%. Giữ mức gộp 0.30, ở mức này recall 24/31 = 77,4%. |
| bicycle | YOLO26m | 0.30 | Gold chỉ có 4 vật, dùng mức gộp chung. Ở mức này recall 2/4 = 50,0%, cần tìm thêm xe bị che. |
| motorbike | YOLO26m | 0.35 | r01, r02, r05. Sweep đạt recall 9/10 = 90,0%, precision 9/11 = 81,8%. Tăng lên 0.50 có thể làm mất xe nhìn rõ như xe ga trắng tiền cảnh ở img_030. |
| car | YOLO26m | 0.30 | Recall cao nhất trong dải ngưỡng đến 0.50 là 7/8 = 87,5%. Ở mức 0.30 precision 7/14 = 50,0%, cần rà box nhầm lớp và trùng. |
| bus, truck | YOLO26m | 0.30 | Gold có 1 bus và 0 truck, chưa đủ để chốt ngưỡng riêng. Dùng mức gộp chung. |
| three_wheeler | YOLOE-s, prompt mặc định tuk-tuk, cyclo, cargo tricycle | 0.15 | r03, r06. Recall trên gold 7/7 = 100%, precision 7/9 = 77,8%. Tìm được tuk-tuk xanh giữa img_002 mà YOLO26 bỏ sót, cần loại xe đẩy tay trong img_008. |

Nguồn sweep là `submission/qc/threshold_sweep.csv` và `threshold_sweep.md`. Các lớp dùng ngưỡng chung được gộp ở 0.30 bằng cờ `--conf`. Lệnh tái lập dưới đây đã được đối chiếu khớp toàn bộ 595 box của tệp gộp đã lưu, trên 28 ảnh batch và gold.

```bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

YOLOE đứng đầu danh sách nguồn để ưu tiên box xe ba bánh khi giao với box car hoặc truck của YOLO26 ở IoU ít nhất 0.6. Sau gộp, 4 ảnh gold dùng để đo máy và 24 ảnh batch dùng để review.

Tôi chọn SAM 2.1, trọng số `sam2.1_b.pt`. r04 ghi nhận 16 mask trên 2 ảnh với thời gian 11,04 giây/ảnh bằng CPU. Phiên bản này đã chạy được trên máy và hỗ trợ lấy mask theo box, sau đó thử tương tác bằng click ở Part 5. Mask ở img_013 vẫn dính người lái và lấp khung hở, nên kết quả cần được đối chiếu biên và quy tắc tách vật. Bảng click và ba ảnh minh hoạ nằm ở mục 5.

Tuk-tuk xanh ở img_004 bị gọi car và xích lô ở img_007 bị gọi motorbike là lỗi lớp. Đổi ngưỡng chỉ giữ hoặc bỏ box theo điểm tin cậy. Tôi dùng YOLOE để bổ sung three_wheeler, rồi kiểm tra loại xe và sửa nhãn theo ảnh. Ở img_008, YOLOE gán xe đẩy tay thành three_wheeler dù conf 0.90. Ở img_002, box YOLOE conf khoảng 0.93 của tuk-tuk xanh đúng vật, còn box conf khoảng 0.37 nằm trên bán tải là sai. Các ca này cho thấy cần kiểm từng vật và cả bốn cạnh box.

Gold gồm 4 ảnh với 61 đối tượng dùng làm mẫu số recall trong `review_stats`. Chọn ngưỡng và đo trên cùng gold làm kết quả lạc quan. Một số lớp có rất ít vật, đặc biệt bicycle 4, bus 1 và truck 0, nên mức ngưỡng hiện tại là lựa chọn ban đầu cần kiểm tiếp trên dữ liệu khác.

## 2. AI error profile (từ `qc/review_stats.md`)

### Quy trình review đã thực hiện cho task 25

Phiên review ngày 18/11/2026 gồm 24 ảnh của job `batch-review`. Các bước giữa export và import được ghi lại từ ảnh gốc, bản XML trước/sau và nhật ký chỉnh sửa trong `work/review_task25/`.

1. Mở job 25 trong CVAT. Ở `img_003.jpg`, bấm biểu tượng con mắt ở đầu danh sách Objects để ẩn toàn bộ nhãn và quét ảnh.
2. Xuất nhãn bằng định dạng CVAT for images 1.1, tắt Save images. Lưu bản gốc thành `work/review_task25/original.xml`, gồm 522 box trên 24 ảnh.
3. Với từng ảnh, mở ảnh gốc để tìm vật trước, sau đó đối chiếu các bản hiển thị box đánh số. Xem riêng nhóm người và phương tiện để giảm chồng chữ. Phóng các vùng đông hoặc mờ thành ảnh crop, riêng vùng giữa giao lộ `img_024.jpg` có thêm lưới tọa độ để chỉnh vị trí.
4. Rà lần lượt vật bị sót, box thừa, sai lớp và box hụt vật hoặc dư nền. Ghi từng thay đổi theo tên ảnh và số box trong `work/review_task25/changes.json`, rồi tạo lại bản hiển thị để kiểm tra. Tổng cộng thêm 122 box, xóa 56 box, đổi lớp 13 box và chỉnh tọa độ 43 box. Một box có thể vừa đổi lớp vừa chỉnh tọa độ. Giữ giá trị `conf` của box máy, để trống `conf` của box bổ sung.
5. Kiểm tra kích thước trên ảnh gốc và bổ sung 8 `ignore_region` tại các cụm nhỏ ở xa trong ảnh 018, 021, 024, 025, 026, 029 và 034. Ảnh 024 có hai vùng. Với ảnh 005 và 028, chưa xác định rõ cụm ít nhất 5 xe dưới 20 px nên giữ từng vật đủ lớn riêng. Kiểm tra cuối cho thấy các box đối tượng còn lại đều cao ít nhất 20 px.
6. Đánh dấu `can_review` cho 17 box trên 12 ảnh có quy tắc cần calibration. Các ảnh gồm 003, 005, 008, 009, 010, 013, 023, 024, 029, 031, 034 và 035. Lưu tên ảnh, lớp, tọa độ box và vấn đề vào `submission/qc/batch_review_notes.csv`.
7. Tạo `work/review_task25/reviewed.xml` từ các thay đổi đã đối chiếu, nạp vào job 25 bằng Upload annotations, định dạng CVAT 1.1 và chế độ Replace. CVAT báo nhập thành công. Kiểm tra lại ảnh 003 có 23 box và ảnh 018 có 56 box cùng vùng ignore đã nạp, sau đó bấm Save.
8. Xuất lại nhãn đã lưu, lưu thành `work/review_task25/saved.xml` và đối chiếu với bản đã review. Kết quả khớp đủ 24 ảnh và 588 đối tượng về lớp, tọa độ, `conf` và `can_review`.

Thời gian tính từ 10:01:00 đến 10:29:32 giờ Việt Nam, trừ hai lần nghỉ một phút sau nhóm 10 ảnh và 20 ảnh, còn 26,53 phút. Khoảng thời gian này gồm đối chiếu ảnh, sửa XML và nạp lại CVAT. Thời gian xuất và kiểm tra sau Save được ghi ngoài thời gian review. Phiên này dùng ảnh gốc và chỉnh tọa độ trong XML, nên lịch sử CVAT thể hiện export, import và Save. Các phím `N`, `Del`, `Ctrl` + số và thao tác kéo góc box là hướng dẫn cho cách review trực tiếp trong giao diện.

Đã chạy `python scripts/validate_labels.py work/review_task25/reviewed.coco.json --expect batch`, kết quả 24 ảnh, 588 đối tượng, OK. Bản COCO kiểm tra được tạo từ XML đã review và bản xuất xác minh từ CVAT là CVAT for images 1.1. Kết quả đo review hiện có trong `submission/qc/review_stats.md` và `submission/qc/review_stats.json`. Các số đếm chỉnh sửa phía trên lấy từ `changes.json` và `summary.json`. `traps_key.json` được giữ kín trong quá trình review.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 416 | 416/511 = 81,4% | 511 box máy |
| edited | 27 | 27/511 = 5,3% | 511 box máy |
| deleted | 68 | 68/511 = 13,3% | 511 box máy |
| added | 126 | 126/569 = 22,1% | 569 đối tượng sau duyệt |

- Xe ba bánh có 25/42 = 59,5% box máy bị xóa, dựa trên 7 accepted + 10 edited + 25 deleted trong review_stats. Ví dụ xe đẩy img_008 bị gọi three_wheeler và mui/thân xe ở img_003 bị tách box. Motorbike chiếm 65/126 = 51,6% tổng số đối tượng được ghi added. Theo cơ chế ghép của script, đổi lớp cũng được tính deleted và added. review_stats ghi 12 trường hợp đổi lớp trong các cặp này, nên cần đọc số added cùng nhận xét theo ảnh.
- So với gold, precision của máy là 50/66 = 75,8%, recall 50/61 = 82,0% trên 4 ảnh. Nguồn review_stats.md và review_stats.json.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Theo `submission/qc/review_stats.md` mục 3 và trường `traps` trong `submission/qc/review_stats.json`, tôi sửa được **8/12 bẫy = 66,7%**, dưới ngưỡng đạt 80% tương ứng ít nhất 10/12 bẫy.

| Loại bẫy | Sửa được | Tỉ lệ | Lỗi còn lọt |
|---|---|---|---|
| Box thừa | 3/3 | 100% | 0/3 |
| Sai lớp | 3/3 | 100% | 0/3 |
| Vật bị sót | 2/3 | 66,7% | 1/3, ở `img_024.jpg` |
| Box lỏng hoặc lệch | 0/3 | 0% | 3/3, ở `img_005.jpg`, `img_021.jpg`, `img_028.jpg` |

Ba box lỏng bị lọt ở các ảnh đông xe máy. Khi đối chiếu, tôi tập trung nhiều vào việc vật đã có box hay chưa và lớp có đúng không. Với box đã bao được vật, tôi chưa kiểm tra đủ cả bốn cạnh có ôm sát phần nhìn thấy của vật hay không. Các box chồng nhau cũng làm việc kiểm tra mép từng box khó hơn. Bước kiểm tra hình học ở cuối checklist chưa được thực hiện kỹ trên từng box, dẫn đến cả ba bẫy lỏng hoặc lệch vẫn còn sau duyệt. Việc đã chỉnh tọa độ một số box khác trong lô chưa bảo đảm phát hiện được ba lỗi này.

Ở `img_024.jpg`, còn một bẫy vật bị sót. Ảnh giao lộ ban đêm có nhiều luồng xe và vệt đèn nhòe, nên cần quét có hệ thống từng vùng để tìm vật chưa có box. Kết quả này cho thấy bước tìm vật độc lập với nhãn máy vẫn còn thiếu sót.

Tỉ lệ accepted là **416/511 box máy = 81,4%**, trong khi bắt bẫy đạt **8/12 = 66,7%**. Hai số đo này cho thấy tôi giữ lại nhiều box máy và vẫn để lọt lỗi hình học cùng một vật bị sót. Kết quả gợi ý thiên kiến tự động hóa khi thấy box có sẵn và đúng lớp thì dễ chấp nhận vị trí của nó. Kiểm tra cấu trúc file và đối chiếu bản xuất sau lưu chỉ xác nhận dữ liệu hợp lệ và được lưu đúng, còn độ ôm sát và vật bị sót cần kiểm tra trực tiếp trên ảnh. Mẫu chỉ có 12 bẫy nên kết luận cần đi cùng danh sách bốn lỗi còn lọt phía trên.

## 4. Máy có đáng dùng không?

Theo `submission/time_log.csv`, gán gold mất 30 phút cho 4 ảnh, tương đương 30/4 = **7,50 phút/ảnh**. Phiên review có pre-label mất 26,53 phút cho 24 ảnh, tương đương 26,53/24 = **1,105 phút/ảnh**, sau khi trừ hai lần nghỉ một phút. Phiên review gồm xem ảnh gốc, chỉnh box qua XML và nạp lại CVAT. Hai nhóm ảnh và cách thao tác khác nhau, gold cũng được làm trước nên đã có kinh nghiệm với cảnh. Chênh lệch thời gian này chưa cô lập được lợi ích của riêng pre-label.

Pre-label giúp tạo điểm bắt đầu cho person, motorbike, car và tuk-tuk. Máy có precision 50/66 = **75,8%** và recall 50/61 = **82,0%** trên gold. Sau review vẫn chỉ bắt được 8/12 = **66,7%** bẫy, trong đó box lỏng là 0/3. Vì vậy, tôi tiếp tục dùng máy để đề xuất nhãn và dành lượt kiểm riêng cho hình học, vật bị sót cùng những lớp dễ nhầm như xe đẩy tay. Quyết định nhận lô ở mục 7 dựa trên QC.

Ba quy tắc cải thiện cho lượt rework tiếp theo là

1. Mỗi ảnh có một lượt quét ảnh gốc trước khi xem box, đi theo bốn góc phần tư và ghi số vật dự kiến theo lớp. Kiểm đủ 24/24 ảnh trong phiếu lượt quét.
2. Với mỗi box, kiểm lần lượt lớp và bốn cạnh trái, trên, phải, dưới. Điểm conf cao vẫn được kiểm. Các ảnh từng lọt box lỏng img_005, img_021 và img_028 phải có lượt kiểm hình học riêng.
3. Mỗi ca chưa rõ quy tắc được tick can_review và ghi tên ảnh, vị trí, câu hỏi cùng case_id calibration. Kiểm lại các ca khó và một mẫu ngẫu nhiên sau khi sửa, chỉ đối chiếu kết quả chấm bẫy sau khi hoàn tất lượt review.

## 5. Mask SAM cho 3 vật (Part 5)

Tôi dùng SAM 2.1 với trọng số `sam2.1_b.pt` cho cả ba vật. Chấm xanh là click dương trên vật cần lấy, chấm đỏ là click âm trên vùng cần loại. Các ảnh trong `submission/masks/` hiển thị kết quả mask cùng vị trí click.

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg`, tuk-tuk | SAM 2.1, `sam2.1_b.pt` | 6 click, gồm 4 dương và 2 âm trên người lái, xe máy bên cạnh | Còn dính người lái. Một số vùng nhìn xuyên qua khung xe còn bị tô, gồm người ngồi phía sau và bàn ghế. | Lấy được mui, thân, đuôi và bánh xe, loại được xe máy bên cạnh. Mask vẫn tràn vào người lái dù đã click âm và còn lấp khoảng hở qua khung. | `tuktuk_img013.jpg` |
| `img_010.jpg`, xích lô đầu hàng | SAM 2.1, `sam2.1_b.pt` | 6 click, gồm 4 dương trên mui và 3 bánh, 2 âm trên người đạp và xích lô sau | Phần lớn người đạp được loại. Vẫn còn những mảng nhỏ dính trên áo người đạp. | Khung, mui và 3 bánh được lấy khá tốt. Vùng ghế và khung quanh người bị lốm đốm, kết quả tách thành 3 polygon. | `xichlo_img010.jpg` |
| `img_003.jpg`, xe đẩy bán hàng | SAM 2.1, `sam2.1_b.pt` | 5 click, gồm 3 dương trên thùng, mui, bánh, 2 âm trên đứa bé và người đàn ông | Gồm cả hàng hoá như trái cây, chậu. Còn dính đứa bé và một phần người đàn ông dù đã click âm. | Lấy được mui, thùng và bánh xe. Mask tràn từ xe sang người và gộp hàng hoá vào xe, nên cần chốt ranh giới xe, người và hàng khi calibration. | `xebanhang_img003.jpg` |

Đối chiếu `guideline/guideline_v1.0_label_contract.md`, mục 4.2, tôi ghi nhận các điểm sau.

- **Quy tắc 1, bám biên phần nhìn thấy.** Cần kiểm tra riêng mui, thân, đuôi và bánh để tìm phần bị thiếu hoặc tràn. Với `img_010.jpg`, vùng ghế và các thanh khung bị lốm đốm cần chỉnh biên và kiểm tra từng mảnh polygon. Xe bị người che cần giữ các phần xe còn thấy, tránh suy đoán phần nằm sau người.
- **Quy tắc 2, lỗ thủng.** Khoảng hở lớn hơn khoảng 5% diện tích vật phải được bỏ khỏi polygon. Ở `img_013.jpg`, vùng nhìn xuyên qua các thanh khung còn bị tô cùng xe, nên cần loại phần nền và các vật phía sau trong những khoảng hở lớn. Với xích lô ở `img_010.jpg`, cần phân biệt khoảng hở thật với các lỗ lốm đốm do mask thiếu phần ghế hoặc khung.
- **Quy tắc 3, bóng đổ.** Ở `img_003.jpg`, vùng bóng đổ trên mặt đường bên trái xe nằm ngoài mask, phù hợp với quy tắc loại bóng đổ khỏi vật.
- **Quy tắc 4, hai vật chạm nhau.** Mỗi vật có polygon riêng, vùng chạm thuộc vật phía trước. Người lái dính vào tuk-tuk ở `img_013.jpg` và đứa bé, người đàn ông dính vào xe ở `img_003.jpg` là những vùng cần tách. Các mảng dính trên áo người đạp ở `img_010.jpg` cũng cần loại khỏi mask xe. Click âm đã đặt vào người nhưng kết quả vẫn còn vùng dính, nên cần kiểm tra lại mask sau click.

Mục 4.2 đã có quy tắc chung về hai vật chạm nhau, nhưng chưa cụ thể hoá cách xử lý người lái ngồi trong khoang xe, cơ thể bị khung xe che và phần hàng hoá đi cùng xe. Quy tắc về hàng nhô ra hơn khoảng 20% chiều dài xe nằm ở mục 4.1 cho box. Phạm vi áp dụng cho mask cần được chốt thêm trong Part 6. Tôi ghi nhận `img_013.jpg` cho ca người lái và khoảng hở qua khung, `img_003.jpg` cho ca người và hàng hoá trên xe bán hàng, `img_010.jpg` cho ca người đạp bị khung che và mask nhiều mảnh. Các ca này cần thống nhất quy tắc trong calibration v1.1.

## 6. Calibration (Part 6)

Có **10 quyết định** trong `submission/qc/calibration_decisions.csv`, gồm 3 dòng nguồn `loi_may` và 7 dòng nguồn `cau_hoi_mo`. Mỗi dòng dẫn ít nhất một ảnh và đều được đưa vào `submission/guidelines/guideline_v1.1_calibrated.md`.

CAL-02 làm thay đổi nhãn trong bản release hiện tại. Ba xe đẩy tay ở img_008 có một trục hai bánh, không có bộ phận đạp hoặc động cơ, nên thuộc nhóm vật ngoài taxonomy. Đã xóa ba box three_wheeler 109, 110 và 111. CAL-03 chốt giá gắn xe ở img_009, giữ motorbike 118 và bỏ can_review. CAL-09 chốt người bị mui che ở img_035, giữ person 573 và bỏ can_review. Các ID này thuộc bản ảnh QC trước calibration.

Đã áp dụng bản sửa lên CVAT task 25 qua API, lưu và đọc lại để xác nhận 585 box. Sau đó xuất COCO 1.0 từ CVAT với include_images=False thành `submission/labels/qc_fixed.json`. Bản xuất đủ 24 ảnh, khớp lớp, tọa độ, conf và can_review với bản sửa, đã qua validate_labels. Đã chạy lại build_release. Manifest xác nhận thay đủ 24 ảnh từ qc_fixed, còn 585 đối tượng, trong đó 20 three_wheeler và 8 ignore_region. Nội dung release đã được đối chiếu khớp bản export CVAT. Bản dùng lấy mẫu QC của phiên này là release trước calibration, có 588 đối tượng và đã được lưu đối chiếu ở `work/qc/qc_baseline.coco.json`.

CAL-01 và CAL-08 liên quan lỗi lớp của máy, gồm tuk-tuk thành car và xe đẩy thành three_wheeler dù điểm cao. CAL-06 là lỗi thực hiện của người review khi để lọt ba box lỏng, tiêu chuẩn bám biên đã có trong v1.0. Các ca còn lại cụ thể hoá phần guideline mơ hồ về xe đẩy, hàng gắn xe, người bị che, xe điện, mask nhiều mảnh và ảnh đêm. CAL-04 dùng img_013, img_010 và img_003 để chốt việc tách người khỏi mask xe, loại lỗ hở lớn và giữ các mảnh cùng một vật. Ba ảnh Part 5 lưu kết quả thử và lỗi quan sát, các mask đó cần chỉnh thêm nếu được dùng làm nhãn mask cuối.

## 7. QC và quyết định (Part 6)

Mẫu QC hiện tại gồm **8/24 ảnh batch**, lấy 3 ảnh có nhiều can_review nhất và 5 ảnh ngẫu nhiên trong phần còn lại, seed 42. Mẫu có **224 đối tượng được gán nhãn**, đã loại ignore_region khỏi mẫu số theo `qc_sample.py`. Người chịu trách nhiệm phiếu là Đỗ Phương Mạnh. Tôi vừa tham gia review vừa kiểm QC, nên đánh giá có thể lặp lại cùng cách nhìn và bỏ sót trước đó.

Đã mở đủ 8 ảnh trong `work/qc/review/`, đối chiếu ảnh gốc và ghi từng lỗi theo ID trong `submission/qc/qc_review.csv`. Các ảnh QC và ID trong phiếu thuộc bản trước calibration, nguồn đối chiếu được lưu ở `work/qc/qc_baseline.coco.json`. Vật thiếu được đặt mã MISS theo ảnh vì chưa có ID trên bản nhãn. Phiếu có 11 dòng lỗi và một dòng khong cho img_004.

| Chỉ số của bản trước sửa calibration | Số và mẫu số | Tỉ lệ |
|---|---|---|
| Ảnh có ít nhất một lỗi | 7/8 ảnh mẫu | 87,5% |
| Đối tượng hoặc vị trí vật bị ghi lỗi | 11/224 đối tượng đã gán trong mẫu | 4,9% |
| Bắt bẫy sau lượt review | 8/12 bẫy | 66,7% |

Mẫu số 224 là số đối tượng đã gán trong mẫu theo qc_sample.csv. Hai dòng bo_sot ghi vị trí vật chưa có box, nên tỉ lệ 11/224 là cách đếm theo phiếu QC và cần đọc cùng danh sách lỗi.

| Loại lỗi | Số dòng lỗi trên tổng 11 | Ảnh ví dụ |
|---|---|---|
| thua | 4/11 = 36,4% | img_003 box 3 trên đồ treo, img_008 box 109–111 trên xe đẩy tay |
| sai_vi_tri_kich_thuoc | 4/11 = 36,4% | img_009 box 115, img_017 box 162, img_018 box 216, img_024 box 338 |
| bo_sot | 2/11 = 18,2% | img_018 xe đạp người áo đỏ, img_024 xe máy cạnh minibus |
| sai_lop | 1/11 = 9,1% | img_035 box 565, xe đạp đang gán motorbike |

Hai nhóm lỗi nhiều nhất cùng có 4/11 dòng lỗi, nhóm tiếp theo có 2/11. Ví dụ cụ thể và tọa độ nằm trong qc_review.csv. Kết quả đã đối chiếu khớp submission/qc/qc_report_auto.md, gồm 7/8 ảnh lỗi, 11/224 đối tượng hoặc vị trí vật bị ghi lỗi và cận trên Wilson 95% là 98%.

### Trạng thái sau vòng sửa calibration

Đối chiếu `qc_fixed.json` với `batch_reviewed.json` xác nhận thay đổi lớp hoặc hình học tập trung ở `img_008.jpg`, nơi đã xóa ba box xe đẩy. Vòng này xử lý **3/11 = 27,3% lỗi QC đã ghi**, còn **8/11 = 72,7% lỗi chưa sửa** trong release 585 đối tượng. Hai cập nhật `can_review` ở img_009 box 118 và img_035 box 573 ghi nhận các ca đã chốt quy tắc. Lỗi vị trí ở img_009 box 115 và lỗi lớp ở img_035 box 565 vẫn còn.

| Ảnh | ID trong bản QC trước calibration | Loại lỗi | Trạng thái trong qc_fixed và release |
|---|---|---|---|
| img_003 | 3 | thua | Box person trên đồ treo vẫn còn. |
| img_009 | 115 | sai_vi_tri_kich_thuoc | Tọa độ box người áo tím chưa được chỉnh. |
| img_017 | 162 | sai_vi_tri_kich_thuoc | Box tuk-tuk tiền cảnh còn dư nền. |
| img_018 | 216 | sai_vi_tri_kich_thuoc | Box xe đạp còn nằm quá cao. |
| img_018 | MISS_018_01 | bo_sot | Chưa bổ sung box xe đạp người áo đỏ. |
| img_024 | 338 | sai_vi_tri_kich_thuoc | Box xe máy còn ăn vào mui ô tô. |
| img_024 | MISS_024_01 | bo_sot | Chưa bổ sung box xe máy cạnh minibus. |
| img_035 | 565 | sai_lop | Box xe đạp vẫn mang lớp motorbike. |

Các ID phía trên dùng để tìm lỗi trên ảnh QC trước calibration, vì export và build lại có thể đánh số đối tượng lại. Các tỉ lệ 7/8 ảnh và 11/224 đối tượng trong qc_report_auto.md là số đo của lượt QC trước sửa. Cần một lượt QC sau rework để có số đo chất lượng cập nhật.

Với 7 ảnh có lỗi trên 8 ảnh mẫu, cận trên Wilson 95% của tỉ lệ ảnh lỗi khoảng **98%**, tính từ k=7 và n=8 theo công thức trong qc_report.py. Mẫu có ba ca khó được chọn có chủ đích, nên tỉ lệ 7/8 phản ánh mẫu kiểm hiện tại. Cỡ mẫu nhỏ và cách chọn này chưa đủ chứng minh toàn lô đạt ngưỡng ảnh lỗi 2%.

Script đếm lỗi theo phiếu, ghép box bằng lớp và IoU, kiểm cấu trúc dữ liệu và tính tỉ lệ. Độ bám biên vật, vật chưa có box, sự mơ hồ về lớp và lỗi người với xe dính trong mask được nhận biết bằng quan sát ảnh. Kiểm tra bản xuất sau Save xác nhận nội dung lưu khớp, phần QC vẫn cần đánh giá chất lượng thực tế.

**Quyết định là Rework, ngày 18/11/2026**, khớp gợi ý trong qc_report_auto.md. Lượt QC trước sửa có tỉ lệ ảnh lỗi 7/8 = 87,5% vượt ngưỡng 2% và bắt bẫy 8/12 = 66,7% thấp hơn yêu cầu ít nhất 10/12. CAL-02, CAL-03 và CAL-09 đã được áp dụng trong CVAT và có qc_fixed.json. Release đã được build lại đủ 24 ảnh và 585 đối tượng. Vòng sửa này mới xử lý 3/11 lỗi QC ở img_008, còn 8/11 lỗi trong bảng trạng thái phía trên cần sửa tiếp. Cần rà bốn cạnh trên các ảnh 005, 021, 028, rồi QC lại và cập nhật số đo trên bản release mới trước khi cân nhắc Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài.** YOLO26 và YOLOE tạo gợi ý box, SAM 2.1 tạo mask. 
- **Gold.** Theo xác nhận của tôi, trước khi gán xong 4 ảnh gold tôi chưa xem pre-label hoặc hỏi AI. time_log ghi 30 phút cho 4 ảnh. Các số đo so với gold dùng 61 đối tượng theo review_stats, log gán tay ghi 64 box.
- **Máy và CVAT.** Windows 11 Pro, RAM 16 GB, chạy CPU, không GPU. Môi trường ghi trong STUDENT là torch 2.15.0+cpu và CVAT Docker v2.81.0. Sáu lượt r01–r06 trong work/runs.csv đều ghi device cpu.
- **Nguồn dữ liệu chạy model.** Tôi tự chạy r05 và r06 trên máy, đã xác nhận lại khi hoàn thiện báo cáo. Các tệp dùng để gộp nằm ở work/prelabel/y26_all và work/prelabel/yoloe_all.
- **Vướng mắc.** Trong phiên hỗ trợ trước, time_log.csv bị Excel giữ khóa ghi, đã lưu và đóng riêng workbook để cập nhật ghi chú. Ở bước calibration, công cụ điều khiển giao diện CVAT lỗi khởi tạo kernel assets. Sau khi cung cấp token truy cập, bản sửa được áp dụng qua API và xuất COCO từ CVAT, có kiểm tra đối chiếu sau lưu.
- **Điều làm khác lần sau.** Dành một lượt kiểm hình học riêng sau khi sửa lớp và tìm vật sót. Ghi checklist bốn cạnh theo box, đặc biệt với ảnh đông xe và ảnh đêm. Nhờ một người khác kiểm mẫu nếu có thể để giảm việc lặp lại cùng lỗi.
