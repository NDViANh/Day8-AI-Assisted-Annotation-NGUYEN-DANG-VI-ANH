# Bảng so sánh các vòng

Tập kiểm thử: 20 ảnh, 417 box tham chiếu (bỏ qua 12 box cao dưới 16 px). Ngưỡng IoU 0.5; P, R, F1 tính tại conf 0.25.

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n | 0 | 0 | 0.771 | — | 0.812 | 0.734 | 0.771 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n | 12 | 204 | 0.793 | +0.022 | 0.831 | 0.759 | 0.793 | 0.217 | 0.571 | 0.589 |
