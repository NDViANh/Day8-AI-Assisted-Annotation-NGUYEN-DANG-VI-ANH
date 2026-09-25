# Vì sao chọn lô này?

AI chạy trên toàn bộ 268 ảnh pool và xếp hạng theo điểm tổng hợp (uncertainty + annotation density + diversity). Tôi xem xét **50 ứng viên đầu** trong danh sách `selection_round1.csv` trước khi quyết định lô 12 ảnh thực tế sửa.

Nếu chỉ được sửa 5 ảnh, tôi chọn frame_0182.jpg (hạng 1, điểm 0.959), frame_0369.jpg (hạng 2, điểm 0.932), frame_0380.jpg (hạng 3, điểm 0.917), frame_0326.jpg (hạng 4, điểm 0.916) và frame_0331.jpg (hạng 5, điểm 0.915). Năm ảnh này đứng đầu danh sách theo điểm bất định. frame_0182.jpg và frame_0183.jpg cách nhau khoảng 0.4 giây, cảnh gần như giống hệt nhau, nên tôi không lấy cả hai — chọn frame_0182.jpg và bỏ frame_0183.jpg (hạng 2 thực sự, điểm 0.941) để tránh lãng phí ngân sách nhãn trên cảnh trùng lặp.

Trong 12 ảnh AI đã chọn, tôi nhìn kỹ frame_0182.jpg, frame_0099.jpg và frame_0107.jpg. Cả ba đều có điểm trên 0.88 và AI khoanh nhiều xe nhưng còn nhiều khung chưa chắc — thiếu xe bị cắt mép, nhầm vệt đèn thành xe, hai khung chồng lên một xe. Đây là bằng chứng rõ nhất từ file selection_round1.csv kết hợp với ảnh contact sheet.

frame_0372.jpg đứng hạng 6 trong toàn bộ pool với điểm 0.910, cao hơn một số ảnh đã được chọn như frame_0270.jpg (0.885) hay frame_0392.jpg (0.871), nhưng AI bỏ qua vì nó cách frame_0369.jpg chỉ 1.2 giây (148.8 − 147.6 = 1.2 s < MIN_GAP_S = 2 s). Hai ảnh gần như cùng một cảnh giao thông, sửa cả hai thì tốn công mà thông tin học thêm gần như bằng không.

Điểm bất định cao chỉ nghĩa là AI đang phân vân với ảnh đó — nó chưa chứng minh được rằng sửa ảnh đó xong thì AI sẽ nhận xe tốt hơn trên tập kiểm tra. Một ảnh khó với AI có thể chỉ là do ánh sáng bất thường hoặc góc che khuất hiếm gặp, không đại diện cho phần lớn dữ liệu thực tế.
