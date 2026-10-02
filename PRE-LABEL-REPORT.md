# Báo cáo thực hành PointPillars — Day 13

## 1. Thông tin nhóm và môi trường

- Nhóm: **727**; phòng: **C402**; ngày thực hành: **01/10/2026**.
- Danh sách thành viên và MSSV: xem `TEAMMATES.md`; báo cáo dùng mã TV01–TV04 để đối chiếu.
- Phân công: TV01 phụ trách tổng hợp; các thành viên cùng rà soát kết quả và nhận xét. Phân công riêng và việc đổi vai giữa A/B/C không được ghi lại.
- Nguồn thực thi: lệnh được chạy bằng công cụ hỗ trợ tự động trên máy nhóm trưởng; nhóm phân tích kết quả đã lưu, ghi trạng thái **`provided-results`**.
- Thời gian chạy: 15:41–15:42 ngày 01/10/2026 (UTC+7).
- Máy: Windows x64, Docker Linux amd64; giới hạn container 4 CPU và 4 GiB RAM.
- Bundle: `student-prelabel-v1`, `student-prelabel-amd64.zip`; SHA256: `f58ca33705fc9c6845ea56a7a5deee9bd2f95b3db2deb32e73f46d82a9737aa9`.
- Image: `day13-pointpillars:lc-20261001-amd64`; ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`.
- Revision nguồn trong bundle: `0831856d921609312d42c7582c366e5a311bb7b1`, `working_tree_dirty: true`. Tài liệu lab dùng bản cập nhật `e226b934c656f23c0da70b1e12cbf365fb78dd82`.
- Input: KITTI demo, `frame_id=demo`; SHA256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`.
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`; SHA256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Cấu hình chung: checkpoint KITTI, score threshold 0.3, front ROI; `z_ground=0.075 m`.
- Output: `output-20261001-154114/`; `smoke.json` có trạng thái **passed**.

PCD Student không giữ reflectance thật; RGB=0 là giá trị thay thế. Adapter KITTI dùng kênh hằng 0.0 cho lượt giữ `vehicles` và 0.7 cho lượt giữ `pedestrian`/`two-wheels`. `z_ground` lấy từ tâm bin đông điểm nhất của histogram z, với bin rộng 0.05 m. Đây là ước lượng từ scan, không phải mặt đường chính xác ở mọi vị trí. Nguồn: `practice/preannotate.py` trong bundle và `data/ATTRIBUTION.md`.

## 2. Kết quả A/B/C

| Lượt | delta (m) | Pillar XY (m) | Số hộp | mean_z (m) | Phân bố class |
| --- | ---: | ---: | ---: | ---: | --- |
| A | 0 | 0.16 | 1 | 0.330 | 1 vehicles |
| B | 1.73 | 0.16 | 13 | 1.034 | 10 vehicles, 2 pedestrian, 1 two-wheels |
| C | 1.73 | 0.32 | 6 | 1.091 | 6 pedestrian |

Bằng chứng trong thư mục output:

- A: `run-A/summary.csv`, `boxes-demo-delta-0-voxel-0.16.json`, `side-demo-delta-0-voxel-0.16.png`.
- B: `run-B/summary.csv`, `boxes-demo-delta-1.73-voxel-0.16.json`, `side-demo-delta-1.73-voxel-0.16.png`.
- C: `run-C/summary.csv`, `boxes-demo-delta-1.73-voxel-0.32.json`, `side-demo-delta-1.73-voxel-0.32.png`.

**A/B:** Khi chỉ đổi delta từ 0 lên 1.73 m, số hộp tăng từ 1 lên 13 và mean_z thay đổi từ 0.330 lên 1.034 m. Model suy luận lại trên input đã dịch nên tập dự đoán có thể đổi; không thể xem đây là dịch cùng một hằng số lên các hộp cũ.

**B/C:** Khi giữ delta và tăng cạnh pillar từ 0.16 lên 0.32 m, số hộp giảm từ 13 xuống 6, phân bố class cũng thay đổi. Pillar là ô gom điểm, không phải kích thước cuboid. C dùng lại checkpoint, không được train riêng cho pillar lớn hơn.

Số hộp, score và mean_z không chứng minh cấu hình nào tốt hơn. Mẫu KITTI không có ground truth hoặc camera để xác nhận chất lượng. Model chỉ xét front ROI; ảnh Side là hình chiếu x-z có thể chồng các đối tượng khác y, nên không đủ để chốt class hoặc yaw. JSON KITTI và ca lỗi chỉ dùng phân tích, không import vào Robotaxi.

## 3. Phép đổi tọa độ và QC pipeline

```text
z_model  = z_source - z_ground - delta
z_source = z_model  + z_ground + delta
```

Với B, lượng bù ngược là `0.075 + 1.73 = 1.805 m`. JSON đã ở hệ nguồn; không cộng bù thêm lần nữa. Đường z=0 của plot không thay cho mặt đường cục bộ.

Helper tạo ba ca từ 13 hộp của B; đây là biến đổi có kiểm soát, không phải inference bổ sung hoặc ground truth.

| Ca | Hộp lệch z / tổng | Độ lệch | Trường giữ nguyên | Quyết định |
| --- | ---: | --- | --- | --- |
| case-correct | 0/13 | 0 m | Class, x/y, dimensions, yaw | Bản đối chiếu giữ prediction B; chưa chứng nhận cuboid đúng. |
| case-batch-z | 13/13 | Xuống 1.805 m mỗi hộp | Class, x/y, dimensions, yaw | Dừng sửa tay; báo LC kiểm transform/pipeline và tạo lại prediction. |
| case-one-box-z | 1/13 | Một hộp xuống 1.805 m | Class, x/y, dimensions, yaw | Kiểm riêng hộp qua nhiều góc nhìn; sửa khi có bằng chứng. |

Bằng chứng: `qc-cases/manifest.json`, `case-correct.json`, `case-batch-z.json`, `case-one-box-z.json` và các ảnh `side-correct.png`, `side-batch-z.png`, `side-one-box-z.png`. Hộp bị dịch riêng là hộp đầu trong JSON, gần x=8.09 m, y=1.21 m.

## 4. Quan sát khi sửa Robotaxi trên CVAT

Nhóm gặp vùng cây bị gán `pedestrian`, hộp không có đối tượng tương ứng rõ ràng, hộp xe chìm dưới mặt đường, sai kích thước và hướng. Trong phần đã xem, nhóm ước lượng khoảng một nửa pre-label cần sửa hoặc xóa. Đây là cảm nhận trong lúc rà, chưa có phép đếm và mẫu số để tính tỷ lệ FP/FN.

| Thành viên | Job nguồn đã nộp v1 | Peer QC | Phản hồi/v2 |
| --- | ---: | --- | --- |
| TV01 | 8 | Chưa ghi số lượt | Chưa có v2 ở lần cập nhật cuối |
| TV02 | 10 | Đã nộp feedback | Chờ QC, chưa nhận feedback |
| TV03 | 6 | Đã nộp feedback | Đã bổ sung hộp xe máy, Save và nộp v2 cho bài được nhận xét |
| TV04 | 30 | Đã nộp feedback | Đã chỉnh width xe buýt, Save và nộp v2 cho bài được nhận xét |

Tổng 54 job v1. Bảng tổng hợp từ ghi nhận của các thành viên; chưa có tổng số lượt QC/v2 để thống kê. Bài đang chờ reviewer không được tính là đã done.

## 5. Nhận xét cá nhân

### TV01

Tôi phụ trách tổng hợp và đã nộp 8 job nguồn v1. Nhóm gặp nhiều lỗi class, hộp thừa, kích thước và hướng. Điều này cho thấy cần đối chiếu pre-label với point cloud và camera thay vì giữ nguyên dự đoán.

Trong A/B, số hộp thay đổi từ 1 lên 13 khi đổi delta; B/C giảm từ 13 xuống 6 khi đổi pillar. Đây là thay đổi dự đoán, chưa chứng minh chất lượng tốt hơn. Khi đưa điểm vào model cần trừ z_ground và delta, rồi cộng lại để trả hộp về hệ nguồn. Với ca batch, cả 13 hộp giảm 1.805 m nên cần dừng chỉnh tay và kiểm pipeline. Nếu chỉ một hộp lệch, cần xem riêng đối tượng.

Điều chưa chắc là chất lượng thực của từng cấu hình vì chỉ có một scan và không có ground truth. Ước lượng khoảng 50% pre-label cần sửa/xóa cũng chưa được đo có hệ thống.

### TV02

Tôi rà pre-label bằng CVAT và camera, đã nộp 10 job nguồn v1. Ở job 3105, một hộp vehicles nằm trong vùng đường trống; camera không cho thấy xe tương ứng và PCD chỉ có ít điểm. Sau khi đối chiếu tôi đã xóa hộp. Tôi cũng nộp feedback về hộp gộp nhiều người, đề nghị kiểm lại và tách cuboid cho từng người xác định được. Bài nguồn của tôi vẫn đang chờ QC.

Từ `run-A/summary.csv` và `run-B/summary.csv`, A có 1 hộp còn B có 13. Đổi delta làm model chạy lại trên input khác, không chỉ dịch các hộp cũ. Phép z thuận là trừ z_ground và delta; phép ngược là cộng lại, với B bằng 1.805 m. Nếu tất cả hộp cùng lệch như case-batch-z thì cần kiểm pipeline; một hộp thừa riêng lẻ không đủ để kết luận lỗi transform.

Tôi còn chưa rõ ranh giới cuboid với xe hai bánh chở hàng cồng kềnh, nhất là có bao gồm hàng và người hay không. Trường hợp này cần quy ước của LC và bằng chứng từng đối tượng.

### TV03

Tôi đối chiếu point cloud và camera, thảo luận kích thước từng xe, đã nộp 6 job nguồn v1. Một hộp ô tô có width rộng hơn thân xe và đáy thấp hơn mặt đường khoảng 0.2 m theo ước lượng của tôi. Tôi dùng Top-view và camera trước để đối chiếu thân xe, nâng hộp theo z và thu hẹp width; không còn nhớ job ID. Kiểm đáy còn cần góc Bên và mặt đường gần đối tượng.

Tôi đã nộp feedback về cột điện bị gán pedestrian. Sau feedback về xe máy bị bỏ sót gần gờ tường, tôi đã thêm cuboid, Save và nộp v2.

B/C có 13 và 6 hộp khi chỉ đổi pillar 0.16 lên 0.32 m. Kích thước pillar khác kích thước hộp xe, và số hộp ít hơn chưa chứng minh kết quả tốt hơn. Phép z đưa điểm về hệ model bằng cách trừ z_ground và delta rồi cộng lại khi xuất. Nếu quên cộng ngược ở ca B, cả batch thấp hơn 1.805 m; phải kiểm pipeline thay vì nâng từng hộp. Với xe trên dốc hoặc gờ giảm tốc, tôi còn thấy thao tác góc nghiêng khó và cần xác nhận cách xử lý pitch/roll của ca.

### TV04

Tôi rà cuboid bằng không gian 3D và camera, đã nộp 30 job nguồn v1. Ở job 3012, một hộp pedestrian nằm trên vùng tán cây. Side-view và camera trước không cho thấy người tại đó nên tôi đã xóa cuboid. Đây là quan sát một trường hợp; không đủ để kết luận lỗi toàn bộ class.

Tôi đã nộp feedback về height của hộp two-wheels ở xa và đề nghị kiểm phần đối tượng bị bỏ ngoài hộp. Sau feedback về hộp xe buýt bao cả điểm nền, tôi đã chỉnh width, Save và nộp v2.

JSON B có 10 vehicles, 2 pedestrian, 1 two-wheels, còn C có 6 pedestrian. Class dự đoán thay đổi khi tăng pillar nhưng không thể dùng số này làm số đối tượng thật. Phép z thuận/ngược trừ rồi cộng z_ground và delta; với B là 1.805 m. Ca batch có 13 hộp lệch cùng lượng, còn ca one-box chỉ có một hộp lệch: trường hợp đầu cần dừng và kiểm pipeline, trường hợp sau kiểm riêng qua nhiều view.

Tôi còn khó xác định ranh giới cuboid của xe hai bánh có rider so với xe đỗ bị che một phần. Cần xác nhận quy ước bao rider với LC và không lấy height cố định cho mọi xe.

## 6. Tổng kết

Nhóm có đủ output A/B/C và ba ca QC để đối chiếu, giải thích được tác động của delta/pillar và cách phân biệt lỗi batch với lỗi từng hộp. Phần Robotaxi được sửa và QC trong hệ thống; output KITTI không được dùng thay cho nhãn Robotaxi.

LC ghi nhận ngày 02/10/2026: **ĐẠT**, đồng ý chuyển sang chỉnh/QC và không yêu cầu chạy lại. Theo góp ý, tên và MSSV được chuyển sang `TEAMMATES.md`. Trong lần thực hành tiếp theo, nhóm cần ghi vai trò từng lượt A/B/C, đổi vai và tự tay chạy lệnh.

Deadline của ca là 12:00 ngày 02/10/2026 (UTC+7). Theo Q&A của LC, QC được giao ngẫu nhiên và có thể đến muộn; khi chưa có feedback thì hoàn thiện phần hiện tại, khi có feedback thì đối chiếu, Save và nộp v2 theo hạn portal.

## LC ghi nhận riêng

> LC ghi nhận ngày 02/10/2026. **Kết luận: ĐẠT.**

- **Quyền dùng PCD/image và đúng ca:** Gói Student KITTI 000008 (giấy phép CC BY-NC-SA 3.0), không dùng dữ liệu Robotaxi. SHA-256 gói `f58ca337…` khớp file kiểm tra chính thức trên Release; input `3b5ea3da…` và image `sha256:e03983bd…` (amd64) khớp `smoke.json`.
- **Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:** Có lượt chạy thật trên máy trưởng nhóm (`smoke.json` passed, 15:41–15:42, 1/13/6, không trùng nhóm nào). Lệnh do công cụ hỗ trợ tự động chạy, nhóm ghi `provided-results`. LC không yêu cầu chạy lại.
- **Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:** Đủ `run-A/B/C` và `qc-cases`, không đổi so với lần 1. Báo cáo ghi rõ JSON KITTI và ca lỗi không import vào Robotaxi; JSON đã ở hệ nguồn, không cộng bù thêm.
- **Nhận xét từng thành viên và quyết định dừng pipeline:** Báo cáo trình bày lại gọn, rõ; mọi số liệu đúng (phân bố lớp A/B/C, 3 ca lỗi 0/13, 13/13 −1,805 m, 1/13 ở hộp đầu x≈8,09). Mỗi người có quan sát CVAT Robotaxi cụ thể (job 3105, job 3012, cột điện gán `pedestrian`, hộp xe chìm ~0,2 m), phép z thuận/ngược, quyết định batch/one-box và điều chưa chắc.
- **Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:** **Đồng ý.**

**Nên sửa (không chặn):**
1. Lần sau ghi vai trò từng lượt A/B/C, đổi vai giữa các lượt và tự tay chạy lệnh.
2. Chuyển họ tên, MSSV từ báo cáo sang `TEAMMATES.md`.
