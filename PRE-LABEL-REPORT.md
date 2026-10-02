# Báo cáo thực hành PointPillars — Day 13 — Nhóm 727

**Bản riêng đã được nhóm rà soát theo xác nhận của trưởng nhóm ngày 2026-10-02; chờ đích nộp và ghi nhận của LC.** Không commit hoặc gửi bản có họ tên/MSSV lên GitHub hay nhóm chat chung. Report này dùng kết quả KITTI minh họa; không phải kết quả nhãn Robotaxi và không được import vào CVAT.

## Nhóm và nguồn kết quả

- Mã nhóm: **727**; phòng/ca: **C402**.
- Thành viên: Phạm Hoàng Anh (02128, nhóm trưởng); Bạch Khánh An (02095); Đỗ Lý Minh Hải (02173); Lê Đức Mạnh (02122).
- Phân công A/B/C: Nhóm cùng rà soát nội dung báo cáo và nhận xét cá nhân khi hoàn thiện. Không còn ghi chép phân công riêng theo lượt A/B/C hoặc lịch đổi vai, nên không tái dựng người vận hành/đọc JSON/xem hình/ghi log theo từng lượt. Lệnh thực tế do Codex thực hiện theo yêu cầu của trưởng nhóm. Việc duyệt báo cáo không thay thế bằng chứng đã luân phiên vận hành tại lớp; phần này được trình bày để LC quyết định có cần bổ sung thực hành hay không.
- Trạng thái theo hướng dẫn LC: **`provided-results`**. Codex đã chạy lệnh trên máy của nhóm trưởng theo yêu cầu của trưởng nhóm; chưa có bằng chứng rằng cả nhóm tự vận hành lệnh. Nếu LC có quy tắc phân loại khác cho lần chạy được hỗ trợ, hỏi LC trước khi đổi trạng thái.
- Người/thời điểm chạy: Codex theo yêu cầu của Phạm Hoàng Anh; 2026-10-01 08:41–08:42 UTC (15:41–15:42 giờ Việt Nam).
- Máy thực thi: máy Windows x64 của trưởng nhóm; Docker Linux `amd64`; container giới hạn 4 CPU và 4 GiB RAM.
- Bundle: release `student-prelabel-v1`, `student-prelabel-amd64.zip`; SHA256 `f58ca33705fc9c6845ea56a7a5deee9bd2f95b3db2deb32e73f46d82a9737aa9`.
- Image: `day13-pointpillars:lc-20261001-amd64`; ID `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`.
- Runner ghi revision `0831856d921609312d42c7582c366e5a311bb7b1` và `working_tree_dirty: true`; checkout Student hiện tại là `e226b934c656f23c0da70b1e12cbf365fb78dd82`. Giữ cả hai giá trị theo nguồn log, không gộp chúng.
- PCD/frame: dữ liệu KITTI demo, `frame_id=demo`; input SHA256 `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`. `smoke.json`: **passed**.
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`; SHA256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Giữ nguyên giữa các lượt: checkpoint KITTI, score threshold **0.3**, front ROI; `z_ground=0.075 m` theo JSON và manifest QC.
- Giả định intensity/kênh thứ tư (Codex đối chiếu code bundle ngày 2026-10-02): PCD không giữ reflectance thật; RGB=0 chỉ là placeholder. Preset KITTI dùng kênh hằng 0.0 cho lượt giữ `vehicles`, và 0.7 cho lượt giữ `pedestrian`/`two-wheels`. Đây không phải intensity đo được hoặc được phục hồi. `estimate_ground` ước lượng mặt đất từ tâm bin z đông điểm nhất, bin rộng 0.05 m; không chứng nhận mặt đường cục bộ mọi vị trí. Nguồn: `student-amd64/practice/preannotate.py`, `PRESETS` và `estimate_ground`, cùng `data/ATTRIBUTION.md`. Nhóm trước đó trả lời chưa rõ; các thành viên cần đọc và hiểu phần bổ sung này.
- Output đầy đủ: `output-20261001-154114/` trong thư mục riêng của nhóm; gồm `smoke.json`, ba thư mục run và ba ca QC có kiểm soát.

## Ba lượt inference

| Lượt | delta (m) | Pillar XY (m) | Số hộp | mean_z (m) | Bằng chứng | Quan sát từ output |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `run-A/summary.csv`, `run-A/side-demo-delta-0-voxel-0.16.png`, `run-A/boxes-demo-delta-0-voxel-0.16.json` | JSON có 1 hộp `vehicles`; hình Side cho thấy một cuboid dự đoán. |
| B | 1.73 | 0.16 | 13 | 1.034 | `run-B/summary.csv`, `run-B/side-demo-delta-1.73-voxel-0.16.png`, `run-B/boxes-demo-delta-1.73-voxel-0.16.json` | Log runner ghi 10 `vehicles`, 2 `pedestrian`, 1 `two-wheels`; hình Side hiển thị 13 cuboid. Đây là dự đoán, chưa được xác nhận đúng theo ground truth. |
| C | 1.73 | 0.32 | 6 | 1.091 | `run-C/summary.csv`, `run-C/side-demo-delta-1.73-voxel-0.32.png`, `run-C/boxes-demo-delta-1.73-voxel-0.32.json` | Log runner ghi 6 `pedestrian`; hình Side hiển thị 6 cuboid. Đây là dự đoán, chưa được xác nhận đúng theo ground truth. |

**So sánh A/B.** Chỉ đổi `delta`: kết quả đổi từ 1 sang 13 hộp; `mean_z` đổi từ 0.330 m sang 1.034 m. `delta` dịch điểm dữ liệu đầu vào trước khi suy luận, nên model có thể tạo tập hộp khác; đây không phải phép cộng/trừ một hằng số lên các hộp đã dự đoán. Khác biệt số hộp là bằng chứng các dự đoán thay đổi, nhưng chưa cho biết cấu hình nào chính xác hơn.

**So sánh B/C.** Chỉ đổi pillar từ 0.16 m sang 0.32 m; số hộp giảm từ 13 xuống 6 và `mean_z` đổi từ 1.034 m sang 1.091 m. Thay đổi độ phân giải lưới ảnh hưởng đầu vào model. Không đủ bằng chứng để kết luận A/B/C tốt hơn: đây là một PCD KITTI, không có đánh giá đối chiếu ground truth; số hộp và `mean_z` không phải metric chất lượng.

**Giới hạn cách đọc.** Model chỉ xét front ROI. Ảnh Side là hình chiếu, nên đối tượng ở các vị trí khác nhau có thể chồng nhau; ảnh này không đủ để xác nhận class, vị trí 3D hay yaw. Không phát hiện hộp ngoài ROI cũng không có nghĩa là không có đối tượng ngoài ROI. Cần đối chiếu JSON/PCD và các góc nhìn theo hướng dẫn, đồng thời ghi rõ phạm vi.

**Khả năng dùng JSON.** Không JSON nào trong output KITTI được import vào job Robotaxi. Chúng thuộc dữ liệu KITTI/frame demo; các ca `qc-cases` chỉ là biến đổi huấn luyện từ dự đoán B, không phải nhãn đúng. Robotaxi pre-label phải được nạp từ portal vào đúng job/frame theo hướng dẫn LC.

## Quan sát CVAT do trưởng nhóm cung cấp

Trong phần Robotaxi pre-label nhóm đã rà, trưởng nhóm cho biết có trường hợp vùng cây bị gán nhầm `pedestrian`, vật thể chưa đủ rõ vẫn được gán nhãn, hộp xe nằm thấp/chìm dưới mặt đường, và cuboid sai kích thước hoặc hướng. Nhóm ước lượng bằng cảm nhận rằng khoảng **một nửa pre-label trong phần đã xem cần sửa hoặc xóa**; nhóm không đếm có hệ thống nên đây không phải tỷ lệ chính xác hay thống kê toàn bộ dữ liệu.

| Thành viên | Job nguồn đã nộp v1 vào QC (số trưởng nhóm báo) | QC đã làm/feedback | v2 đã phản hồi? |
| --- | ---: | --- | --- |
| Phạm Hoàng Anh — 02128 | 8 | Chưa ghi tổng số/lượt cụ thể | Chưa có v2 theo cập nhật của trưởng nhóm |
| Đỗ Lý Minh Hải — 02173 | 6 | Tự báo đã nộp toàn bộ feedback; chưa có tổng số lượt | Tự báo đã bổ sung hộp xe máy, Save và nộp v2 cho bài nhận feedback; chưa có tổng số |
| Lê Đức Mạnh — 02122 | 30 | Tự báo đã nộp feedback; chưa có tổng số lượt | Tự báo đã sửa width xe buýt, Save và nộp v2 cho bài nhận feedback; chưa có tổng số |
| Bạch Khánh An — 02095 | 10 | Tự báo đã nộp toàn bộ feedback; chưa có tổng số lượt | Tự báo đang chờ QC, chưa nhận feedback |

## Ba ca QC có kiểm soát — chỉ để luyện, tuyệt đối không import CVAT

Nguồn là 13 hộp của lượt B. Manifest ghi `delta=1.73 m`, `z_ground=0.075 m`; helper dịch z có chủ đích `1.805 m` (`delta + z_ground`). Các trường class, x/y, kích thước và yaw không đổi trong những ca bị dịch.

| Ca | Số hộp z bị lệch / tổng | Độ lệch | Class/x/y/kích thước/yaw | Hành động | Bằng chứng |
| --- | ---: | ---: | --- | --- | --- |
| `case-correct` | 0/13 | 0 m | Không đổi; bản sao prediction B | Bản đối chiếu của helper, không phải ground truth hay nhãn đúng. | `qc-cases/case-correct.json`, `qc-cases/side-correct.png` |
| `case-batch-z` | 13/13 | Mỗi hộp xuống `1.805 m` | Không đổi | Dừng sửa tay từng hộp, báo LC kiểm pipeline vì cả loạt cùng lệch. | `qc-cases/case-batch-z.json`, `qc-cases/side-batch-z.png`; manifest mô tả shift `delta + z_ground`. |
| `case-one-box-z` | 1/13 | Một hộp xuống `1.805 m`; hộp đầu trong JSON, tâm gần x=8.09 m, y=1.21 m | Không đổi | Kiểm hộp này qua nhiều góc nhìn; chỉ sửa riêng nếu có bằng chứng. | `qc-cases/case-one-box-z.json`, `qc-cases/side-one-box-z.png`. |

## Nhận xét cá nhân — nhóm đã rà soát

Các mục dưới đây được biên tập từ lời trưởng nhóm và câu trả lời từng thành viên được trưởng nhóm chuyển ngày 2026-10-02. Trưởng nhóm xác nhận cả nhóm đã rà soát các nhận xét. Quan sát Robotaxi/CVAT được tách khỏi thí nghiệm KITTI A/B/C. Phần “phân tích bổ sung” được Codex soạn từ output đã lưu khi hoàn thiện báo cáo; không phải ghi chép cuộc thảo luận đã diễn ra hoặc bằng chứng từng người đã tự chạy/đọc kết quả tại lớp. Lệnh do Codex chạy theo yêu cầu của trưởng nhóm. Các lời kể về thao tác portal chưa được trợ lý kiểm trực tiếp.

### Phạm Hoàng Anh — 02128 (nhóm trưởng)
- Vai trò thực tế: nhóm trưởng; đã nộp 8 job nguồn v1 vào QC. Codex chạy lệnh A/B/C theo yêu cầu của tôi; tôi không trực tiếp vận hành model.
- Quan sát CVAT: Trong phần nhóm đã rà, có vùng cây bị gán `pedestrian`, có vật thể chưa đủ rõ vẫn được gán nhãn, hộp xe nằm thấp/chìm dưới mặt đường, và cuboid sai kích thước hoặc hướng. Nhóm ước lượng cảm tính khoảng một nửa pre-label cần sửa hoặc xóa; chúng tôi không đếm theo một mẫu thống kê.
- Quan sát A/B/C: Theo `run-A/summary.csv`, A có 1 hộp (`mean_z=0.330 m`); B có 13 (`1.034 m`) khi chỉ đổi `delta` từ 0 lên 1.73 m. Theo `run-C/summary.csv`, C có 6 hộp (`mean_z=1.091 m`) khi đổi pillar từ 0.16 lên 0.32 m so với B. Đây là thay đổi dự đoán, không chứng minh chất lượng tốt hơn.
- Giải thích z: `z_model = z_source - z_ground - delta`; đổi ngược lại `z_source = z_model + z_ground + delta`. Delta đưa input về hệ độ cao model quen; khi đọc output phải cộng lại `z_ground` và `delta`. Với B, cộng lại `0.075 + 1.73 = 1.805 m`.
- Quyết định lỗi cả loạt: Trong ca mô phỏng `case-batch-z`, cả 13/13 hộp lệch xuống 1.805 m trong khi class, x/y, kích thước và yaw giữ nguyên. Tôi sẽ dừng chỉnh tay, báo LC kiểm pipeline; nếu chỉ một hộp lệch như `case-one-box-z`, kiểm hộp đó qua các góc nhìn và sửa riêng khi có bằng chứng.
- Điều còn chưa chắc: Một PCD KITTI và ảnh Side không đủ ground truth để quyết định dự đoán nào đúng; front ROI không cho kết luận về vùng ngoài ROI. Ước lượng 50% của nhóm không có mẫu số và chưa được đo có hệ thống.

### Bạch Khánh An — 02095
- Vai trò thực tế: Tôi rà pre-label Robotaxi trên CVAT kết hợp ảnh camera; đã nộp 10 job nguồn v1 theo số nhóm ghi nhận. Không ghi nhận công việc kiểm tracking: lab dùng từng frame độc lập, CSV A/B/C không chứa ID/chuỗi tracking. Vai trò từng lượt KITTI A/B/C chưa được nhóm nhớ lại.
- Quan sát và sửa nguồn: Trong phần tôi rà có hộp thừa ở khu vực biển báo, gờ tường hoặc ngã tư. Ở job **3105** (ID theo tôi cung cấp), tôi thấy hộp `vehicles` trong vùng đường trống; ảnh camera không cho thấy xe tương ứng và PCD chỉ có ít điểm tại vùng đó. Sau khi đối chiếu, tôi đã xóa hộp này. Đây là nhận xét về một trường hợp, không phải tỷ lệ false positive đã đo.
- QC người khác: Tôi báo đã bấm **Nộp toàn bộ feedback** về một hộp gộp nhiều người đi sát nhau, đề nghị tác giả đối chiếu và tách thành cuboid riêng cho từng người xác định được. Không giữ số frame hoặc số người chính xác làm bằng chứng vì nhóm chưa xác nhận được chi tiết đó. Chưa có ID QC để đối chiếu lại nhận xét.
- Phản hồi/v2: Bài nguồn của tôi đang chờ QC, chưa nhận feedback theo thông tin tôi cung cấp; chưa báo đã nộp v2. Tôi cần theo dõi portal và hạn phản hồi khi feedback đến.
- Điều còn chưa chắc: Với xe hai bánh chở hàng cồng kềnh, tôi chưa rõ ranh giới cuboid có bao gồm hàng và người hay không. Tôi cần LC xác nhận quy ước taxonomy/ràng buộc của ca; chưa tự áp một kích thước chuẩn chung.
- Phân tích KITTI bổ sung từ output: `run-A/summary.csv` ghi 1 hộp, B ghi 13 khi chỉ đổi delta 0 → 1.73 m. Số hộp thay đổi cho thấy model được chạy lại trên input đã dịch, không chỉ dịch hộp cũ. Cần đối chiếu hộp với dữ liệu; số lượng tăng chưa chứng minh phát hiện đúng hơn, và KITTI demo không có ground truth để tính FP/FN.
- Phép z và quyết định batch — phần bổ sung: `z_model = z_source - z_ground - delta`; `z_source = z_model + z_ground + delta`. Với B, lượng bù là 1.805 m. Hộp thừa riêng lẻ không tự chứng minh sai transform. Nếu ca `case-batch-z` có cả 13 hộp cùng giảm 1.805 m trong khi x/y, class và yaw giữ nguyên, cần dừng sửa tay và nhờ LC kiểm pipeline. Ca một hộp cần kiểm riêng; không đưa các ca này vào Robotaxi.

### Đỗ Lý Minh Hải — 02173
- Vai trò thực tế: Tôi kiểm point cloud 3D và các ảnh camera được cấp trên CVAT, thảo luận với nhóm về kích thước cuboid `vehicles`; đã nộp 6 job nguồn v1 theo số nhóm ghi nhận. Tôi đối chiếu hình học của từng đối tượng; chưa có cơ sở gọi kết quả thảo luận là bộ “kích thước chuẩn” áp dụng cho mọi xe. Vai trò trong từng lượt KITTI A/B/C còn cần xác nhận.
- Quan sát và sửa nguồn: Trong phần tôi rà có hộp xe rộng hơn thân xe và đáy nằm thấp hơn mặt đường. Ở một hộp ô tô con, tôi ước lượng đáy thấp khoảng **0.2 m**; đây là ước lượng theo lời kể, chưa có số tọa độ đối chiếu. Tôi dùng Top-view và camera trước để đối chiếu thân xe, nâng hộp theo trục z và thu hẹp width. Job ID không còn nhớ. Lệch vị trí z và sai kích thước là hai vấn đề khác nhau; cần kiểm Bên/góc xoay cùng mặt đường cục bộ để xác nhận đáy, không suy z chỉ từ Top-view.
- QC người khác: Tôi báo đã **Nộp toàn bộ feedback** về một cột điện bị gán `pedestrian`, đề nghị tác giả kiểm lại và xóa hộp nếu xác nhận không có người. “Frame 15” là chi tiết theo lời kể, chưa có job ID/ID QC và góc nhìn cụ thể để tìm lại. Chưa coi nhận xét này là bộ bằng chứng đầy đủ trước khi đối chiếu portal.
- Phản hồi/v2: Tôi nhận feedback về một xe máy bị bỏ sót gần gờ tường, vùng điểm LiDAR bị che. Tôi báo đã thêm cuboid, **Save** và nộp v2. Report ghi hành động đã báo; chưa có file/job ID để trợ lý xác nhận hình học. Đối tượng bị khuất chỉ nên thêm khi camera/PCD cung cấp cơ sở nhận diện và ranh giới.
- Điều còn chưa chắc: Khi xe ở đường dốc hoặc gờ giảm tốc, tôi thấy thao tác chỉnh góc nghiêng khó và dễ làm lệch hộp. Tôi cần xác nhận với LC/editor cách xử lý pitch/roll của ca; chưa khẳng định phải chỉnh pitch/roll cho mọi hộp. Cần kiểm lại tâm, dimensions và orientation sau thao tác; xoay hộp không đồng nghĩa chiều dài thực của xe thay đổi.
- Phân tích KITTI bổ sung từ output: B ghi 13 hộp, `mean_z=1.034 m`; C ghi 6 hộp, `mean_z=1.091 m` trong `summary.csv` khi chỉ tăng cạnh pillar 0.16 → 0.32 m. Pillar là ô gom điểm, không phải chiều rộng cuboid; tăng pillar không đồng nghĩa phải đổi kích thước xe bằng tay. Hai lượt dùng cùng checkpoint và delta; kết quả khác không chứng minh C tốt hơn.
- Phép z và quyết định batch — phần bổ sung: Trừ `z_ground` và `delta` để đưa điểm vào hệ độ cao model rồi cộng lại khi xuất về hệ nguồn: `z_model = z_source - z_ground - delta`; `z_source = z_model + z_ground + delta`. Nếu quên cộng ngược ở ca B, cả 13 hộp thấp hơn 1.805 m. Cần dừng và kiểm pipeline thay vì nâng từng hộp. Một xe có đáy thấp cần kiểm mặt đường cục bộ và nhiều view; không lấy z=0 hoặc 1.805 m làm mức sửa chung cho Robotaxi.

### Lê Đức Mạnh — 02122
- Vai trò thực tế: Tôi rà cuboid Robotaxi bằng không gian 3D CVAT kết hợp ảnh camera; đã nộp 30 job nguồn v1 theo số nhóm ghi nhận. Nhận xét lỗi là định tính; không ghi đã xuất thống kê FP/FN vì không có phép đếm, mẫu số và reference. Vai trò từng lượt KITTI A/B/C chưa được nhóm nhớ lại.
- Quan sát và sửa nguồn: Trong phần tôi rà có vùng cây/tán lá bị gán `pedestrian` và một số hộp không có đối tượng tương ứng rõ ràng ở khu vực ngã tư. Ở job **3012** (ID theo tôi cung cấp), một cuboid `pedestrian` nằm trên vùng tán cây. Tôi đối chiếu Side-view và camera trước, không thấy bằng chứng có người tại đó và đã xóa cuboid. Không kết luận bóng râm gây lỗi LiDAR/model từ quan sát này; cũng chưa có cơ sở xác nhận lỗi hệ thống của toàn class.
- QC người khác: Tôi báo đã nộp feedback về hộp `two-wheels` ở xa có chiều cao chưa bao đủ phần đối tượng mà tôi quan sát, với đề xuất nâng z-max. Lời kể ban đầu khái quát “đều sai” ở khoảng cách >30 m; report không dùng ngưỡng này làm quy tắc chung. Với rider/xe, cần xác nhận quy ước cuboid của LC và bằng chứng từng hộp trước khi quyết định mở rộng chiều cao. Chưa có ID QC/vùng và camera/view cụ thể của lượt đó trong hồ sơ.
- Phản hồi/v2: Tôi nhận feedback yêu cầu thu hẹp width của một hộp xe buýt đang bao cả điểm nền/mặt đường. Tôi báo đã chỉnh, **Save** và nộp v2. Chưa có job ID/tổng số v2 để xác minh mọi nguồn đã hoàn tất; không suy ra 30 v1 đều đã done.
- Điều còn chưa chắc: Tôi còn khó phân biệt ranh giới cuboid của xe hai bánh có rider với xe đỗ bị cây che một phần. Tôi cần hỏi LC quy ước bao rider và dùng PCD/camera để suy phần thân có cơ sở; không xác định height bằng tình trạng máy đang bật/tắt hoặc một hằng số cho mọi xe.
- Phân tích KITTI bổ sung từ output: JSON B có 10 `vehicles`, 2 `pedestrian`, 1 `two-wheels`; JSON C có 6 `pedestrian`. B/C chỉ đổi pillar 0.16 → 0.32 m nên phân bố class của dự đoán thay đổi theo biểu diễn đầu vào. Không dùng số pedestrian của C để kết luận scene có sáu người hoặc class này chính xác. Kinh nghiệm gặp cây bị gán người nhắc rằng class dự đoán cần bằng chứng; KITTI demo không có camera/ground truth để xác nhận các nhãn đó.
- Phép z và quyết định batch — phần bổ sung: `z_model = z_source - z_ground - delta`; `z_source = z_model + z_ground + delta`. Với B, `0.075 + 1.73 = 1.805 m`. Trong `case-batch-z`, cả 13 hộp giảm đúng lượng đó còn class/x/y/dimensions/yaw giữ nguyên: dừng chỉnh từng hộp, báo LC kiểm phép chuyển và tạo lại prediction. Trong `case-one-box-z`, chỉ hộp đầu gần x=8.09 m, y=1.21 m lệch: kiểm riêng bằng nhiều view. Sai class ở cây hoặc một hộp thừa chưa đủ kết luận lỗi batch.

## Tiến độ CVAT/QC và xác nhận LC

**Deadline của ca: 12:00 trưa ngày 2026-10-02 (UTC+7)**, theo trưởng nhóm xác nhận từ Q&A của LC. LC giải thích QC được giao ngẫu nhiên, có thể đến muộn; khi có feedback thì sửa và nộp v2, khi chưa có thì hoàn thiện những phần hiện tại. Bài đang chờ QC được ghi là **đã nộp v1, chờ reviewer; chưa phát sinh phản hồi/v2**, không coi thiếu v2 khi chưa có feedback là việc học viên chưa thực hiện được. Vẫn theo dõi feedback và hạn phản hồi trên portal sau đó. Q&A này không xác nhận nhóm đã nộp báo cáo hoặc mọi bài đã hoàn tất.

Trưởng nhóm xác nhận số trong bảng là job nguồn đã nộp v1 vào QC, tổng 54 job. Cập nhật ngày 2026-10-02: An, Hải và Mạnh báo đã nộp feedback; Hải và Mạnh trước đó báo đã Save/nộp v2 cho tình huống họ mô tả; An đang chờ QC. Hoàng Anh cập nhật chưa có v2; chưa cung cấp tổng số QC đã thực hiện. Chưa có tổng số QC/v2 của nhóm hoặc kiểm tra trực tiếp portal, nên không xác nhận mọi nguồn đều done. Kiểm trạng thái và hạn phản hồi trên portal; bài chờ reviewer ghi đúng đang chờ, không coi là đã nộp v2.

- Phòng: C402; mã ca chi tiết chưa ghi. Quyền dùng KITTI Student theo attribution/license đã giữ trong bundle; Robotaxi được thao tác trong CVAT/portal được cấp.
- Số job nguồn đã nộp v1: Hoàng Anh 8; An 10; Hải 6; Mạnh 30.
- QC/v2: ghi theo lời từng người trong bảng; chưa có số tổng để thống kê.
- Giờ phiên 240 phút từng người: không có ghi chép chính xác; không điền lại giờ ước đoán.
- Trạng thái report: **chưa gửi LC** (theo trưởng nhóm).
- Đích nộp riêng và LC ghi nhận: trưởng nhóm sẽ cung cấp sau; chưa thực hiện trong giai đoạn này.

## LC ghi nhận riêng — để LC xác nhận

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:

**TEAM ACTION REQUIRED:** Khi có feedback, đối chiếu nguồn, sửa có căn cứ, Save và nộp phản hồi/v2 theo hạn portal. Đích nộp và xác nhận LC sẽ được trưởng nhóm cung cấp sau; khi có, nộp bộ report/TEAMMATES/output riêng. LC quyết định phần thực hành còn cần bổ sung và quy ước rider/hàng. Mục LC để LC xác nhận; không tự điền phê duyệt hoặc vai trò chưa ghi nhận.
