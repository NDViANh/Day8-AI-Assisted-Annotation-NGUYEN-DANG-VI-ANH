# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Nguyen Dang Vi Anh

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Camera được đặt cố định nhìn xuống đường cao tốc và ghi liên tục. Mỗi chiếc xe chạy qua sẽ xuất hiện trong nhiều khung hình liền tiếp trong vài giây. Nếu chia dữ liệu ngẫu nhiên, cùng một chiếc xe rất có thể vừa nằm trong tập huấn luyện vừa nằm trong tập kiểm tra — model sẽ "nhớ" xe đó từ lúc train và trả kết quả đúng khi test không phải vì nó thật sự học được cách nhận dạng xe, mà vì nó đã gặp xe đó rồi.

Để tránh hiện tượng rò rỉ dữ liệu (data leakage) này, dữ liệu được chia theo trục thời gian: tập pool (để huấn luyện chủ động) lấy từ giai đoạn đầu, tập kiểm tra lấy từ giai đoạn sau, với một vùng đệm thời gian ở giữa để đảm bảo không có xe nào trong tập train còn xuất hiện trong tập test. Nhờ đó, điểm AP50 trên tập kiểm tra mới phản ánh đúng khả năng tổng quát hóa của model trên dữ liệu chưa từng thấy. Nếu chia ngẫu nhiên, số đo sẽ cao hơn thực tế — lệch theo hướng lạc quan giả tạo.

## 2. Mô hình khởi đầu lạnh (cold start)

Dòng vòng 0 trong `rounds_table.md`:

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n | 0 | 0 | 0.771 | — | 0.812 | 0.734 | 0.771 | 0.182 | 0.547 | 0.561 |

AP50 khởi đầu là 0.771. Nhìn vào `outputs/compare_round0.jpg`, thấy model bỏ sót nhiều xe nhỏ ở xa — đặc biệt những xe chỉ còn hai chấm đèn trắng gần chân cầu. Recall theo kích thước cho thấy xe nhỏ (small) chỉ đạt 0.182, tức là 8 trong 10 xe nhỏ bị bỏ qua hoàn toàn. Xe vừa (medium) và xe lớn (large) tốt hơn nhưng cũng chưa đạt 0.6. Một điểm cần lưu ý: nhãn dùng để chấm điểm là do model khác vẽ sẵn, chưa có người kiểm từng khung thủ công — vì vậy có thể một số trường hợp model của bạn không sai mà nhãn chấm mới thiếu chính xác, đặc biệt với xe nhỏ ở xa.

## 3. Chiến lược chọn mẫu

AI chọn ảnh bằng công thức: `score = W_U·U + W_A·A + W_D·D`, trong đó:
- **U (Uncertainty)**: trọng số 0.5 — đo mức độ AI không chắc chắn. Tính bằng 1 − max_confidence trung bình trên các box. Ảnh có nhiều box mà AI do dự sẽ có U cao.
- **A (Annotation density)**: trọng số 0.3 — đo số box AI đề xuất nhưng còn lưỡng lự (conf thấp). Ảnh đông xe với nhiều box không chắc sẽ có A cao.
- **D (Diversity / time gap)**: trọng số 0.2 — phần thưởng cho ảnh khác về mặt thời gian so với ảnh đã được chọn. `MIN_GAP_S = 2 giây` đảm bảo hai ảnh trong cùng lô phải cách nhau ít nhất 2 giây, vì camera đứng yên nên ảnh sát nhau gần như giống hệt — chọn cả hai tốn công mà không học thêm gì mới.

Ba frame tiêu biểu trong `SELECTION.md`: frame_0182.jpg (điểm 0.959, AI có nhiều box lưỡng lự với đoàn xe đông trên cầu), frame_0099.jpg (điểm 0.901, AI bỏ sót xe bị cắt góc và nhầm vệt đèn), frame_0107.jpg (điểm 0.892, AI khoanh nhầm phản chiếu trên mặt đường). Một ảnh không được chọn dù điểm cao: frame_0372.jpg (0.910) bị loại vì quá gần frame_0369.jpg về thời gian (chỉ 1.2 s). Điểm bất định cao không chứng minh sửa ảnh đó sẽ làm AI giỏi hơn — nó chỉ nói AI đang phân vân, không nói gì về việc phân vân đó có đại diện cho phần lớn dữ liệu không.

## 4. Các vòng học chủ động (active learning)

Bảng so sánh từ `rounds_table.md`:

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n | 0 | 0 | 0.771 | — | 0.812 | 0.734 | 0.771 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n | 12 | 204 | 0.793 | +0.022 | 0.831 | 0.759 | 0.793 | 0.217 | 0.571 | 0.589 |

Từ `outputs/round1_diff.md`: model đề xuất 176 box, sau khi sửa còn 204 box. Cụ thể: giữ nguyên 131 box (accepted), kéo lại 31 box (edited), xóa 14 box false positive (deleted), thêm 42 box bị bỏ sót (added).

AP50 tăng từ 0.771 lên 0.793 (+0.022). Recall xe nhỏ tăng từ 0.182 lên 0.217 — vẫn còn thấp nhưng có cải thiện. Recall xe vừa và xe lớn cũng tăng nhẹ. Nhìn `compare_round0.jpg` và `compare_round1.jpg`: một xe tải lớn ở cuối ảnh frame_0380.jpg mà vòng 0 khoanh lệch về phía sau, vòng 1 đã khoanh sát hơn phần thân đầy đủ.

Phân biệt ba nguồn thông tin: trong `BLIND_SCAN.md`, tôi ghi lại quan sát độc lập trước khi xem pre-label — đếm được khoảng 18 xe trong frame_0099.jpg, nhận ra góc phải dưới có xe bị cắt và khu vực xa có chấm đèn. Trong `REVIEW_LOG.csv`, tôi ghi đúng 6 ca sửa cụ thể trên CVAT: thêm xe bị cắt mép, xóa vệt đèn nhầm, kéo khung cho sát. Sau khi học lại, `round1_diff.md` cho thấy model đã học được một phần từ các sửa đổi đó và AP50 tăng.

## 5. Kết luận và giới hạn

AP50 vòng 1 (0.793) cao hơn cold start (0.771) một khoảng +0.022. Mức cải thiện khiêm tốn nhưng có ý nghĩa với chỉ 12 ảnh huấn luyện thêm. Tôi quyết định dừng ở vòng 1 vì: (1) gain đã dương và ổn định; (2) thêm vòng 2 đòi hỏi sửa thêm 12 ảnh nữa mà lợi ích kỳ vọng giảm dần — mỗi ảnh mới càng khó tìm thêm thông tin mới do hạn chế MIN_GAP_S; (3) tập kiểm tra chỉ 20 ảnh, dao động ngẫu nhiên có thể che lấp xu hướng thật.

Hai ca còn yếu cho vòng sau nếu tiếp tục: (1) xe ở rất xa chỉ còn hai chấm đèn — recall small vẫn chỉ 0.217, cần thêm nhãn ảnh có nhiều xe xa; (2) xe bị cắt mép ảnh — model thường bỏ qua vì khung tham chiếu bị cắt, cần thêm ví dụ. Cả hai ca đều có nguy cơ ảnh gần trùng vì camera đứng yên, phải kiểm kỹ thời gian giữa các ảnh được chọn. Chi phí rà nhãn cao vì xe nhỏ và xe bị cắt khó xác định ranh giới chính xác.

Giới hạn của kết luận: tập kiểm tra chỉ 20 ảnh nên khoảng tin cậy của AP50 rộng; luật bỏ qua xe cao dưới 16 px có thể bỏ qua xe xa mà thật ra model cần học; nhãn tham chiếu do model tạo chưa được người rà thủ công nên một số "lỗi" có thể là lỗi nhãn chứ không phải lỗi model. Nếu AP50 giảm ở vòng sau, tôi sẽ kiểm lại từng khung đã sửa trong REVIEW_LOG.csv — đặc biệt các ca "added" — để xem có nhãn nào vẽ sai quy tắc không trước khi quyết định train thêm.
